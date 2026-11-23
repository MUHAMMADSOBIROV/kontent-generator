# Texnik Spetsifikatsiya (Technical Spec): AI Ijtimoiy Tarmoq Kontent Generatori

**Hujjat versiyasi:** 1.0  
**Asosiy hujjat:** Mahsulot talablari hujjati (PRD v1.0)  
**Sana:** 2026-yil 16-noyabr  
**Maqsadli platforma:** Mobile-First Web Application (PWA-ready SPA)  
**AI Dvigateli:** Google Gemini 3.5 Flash Lite API (`gemini-3.5-flash-lite`)  

## 1. Tizim Arxitekturasi

### 1.1. Yuqori darajadagi arxitektura
Ilova uch qatlamli yengil (lightweight) arxitektura asosida quriladi:
1. **Taqdimot qatlami (Client UI — Mobile-First SPA):** Foydalanuvchi kiritmalari, platforma tanlagich, "yozuv mashinkasi" (typewriter) render dvigateli, variantlar karuseli/tablari va postlar tarixi oynasini boshqaradi.
2. **Xavfsiz API Proksi qatlami (Edge / Serverless Function):** Frontenddan kelgan so‘rovlarni qabul qiladi, `GEMINI_API_KEY` kalitini yashirgan holda Google Gemini 3.5 Flash Lite API bilan Server-Sent Events (SSE) / ReadableStream orqali bog‘lanadi.
3. **Mahalliy ma’lumotlar qatlami (`localStorage` Storage Engine):** Biznes profili ma’lumotlari va oxirgi 50 ta generatsiya qilingan postlar tarixini oflayn rejimda ham ishlashi uchun brauzerda saqlaydi.

### 1.2. Texnologik stek
| Qatlam | Texnologiya | Tanlov sababi |
| :--- | :--- | :--- |
| **Frontend Framework** | React 19 + TypeScript (Vite) *yoki* Vanilla ES6+ | Tezkor yuklanish (bundle size < 50 KB gzipped), qat’iy tipizatsiya |
| **Stillashtirish (Styling)** | Tailwind CSS v4 (Mobile-First utility classes) | `360px–430px` mobil ekranlarga moslashuvchanlik va `48px` touch-target standartlari |
| **Holat boshqaruvi (State)** | Zustand *yoki* React Context + Custom Hooks | `localStorage` bilan bevosita sinxronlash va oqimli matn holatini boshqarish |
| **Backend Proksi** | Vercel Edge Functions / Cloudflare Workers / Node.js | Past kechikish (Edge TTFB < 100ms), `ReadableStream` striming qo‘llab-quvvatlovi |
| **AI SDK / Endpoint** | `@google/genai` SDK (`gemini-3.5-flash-lite`) | Strukturalangan JSON va yuqori tezlikdagi token strimingi |

### 1.3. Ma’lumotlar oqimi ketma-ketligi (Data Flow)
1. Foydalanuvchi `businessInfo` va `postIdea` maydonlarini to‘ldiradi hamda `platform` (`telegram` | `instagram` | `youtube`) qiymatini tanlaydi.
2. Mijoz ilovasi `businessInfo` qiymatini darhol `localStorage` ga saqlaydi va `POST /api/generate` endpointiga so‘rov yuboradi.
3. Serverless proksi tizim prompti (`systemInstruction`) va foydalanuvchi parametrlarini birlashtirib, `gemini-3.5-flash-lite` modelining `streamGenerateContent` metodini chaqiradi.
4. Oqimli javob (SSE / Stream chunks) mijozga uzatiladi; `TypewriterEngine` matn bo‘laklarini buferga yig‘ib, `15ms` qadam bilan ekranda jonli yozuv mashinkasi effektini hosil qiladi.
5. Generatsiya yakunlangach, 3 ta variant (`variants[0..2]`) to‘liq obyekt sifatida `HistoryManager` orqali `localStorage` tarixiga yoziladi.

## 2. Ma’lumotlar Modellari va TypeScript Interfeyslari

```typescript
// 1. Qo'llab-quvvatlanadigan ijtimoiy tarmoq platformalari
export type PlatformType = 'telegram' | 'instagram' | 'youtube';

// 2. Bitta post variantining ichki tuzilmasi
export interface PostVariant {
  id: string;                  // Masalan: "var_1", "var_2", "var_3"
  variantTitle: string;        // Masalan: "1-variant: Sotuv", "2-variant: Hikoyanamo", "3-variant: Dinamik"
  headline: string;            // Sarlavha (Hook)
  body: string;                // Asosiy matn
  cta: string;                 // Harakatga chaqiruv (Call to Action)
  hashtags: string[];          // ["#biznes", "#aksiya", ...]
}

// 3. API ga yuboriladigan so'rov (Request Payload)
export interface GenerationRequest {
  businessInfo: string;        // 10 - 500 belgi
  postIdea: string;            // 10 - 1000 belgi
  platform: PlatformType;
  isRegenerate?: boolean;      // Qayta yaratish bosilganda true bo'ladi
  previousHeadlines?: string[]; // Qayta yaratishda takrorlanishning oldini olish uchun
}

// 4. API dan qaytadigan tuzilgan javob (Response Payload)
export interface GenerationResponse {
  platform: PlatformType;
  variants: [PostVariant, PostVariant, PostVariant]; // Qat'iy 3 ta variant
  generatedAt: string;         // ISO 8601 vaqt tamg'asi
}

// 5. Postlar tarixida saqlanadigan yozuv (History Item)
export interface HistoryItem {
  id: string;                  // UUID v4
  createdAt: string;           // ISO 8601
  platform: PlatformType;
  businessInfo: string;
  postIdea: string;
  selectedVariantIndex: number; // Foydalanuvchi oxirgi ko'rgan/nusxalagan variant (0, 1 yoki 2)
  variants: PostVariant[];
}

// 6. Ilova UI holati (State Machine)
export type GenerationStatus = 'idle' | 'connecting' | 'streaming' | 'completed' | 'error';

export interface AppState {
  businessInfo: string;
  postIdea: string;
  selectedPlatform: PlatformType;
  status: GenerationStatus;
  activeVariantIndex: number;  // 0 | 1 | 2
  currentVariants: PostVariant[];
  isTypewriterActive: boolean;
  errorMessage: string | null;
  history: HistoryItem[];
  historyFilter: 'all' | PlatformType;
}
```

## 3. Gemini 3.5 Flash Lite API Spetsifikatsiyasi

### 3.1. Endpoint va Konfiguratsiya
* **Model identifikatori:** `gemini-3.5-flash-lite`
* **Metod:** `ai.models.generateContentStream` (yoki REST: `POST https://generativelanguage.googleapis.com/v1beta/models/gemini-3.5-flash-lite:streamGenerateContent?alt=sse`)
* **Generatsiya parametrlari (`generationConfig`):**

| Parametr | Standart ("Post yaratish") | Qayta yaratish ("Regenerate") | Izoh |
| :--- | :--- | :--- | :--- |
| `temperature` | `0.75` | `0.90` | Qayta yaratishda yangi kreativ burchaklar olish uchun oshiriladi |
| `topP` | `0.95` | `0.95` | Tabiiy o‘zbek tili leksikasini saqlash uchun |
| `maxOutputTokens` | `2048` | `2048` | 3 ta to‘liq variant uchun yetarli zaxira |
| `responseMimeType` | `application/json` | `application/json` | Qat’iy JSON sxema bo‘yicha javob olish uchun |

### 3.2. Platformaga mos qoidalar matritsasi (Prompt Context Injection)
Backend proksi tanlangan `platform` qiymatiga qarab tizim promptiga quyidagi spetsifik qoidalarni qo‘shadi:

```typescript
export const PLATFORM_RULES: Record<PlatformType, string> = {
  telegram: `
    - Format: Qisqa va ravon abzaslar (har bir abzas 2-3 jumladan oshmasin).
    - Vizual urg'u: Asosiy kalit iboralar va raqamlarni o'rinli emojilar bilan ajrat.
    - CTA: Kanalga obuna bo'lish, izohlarda fikr qoldirish yoki bot/admin profiliga yozishga chaqir.
    - Xeshteglar: 3 tadan 5 tagacha eng muhim xeshteg.
  `,
  instagram: `
    - Format: Birinchi jumla (Headline) lentani varaqlayotgan odamni to'xtatadigan kuchli "Hook" bo'lsin.
    - Asosiy matn: Hissiyotga boy hikoya yoki o'qishga qulay punktli ro'yxat.
    - CTA: Postni saqlab qo'yish (Save), do'stlarga yuborish, izohda "+" qoldirish yoki Direct'ga yozishga unda.
    - Xeshteglar: 8 tadan 15 tagacha maqsadli va trenddagi xeshteglar.
  `,
  youtube: `
    - Format: Sarlavha qidiruvga mos (SEO) va klikbop (lekin aldamchi bo'lmagan) bo'lsin.
    - Asosiy matn: Video tavsifi (Description) yoki YouTube Community posti uchun qiziqtiruvchi anons va asosiy foydalar.
    - CTA: Kanalga obuna bo'lish, qo'ng'iroqchani bosish va izohda o'z tajribasini yozishga chaqir.
    - Xeshteglar: 3 tadan 8 tagacha SEO kalit so'zli xeshteglar.
  `
};
```

### 3.3. Serverless API Handler Namuna Kodi (`/api/generate.ts`)

```typescript
import { GoogleGenAI, Type, Schema } from '@google/genai';
import { PLATFORM_RULES, GenerationRequest } from './types';

const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

const postResponseSchema: Schema = {
  type: Type.OBJECT,
  properties: {
    platform: { type: Type.STRING },
    variants: {
      type: Type.ARRAY,
      items: {
        type: Type.OBJECT,
        properties: {
          id: { type: Type.STRING },
          variantTitle: { type: Type.STRING },
          headline: { type: Type.STRING },
          body: { type: Type.STRING },
          cta: { type: Type.STRING },
          hashtags: {
            type: Type.ARRAY,
            items: { type: Type.STRING }
          }
        },
        required: ['id', 'variantTitle', 'headline', 'body', 'cta', 'hashtags']
      }
    }
  },
  required: ['platform', 'variants']
};

export async function POST(req: Request): Promise<Response> {
  try {
    const body: GenerationRequest = await req.json();
    const { businessInfo, postIdea, platform, isRegenerate, previousHeadlines } = body;

    if (!businessInfo || businessInfo.trim().length < 10 || !postIdea || postIdea.trim().length < 10) {
      return new Response(
        JSON.stringify({ error: "Biznes ma'lumoti va post g'oyasi kamida 10 ta belgidan iborat bo'lishi kerak." }),
        { status: 400, headers: { 'Content-Type': 'application/json' } }
      );
    }

    const avoidClause = isRegenerate && previousHeadlines?.length
      ? `\nDIQQAT: Avvalgi variantlarda ishlatilgan quyidagi sarlavha va yondashuvlarni takrorlama, mutlaqo yangi kreativ burchaklardan foydalan: ${previousHeadlines.join(' | ')}`
      : '';

    const systemInstruction = `
Sen professional SMM strateg va kopiraytersan. Foydalanuvchi kiritgan biznes ma'lumoti va post g'oyasi asosida ${platform.toUpperCase()} platformasi uchun o'zbek tilida 3 ta bir-biridan farq qiluvchi post variantini yarat:
- 1-variant ("1-variant: Sotuv"): To'g'ridan-to'g'ri foyda va harakatga yo'naltirilgan.
- 2-variant ("2-variant: Hikoyanamo"): Samimiy, muammo va yechimga asoslangan.
- 3-variant ("3-variant: Dinamik"): Qisqa, punktlarga (bullet points) ajratilgan zamonaviy uslub.

Platforma qoidalari:
${PLATFORM_RULES[platform]}
${avoidClause}
    `.trim();

    const responseStream = await ai.models.generateContentStream({
      model: 'gemini-3.5-flash-lite',
      contents: `Biznes haqida ma'lumot: ${businessInfo.trim()}\nPost g'oyasi: ${postIdea.trim()}\nPlatforma: ${platform}`,
      config: {
        systemInstruction,
        temperature: isRegenerate ? 0.9 : 0.75,
        topP: 0.95,
        maxOutputTokens: 2048,
        responseMimeType: 'application/json',
        responseSchema: postResponseSchema
      }
    });

    const encoder = new TextEncoder();
    const readable = new ReadableStream({
      async start(controller) {
        try {
          for await (const chunk of responseStream) {
            if (chunk.text) {
              controller.enqueue(encoder.encode(`data: ${JSON.stringify({ chunk: chunk.text })}\n\n`));
            }
          }
          controller.enqueue(encoder.encode(`data: [DONE]\n\n`));
          controller.close();
        } catch (streamError) {
          controller.enqueue(encoder.encode(`data: ${JSON.stringify({ error: 'STREAM_INTERRUPTED' })}\n\n`));
          controller.close();
        }
      }
    });

    return new Response(readable, {
      headers: {
        'Content-Type': 'text/event-stream; charset=utf-8',
        'Cache-Control': 'no-cache, no-transform',
        'Connection': 'keep-alive'
      }
    });
  } catch (error) {
    return new Response(
      JSON.stringify({ error: "Server xatoligi yoki API limiti tugadi. Iltimos, birozdan so'ng qayta urinib ko'ring." }),
      { status: 500, headers: { 'Content-Type': 'application/json' } }
    );
  }
}
```

## 4. “Yozuv Mashinkasi” (Typewriter) Effekti Algoritmi

### 4.1. Ishlash mexanizmi
Mobil qurilmalarda yuqori silliqlik (60 FPS) va tuzilgan ko‘rinishni ta’minlash uchun ikki bosqichli gibrid renderlash qo‘llaniladi:
1. **Oqimni qabul qilish va tahlil qilish:** Gemini 3.5 Flash Lite juda yuqori tezlikda (TTFB < 1.2s) JSON javobni qaytaradi. Oqim yakunlanishi yoki har bir variant bloki tayyor bo‘lishi bilan `TypewriterController` faol variantning 4 ta qismini (`headline` → `body` → `cta` → `hashtags`) ketma-ketlikda jonlantiradi.
2. **Belgi/Bo‘lak navbati (Character Queue):**
   * Har `15ms` da navbatdan `2–3` ta belgi olinib, faol qismga qo‘shib boriladi.
   * Qaysi qism yozilayotgan bo‘lsa, aynan o‘sha qism oxirida miltillovchi kursor (`<span class="animate-pulse text-indigo-600">▍</span>`) turadi.
   * **"O‘tkazib yuborish" (`skipTypewriter()`)**: Foydalanuvchi animatsiya davomida `"Tezlashtirish / To‘liq ko‘rish"` tugmasini bossa yoki boshqa variant tabiga (`2-variant`, `3-variant`) o‘tsa, animatsiya darhol to‘liq matn holatiga (`isTypewriterActive = false`) o‘tadi.

### 4.2. Typewriter Hook Spetsifikatsiyasi (`useTypewriterPost.ts`)

```typescript
export interface TypewriterViewState {
  displayedHeadline: string;
  displayedBody: string;
  displayedCta: string;
  displayedHashtags: string[];
  activeSection: 'headline' | 'body' | 'cta' | 'hashtags' | 'done';
  skip: () => void;
}
```

* **Avtomatik siljish (Auto-scroll):** Matn kengayib borayotganda, agar foydalanuvchi sahifani o‘zi yuqoriga surmagan bo‘lsa (`isUserScrolledUp === false`), natija kartochkasi `element.scrollIntoView({ behavior: 'smooth', block: 'nearest' })` orqali ekran ko‘rinishida ushlab turiladi.

## 5. Mobil UI/UX va Komponentlar Spetsifikatsiyasi

### 5.1. Komponentlar ierarxiyasi
```text
AppRoot (max-w-[640px] mx-auto min-h-dvh flex flex-col bg-slate-50)
├── AppHeader
│   ├── BrandLogo ("SMM AI Studio")
│   └── HistoryTriggerButton (Badge bilan: "Tarix (14)")
├── MainWorkspace (flex-1 px-4 py-3 space-y-4)
│   ├── GeneratorForm
│   │   ├── BusinessInfoTextarea (localStorage auto-save, belgi hisoblagichi 0/500)
│   │   ├── PostIdeaTextarea (belgi hisoblagichi 0/1000)
│   │   ├── PlatformSelector (Segmented Pills: Telegram | Instagram | YouTube)
│   │   └── SubmitButton ("✨ Post yaratish", min-h-[52px])
│   └── OutputSection (status !== 'idle' bo'lganda ko'rinadi)
│       ├── VariantTabs ("1-variant", "2-variant", "3-variant")
│       ├── TypewriterPostCard
│       │   ├── SectionBlock: Sarlavha (Headline)
│       │   ├── SectionBlock: Asosiy matn (Body, whitespace-pre-line)
│       │   ├── SectionBlock: Harakatga chaqiruv (CTA, ajratilgan fonda)
│       │   ├── SectionBlock: Xeshteglar (Pill teglar ko'rinishida)
│       │   └── SkipAnimationButton (faqat isTypewriterActive === true bo'lganda)
│       └── ActionToolbar (Sticky bottom yoki Card footer)
│           ├── CopyPostButton ("📋 Nusxalash" -> "✓ Nusxalandi!")
│           └── RegenerateButton ("🔄 Qayta yaratish")
├── HistoryDrawer (Slide-over / Bottom Sheet modal)
│   ├── PlatformFilterBar (Barchasi | Telegram | Instagram | YouTube)
│   ├── HistoryList
│   │   └── HistoryCard (Platform badge, Vaqt, Sarlavha preview, Bir marta bosishda "Nusxalash" tugmasi, O'chirish)
│   └── ClearAllHistoryButton ("Tarixni tozalash")
└── ToastNotification ("Post vaqtinchalik xotiraga nusxalandi!")
```

### 5.2. Mobil Dizayn Tokenlari va Ergonomika Qoidalari
| Element | CSS / Dizayn spetsifikatsiyasi | Maqsad |
| :--- | :--- | :--- |
| **Konteyner kengligi** | `width: 100%; max-width: 640px; padding: 0 16px;` | `360px` dan katta barcha smartfonlarda gorizontal skrollning oldini olish |
| **Kiritish maydonlari (`textarea`)** | `font-size: 16px; line-height: 1.5; border-radius: 12px; padding: 12px;` | iOS Safari’da fokus bo‘lganda ekranning o‘z-o‘zidan kattalashib ketishini (auto-zoom) bloklash |
| **Sensorli tugmalar** | `min-height: 48px; min-width: 48px; touch-action: manipulation;` | Bosh barmoq bilan xatosiz bosish va 300ms tap-delay’ni yo‘qotish |
| **Platforma ranglari** | Telegram: `#0088CC`, Instagram: `#E1306C`, YouTube: `#FF0000` | Foydalanuvchi qaysi tarmoq tanlanganini bir qarashda ajratishi uchun |

## 6. Xotira (`localStorage`) va Nusxalash (`Clipboard`) Spetsifikatsiyasi

### 6.1. `localStorage` Kalitlari va Kvotani Boshqarish
| Kalit nomi (`Key`) | Ma’lumot turi | Maksimal hajm / Cheklov |
| :--- | :--- | :--- |
| `smm_ai_business_info_v1` | `string` | 500 belgi (`onInput` hodisasida `300ms` debounce bilan avtomatik saqlanadi) |
| `smm_ai_selected_platform_v1` | `PlatformType` | Oxirgi tanlangan platformani eslab qolish (`'telegram'`) |
| `smm_ai_post_history_v1` | `HistoryItem[]` | Maksimal **50 ta** obyekt (FIFO: 51-yozuv qo‘shilganda eng eski yozuv o‘chiriladi) |

### 6.2. Bir Marta Bosishda Nusxalash (`One-Click Copy`) Formatlash Qoidasi
Foydalanuvchi joriy natija kartochkasida yoki "Postlar tarixi" ro‘yxatida **"Nusxalash"** tugmasini bosganda, `formatPostForClipboard(variant: PostVariant)` funksiyasi post qismlarini ijtimoiy tarmoqqa to‘g‘ridan-to‘g‘ri joylashga (`Paste`) tayyor ko‘rinishga keltiradi:

```typescript
export function formatPostForClipboard(variant: PostVariant): string {
  const cleanHashtags = variant.hashtags
    .map(tag => (tag.startsWith('#') ? tag.trim() : `#${tag.trim()}`))
    .join(' ');

  return [
    variant.headline.trim(),
    '',
    variant.body.trim(),
    '',
    variant.cta.trim(),
    '',
    cleanHashtags
  ].join('\n');
}

export async function copyToClipboard(text: string): Promise<boolean> {
  try {
    if (navigator.clipboard && window.isSecureContext) {
      await navigator.clipboard.writeText(text);
      return true;
    }
    // Mobil ichki brauzerlar (Telegram WebView / Instagram In-App Browser) uchun fallback:
    const textArea = document.createElement('textarea');
    textArea.value = text;
    textArea.style.position = 'fixed';
    textArea.style.left = '-9999px';
    document.body.appendChild(textArea);
    textArea.focus();
    textArea.select();
    const successful = document.execCommand('copy');
    document.body.removeChild(textArea);
    return successful;
  } catch {
    return false;
  }
}
```

## 7. Xatoliklarni Boshqarish va Xavfsizlik

### 7.1. Xatolik holatlari jadvali (Error Handling Matrix)
| Xatolik turi | Yuza kelish sababi | UI Harakati va Foydalanuvchiga Xabar |
| :--- | :--- | :--- |
| **`VALIDATION_ERROR`** | Maydonlarda 10 tadan kam belgi kiritilgan | Maydon chegarasi qizil rangga kiradi: *"Iltimos, kamida 10 ta belgidan iborat ma’lumot kiriting."* |
| **`NETWORK_OFFLINE`** | Qurilmada internet aloqasi uzilgan | Banner chiqadi: *"Internet aloqasi yo‘q. Avval yaratilgan postlarni 'Tarix' bo‘limidan ko‘rishingiz va nusxalashingiz mumkin."* |
| **`RATE_LIMIT_429`** | Gemini API so‘rovlar limiti oshib ketgan | *"So‘rovlar ko‘payib ketdi. 10 soniyadan so‘ng qayta urinib ko‘ring."* (Avtomatik countdown tugmasi bilan) |
| **`PARSE_OR_STREAM_ERROR`** | Oqim uzilib qoldi yoki noto‘g‘ri JSON | Xavfsiz fallback: *"Matn yaratishda uzilish yuz berdi."* + **"🔄 Qayta urinib ko‘rish"** tugmasi |

### 7.2. Xavfsizlik talablari
* **XSS himoyasi:** AI tomonidan qaytarilgan barcha matnlar HTML sifatida (`dangerouslySetInnerHTML` yoki `innerHTML`) emas, balki xavfsiz matn tugunlari (`textContent`) sifatida render qilinadi.
* **Prompt Injection himoyasi:** Foydalanuvchi kiritgan `businessInfo` va `postIdea` ma’lumotlari tizim ko‘rsatmasidan (`systemInstruction`) alohida `contents` obyektida uzatiladi va uzunligi server tomonida ham qat’iy cheklanadi (`businessInfo <= 500`, `postIdea <= 1000`).

## 8. Sifat Nazorati va Test Stsenariylari (QA Test Plan)

| Test ID | Tekshiriladigan funksiya | Boshlang‘ich holat / Amal | Kutilayotgan natija (Pass Criteria) |
| :--- | :--- | :--- | :--- |
| **TC-01** | Forma validatsiyasi va saqlash | `businessInfo` ga matn yozib, sahifani yangilash (Refresh) | Matn `localStorage` dan tiklanadi; ikkala maydon to‘lmaguncha "Post yaratish" tugmasi nofaol turadi. |
| **TC-02** | Platformaga mos generatsiya | Telegram, Instagram va YouTube ni navbat bilan tanlab generatsiya qilish | Har bir so‘rovda 3 tadan variant qaytadi; har bir variantda Sarlavha, Matn, CTA va Xeshteglar alohida ko‘rinadi. |
| **TC-03** | Yozuv mashinkasi effekti | "Post yaratish" tugmasini bosish va "Skip" tugmasini sinash | Matn `▍` kursori bilan bosqichma-bosqich chiqadi; "Skip" bosilganda to‘liq matn darhol namoyon bo‘ladi. |
| **TC-04** | Qayta yaratish (Regenerate) | Natija chiqqach, "🔄 Qayta yaratish" tugmasini bosish | Forma ma’lumotlari o‘chmaydi, `isRegenerate: true` bilan yangi 3 ta variant generatsiya qilinadi. |
| **TC-05** | Tarixdan 1-bosishda nusxalash | "Tarix" oynasini ochib, ro‘yxatdagi postning "Nusxalash" tugmasini bosish | Postni ochmasdan turib to‘liq matn clipboard’ga nusxalanadi va `✓ Nusxalandi!` vizual tasdig‘i ko‘rinadi. |
| **TC-06** | Mobil moslashuvchanlik | `360x800` (Android) va `390x844` (iOS) o‘lchamda tekshirish | Gorizontal skroll yo‘q, `textarea` bosilganda sahifa zoom bo‘lib ketmaydi, barcha tugmalar `>=48px`. |
