# Mahsulot talablari hujjati (PRD): AI Ijtimoiy Tarmoq Kontent Generatori

**Hujjat versiyasi:** 1.0  
**Sana:** 2026-yil 16-noyabr  
**Asosiy AI modeli:** Gemini 3.5 Flash Lite API  
**Platforma yo‘nalishi:** Mobile-First Web Application (Birinchi navbatda mobil qurilmalarga mo‘ljallangan veb-ilova)  

## 1. Loyiha haqida umumiy ma’lumot

### 1.1. Loyiha maqsadi
Ushbu loyiha kichik va o‘rta biznes egalari, SMM mutaxassislari hamda kontent kreatorlar uchun **Gemini 3.5 Flash Lite API** bazasida ishlovchi tezkor, qulay va mobil qurilmalarga moslashtirilgan veb-ilovani yaratishga qaratilgan. Ilova foydalanuvchidan biznes tavsifi va post g‘oyasini qabul qilib, tanlangan ijtimoiy tarmoq (**Telegram**, **Instagram** yoki **YouTube**) xususiyatlariga to‘liq mos keladigan tayyor post variantlarini yaratib beradi.

### 1.2. Muammo va yechim
* **Muammo:** Biznes egalari va kontent mualliflari har kuni turli platformalar uchun moslashtirilgan, jalb qiluvchi sarlavha, matn, harakatga chaqiruv (CTA) va xeshteglardan iborat postlarni yozishga ko‘p vaqt sarflaydilar. Mobil qurilmalarda uzun matnlarni tahrirlash va oldingi variantlarni saqlab borish noqulaylik tug‘diradi.
* **Yechim:** Foydalanuvchi atigi ikkita maydonni to‘ldirishi va platformani tanlashi orqali bir necha soniya ichida **“yozuv mashinkasi” (typewriter)** effekti bilan jonli shakllanadigan bir nechta post variantlariga ega bo‘ladi. Yoqqan variantni bir marta bosish orqali nusxalash, qayta generatsiya qilish yoki tarixda saqlab qo‘yish mumkin.

## 2. Maqsadli auditoriya va foydalanuvchi stsenariylari

### 2.1. Maqsadli foydalanuvchilar
| Foydalanuvchi guruhi | Ehtiyoji | Asosiy foydalanish qurilmasi |
| :--- | :--- | :--- |
| **Kichik biznes egalari** | Mahsulot va aksiyalar haqida tezda sifatli post chiqarish | Smartfon (iOS / Android brauzerlari) |
| **SMM menejerlar** | Bir vaqtning o‘zida Telegram, Instagram va YouTube uchun bir nechta kreativ variantlar olish | Smartfon va planshet |
| **Kontent kreatorlar / Blogerlar** | Video tavsiflari, post g‘oyalari va auditoriyani faollashtiruvchi CTA iboralarini topish | Smartfon |

### 2.2. Foydalanuvchi stsenariylari (User Stories)
1. **Ma’lumot kiritish:** Foydalanuvchi sifatida men biznesim haqida qisqacha ma’lumot va bugungi post g‘oyasini kiritishni xohlayman, shunda sun’iy intellekt brendim uslubiga mos matn yozadi.
2. **Platformani tanlash:** Foydalanuvchi sifatida men Telegram, Instagram yoki YouTube platformalaridan birini tanlashni xohlayman, shunda yaratilgan post o‘sha tarmoqning format va ohangiga mos bo‘ladi.
3. **Tuzilgan post qismlari:** Foydalanuvchi sifatida men har bir postda aniq ajratilgan **Sarlavha**, **Asosiy matn**, **Harakatga chaqiruv (CTA)** va **Xeshteglar** bo‘lishini xohlayman.
4. **Bir nechta variantlar:** Foydalanuvchi sifatida men bitta so‘rovda bir nechta post variantini ko‘rishni va ulardan eng yaxshisini tanlashni xohlayman.
5. **Jonli matn effekti:** Foydalanuvchi sifatida men javobning “yozuv mashinkasi” effekti bilan bosqichma-bosqich paydo bo‘lishini xohlayman, shunda kutish vaqti sezilmaydi va jarayon interaktiv ko‘rinadi.
6. **Qayta yaratish (Regenerate):** Foydalanuvchi sifatida men kiritilgan ma’lumotlarni qaytadan yozmasdan, bitta tugma orqali yangi variantlarni qayta generatsiya qilishni xohlayman.
7. **Tarix va tezkor nusxalash:** Foydalanuvchi sifatida men avval yaratilgan postlar tarixini ko‘rishni va istalgan postni bir marta bosish orqali vaqtinchalik xotiraga (clipboard) nusxalashni xohlayman.

## 3. Funksional talablar

### 3.1. Ma’lumotlarni kiritish moduli (Input Form)
* **Biznes haqida ma’lumot maydoni (`business_info`):**
  * Ko‘p qatorli matn maydoni (`textarea`), tavsiya etilgan uzunlik: 10–500 belgi.
  * Misol ko‘rsatkichi (placeholder): *"Masalan: Toshkentdagi tabiiy shirinliklar va qahvaxona tarmog‘i, maqsadli auditoriya — yoshlar va oilalar."*
  * Qulaylik uchun oxirgi kiritilgan biznes ma’lumotini brauzer xotirasida (`localStorage`) saqlab qolish, toki foydalanuvchi har safar uni qayta yozmasin.
* **Post g‘oyasi / mavzusi maydoni (`post_idea`):**
  * Ko‘p qatorli matn maydoni (`textarea`), tavsiya etilgan uzunlik: 10–1000 belgi.
  * Misol ko‘rsatkichi (placeholder): *"Masalan: Juma kuni soat 14:00 dan 18:00 gacha barcha kruassanlarga 30% chegirma e’lon qilish."*
* **Validatsiya:** Har ikki maydon bo‘sh bo‘lganda "Yaratish" tugmasi nofaol (`disabled`) holatda bo‘ladi yoki foydalanuvchiga yumshoq ogohlantirish xabari ko‘rsatiladi.

### 3.2. Platforma tanlash moduli (Platform Selector)
Foydalanuvchi bitta faol platformani katta, bosh barmoq bilan bosishga qulay (touch-friendly) tugmalar (segmented control / pill cards) orqali tanlaydi:

| Platforma | Kontent uslubi va format qoidalari |
| :--- | :--- |
| **Telegram** | Qisqa va aniq paragraflar, o‘rinli emojilar, muhim iboralarni qalin (`bold`) shriftda ajratish, munozaraga yoki bot/kanal havolasiga yo‘naltiruvchi CTA, 3–5 ta dolzarb xeshteg. |
| **Instagram** | Birinchi qatordayoq e’tiborni tortuvchi kuchli "hook" (sarlavha), vizual tasavvur uyg‘otuvchi hikoyanamo yoki ro‘yxatli matn, izoh qoldirish / saqlab qo‘yish / Direct'ga yozishga undovchi CTA, 8–15 ta maqsadli xeshteg. |
| **YouTube** | Qidiruvga mos (SEO-friendly) va qiziqtiruvchi sarlavha, videoning qisqacha mazmuni yoki Community post matni, obuna bo‘lish / layk bosish / izohda fikr bildirishga chaqiruvchi CTA, 3–8 ta kalit so‘zli xeshteg. |

### 3.3. AI Generatsiya va Tarkibiy Tuzilma
Har bir yaratilgan post varianti qat’iy ravishda quyidagi 4 ta mantiqiy blokdan iborat bo‘lishi shart:
1. **Sarlavha (Headline / Hook):** O‘quvchi e’tiborini dastlabki 2 soniyada tortuvchi qisqa va ta’sirchan jumla.
2. **Asosiy matn (Body):** Biznes qiymati va post g‘oyasini ochib beruvchi, o‘qishga yengil va abzaslarga bo‘lingan asosiy qism.
3. **Harakatga chaqiruv (Call to Action — CTA):** Auditoriyani aniq bir amalni bajarishga (xarid qilish, izoh yozish, havolaga o‘tish, do‘stlariga ulashish) undovchi yakuniy jumla.
4. **Xeshteglar (Hashtags):** Mavzu va platformaga mos `#xeshteglar` to‘plami.

### 3.4. Bir nechta variantlarni taqdim etish (Multiple Variants)
* Bitta generatsiya so‘rovida tizim **3 ta turli ohangdagi post variantini** yaratadi:
  * **1-variant (Sotuv va harakatga yo‘naltirilgan):** To‘g‘ridan-to‘g‘ri foyda, aksiya va tezkor qaror qabul qilishga urg‘u beradi.
  * **2-variant (Hikoyanamo / Samimiy):** Auditoriya bilan hissiy aloqa o‘rnatish va muammo-yechim orqali yondashishga urg‘u beradi.
  * **3-variant (Qisqa va dinamik):** Tez o‘qiladigan, punktlarga (bullet points) asoslangan zamonaviy format.
* Mobil ekranda joyni tejash maqsadida variantlar **gorizontal varaqlar (Tabs: "Variant 1", "Variant 2", "Variant 3")** yoki **suriladigan kartochkalar (Swipeable Carousel)** ko‘rinishida taqdim etiladi.

### 3.5. “Yozuv mashinkasi” effekti (Typewriter Streaming Effect)
* Natijalar ekranda birdaniga emas, balki harf-ma-harf / so‘z-ma-so‘z bosqichma-bosqich paydo bo‘ladi.
* **Texnik amalga oshirish:**
  * Gemini 3.5 Flash Lite API’ning `streamGenerateContent` oqimli javobidan foydalanish yoki qabul qilingan tuzilgan javobni mijoz tomonida (`requestAnimationFrame` / `setInterval` — har 12–20 ms da) silliq yozuv mashinkasi animatsiyasi bilan chiqarish.
  * Matn yozilayotgan vaqtda oxirida miltillovchi kursor (`▍`) ko‘rsatiladi.
  * Foydalanuvchi xohlasa, **"Animatsiyani o‘tkazib yuborish" (Skip)** tugmasini bosib, to‘liq matnni darhol ko‘rishi mumkin.

### 3.6. Natijani qayta yaratish (Regenerate)
* Natija kartochkasining pastki qismida **"Qayta yaratish" (Regenerate)** tugmasi joylashadi.
* Tugma bosilganda, foydalanuvchi kiritgan biznes ma’lumoti, g‘oya va tanlangan platforma saqlangan holda Gemini 3.5 Flash Lite API’ga yangi so‘rov yuboriladi (yuqoriroq kreativlik yoki yangi burchak ko‘rsatmasi bilan) va yangi 3 ta variant yozuv mashinkasi effekti bilan shakllanadi.

### 3.7. Postlar tarixi va Bir marta bosish orqali nusxalash (History & One-Click Copy)
* **Bir marta bosish orqali nusxalash (One-Click Copy):**
  * Har bir joriy variantda va tarixdagi har bir postda **"Nusxalash" (Copy)** tugmasi mavjud.
  * Bosilganda `navigator.clipboard.writeText()` API orqali postning to‘liq matni (Sarlavha + Asosiy matn + CTA + Xeshteglar) xotiraga nusxalanadi.
  * Tugma ikonkasi 2 soniyaga tasdiq belgisiga (`✓ Nusxalandi!`) o‘zgaradi va ekranning pastki qismida qisqa bildirishnoma (Toast notification) chiqadi.
* **Postlar tarixi (Post History):**
  * Foydalanuvchi yaratgan yoki tanlagan postlar avtomatik ravishda tarix ro‘yxatiga (`localStorage` orqali kamida oxirgi 50 ta yozuv) saqlanadi.
  * Har bir tarix elementida quyidagilar aks etadi:
    * Platforma nishoni (Telegram / Instagram / YouTube ikonkasi va rangi).
    * Yaratilgan vaqti (masalan, *"Bugun, 19:30"*).
    * Post sarlavhasi va asosiy matndan qisqa parcha (preview).
    * **Tezkor "Nusxalash" tugmasi** (postni ochib o‘tirmasdan ro‘yxatning o‘zidan bir marta bosishda nusxalash imkoniyati).
    * Element ustiga bosganda to‘liq matnni ko‘rish va yakka tartibda o‘chirish yoki butun tarixni tozalash tugmasi.

## 4. Mobil qurilmalarga moslashtirilgan UI/UX talablari (Mobile-First Design)

### 4.1. Ekran va tartib (Layout) mezonlari
* **Asosiy ko‘rinish (Viewport):** `360px` dan `430px` gacha bo‘lgan smartfon ekranlari asosiy dizayn nuqtasi hisoblanadi; planshet va kompyuter ekranlarida (`max-width: 640px` markazlashgan konteyner) ham toza ko‘rinishini saqlaydi.
* **Bosh barmoq zonasi (Thumb Zone):** Barcha asosiy interaktiv elementlar (Platforma tanlash, "Post yaratish", "Qayta yaratish", "Nusxalash" va "Tarix" bo‘limlariga o‘tish) ekranning pastki va o‘rta qismida, oson yetib boriladigan hududda joylashadi.
* **Sensorli nishon o‘lchami (Touch Targets):** Har bir tugma va interaktiv element kamida `48x48 px` o‘lchamga ega bo‘lishi shart.
* **Mobil klaviatura moslashuvi:** `input` va `textarea` shrift o‘lchami kamida `16px` bo‘lishi kerak (iOS Safari brauzerida avtomatik yaqinlashtirish — auto-zoom muammosining oldini olish uchun).

### 4.2. Ekran tuzilmasi (Navigation & Views)
1. **Yuqori panel (Header):** Ilova logotipi, nomi va "Tarix" hisoblagichi (masalan, `Tarix (12)`).
2. **Asosiy ishchi maydon (Generator Tab):**
   * Biznes haqida ma’lumot kiritish bloki.
   * Post g‘oyasini kiritish bloki.
   * Platforma tanlash tugmalari (Telegram | Instagram | YouTube).
   * Asosiy harakat tugmasi: **"✨ Post yaratish"**.
3. **Natija maydoni (Output Card):**
   * Variantlar tablari (`1-variant`, `2-variant`, `3-variant`).
   * Yozuv mashinkasi effekti bilan chiqadigan strukturalangan post (Sarlavha, Matn, CTA, Xeshteglar vizual ajratilgan holda).
   * Harakatlar qatori: **"📋 Nusxalash"** (asosiy urg‘uli tugma) va **"🔄 Qayta yaratish"**.
4. **Tarix bo‘limi (Bottom Sheet / Tab):**
   * Platformalar bo‘yicha filtrlash (`Barchasi`, `Telegram`, `Instagram`, `YouTube`).
   * Har bir kartochkada bir marta bosish bilan nusxalash tugmasi.

## 5. Texnik arxitektura va Gemini 3.5 Flash Lite API integratsiyasi

### 5.1. Texnologik stek tavsiyasi
* **Frontend:** HTML5, Tailwind CSS (yoki zamonaviy Mobile-First CSS Grid/Flexbox), React / Vue / Vanilla JS (ES6+).
* **AI Model:** Google Gemini API — `gemini-3.5-flash-lite` (past kechikish va yuqori tezlikda matn generatsiyasi uchun optimallashtirilgan model).
* **Ma’lumotlarni saqlash:** `window.localStorage` (avtorizatsiyasiz MVP bosqichida postlar tarixi va biznes profilini saqlash uchun).

### 5.2. Prompt muhandisligi va ma’lumotlar sxemasi
Gemini 3.5 Flash Lite API’dan barqaror va tuzilgan javob olish uchun tizim ko‘rsatmasi (`systemInstruction`) hamda JSON sxemasi yoki maxsus ajratgichli striming formati qo‘llaniladi.

**Tavsiya etilgan JSON chiqish sxemasi (`responseSchema`):**

```json
{
  "type": "OBJECT",
  "properties": {
    "platform": { "type": "STRING" },
    "variants": {
      "type": "ARRAY",
      "items": {
        "type": "OBJECT",
        "properties": {
          "variant_title": { "type": "STRING" },
          "headline": { "type": "STRING" },
          "body": { "type": "STRING" },
          "cta": { "type": "STRING" },
          "hashtags": {
            "type": "ARRAY",
            "items": { "type": "STRING" }
          }
        },
        "required": ["variant_title", "headline", "body", "cta", "hashtags"]
      }
    }
  },
  "required": ["platform", "variants"]
}
```

### 5.3. Tizim prompti shabloni (System Prompt)
```text
Sen professional SMM strateg va kopiraytersan. Foydalanuvchi kiritgan biznes ma’lumoti va post g‘oyasi asosida tanlangan ijtimoiy tarmoq ({platform}) uchun o‘zbek tilida 3 ta bir-biridan farq qiluvchi, yuqori konversiyali post variantini yaratib ber.

Qoidalar:
1. Har bir variantda quyidagi 4 qism bo‘lishi shart:
   - Sarlavha (headline): E’tiborni tortuvchi kuchli kirish jumlasi.
   - Asosiy matn (body): Platforma formatiga mos, o‘qishga qulay va qiymat beruvchi asosiy matn.
   - Harakatga chaqiruv (cta): Auditoriyani aniq harakatga undovchi jumla.
   - Xeshteglar (hashtags): Mavzuga mos xeshteglar ro‘yxati.
2. Platforma xususiyatlari:
   - Telegram: Qisqa abzaslar, muhim joylarga urg‘u, samimiy va ekspert ohangi.
   - Instagram: Emotsional kirish, vizual tasavvur, izoh va saqlashga undovchi CTA.
   - YouTube: Qiziqtiruvchi sarlavha, mazmunli tavsif va obuna/izoh uchun CTA.
```

## 6. Nofunksional talablar

| Kategoriya | Talab ko‘rsatkichi |
| :--- | :--- |
| **Tezlik (Latency)** | Gemini 3.5 Flash Lite API orqali birinchi belgi paydo bo‘lish vaqti (TTFB) < 1.2 soniya. |
| **Animatsiya silliqligi** | Yozuv mashinkasi effekti mobil brauzerlarda 60 FPS tezlikda qotishlarsiz ishlashi va sahifa pastga avtomatik silliq siljishi (auto-scroll) kerak. |
| **Xatoliklarni boshqarish** | Internet uzilishi, API kaliti xatoligi yoki limit tugashi holatlarida foydalanuvchiga tushunarli o‘zbekcha xabar va "Qayta urinib ko‘rish" tugmasi ko‘rsatilishi shart. |
| **Xavfsizlik** | API kaliti ochiq frontend kodida qoldirilmasligi, serverless funksiya (masalan, Cloudflare Workers, Vercel Edge yoki backend proksi) orqali himoyalanishi yoki test rejimida xavfsiz muhit o‘zgaruvchisida saqlanishi lozim. |
| **Oflayn chidamlilik** | Internet yo‘q bo‘lganda ham foydalanuvchi "Postlar tarixi" bo‘limiga kirib, avvalgi postlarni ko‘rishi va bir marta bosishda nusxalashi mumkin bo‘lishi kerak. |

## 7. Qabul qilish mezonlari (Acceptance Criteria)

1. Foydalanuvchi biznes ma’lumoti va g‘oyani kiritib, Telegram, Instagram yoki YouTube’dan birini tanlay oladi.
2. "Post yaratish" tugmasi bosilganda, Gemini 3.5 Flash Lite API chaqiriladi va natija "yozuv mashinkasi" effekti bilan bosqichma-bosqich ekranga chiqadi.
3. Har bir natijada **Sarlavha**, **Asosiy matn**, **CTA** va **Xeshteglar** aniq ko‘rinadi hamda tanlash uchun **3 ta variant** taqdim etiladi.
4. "Qayta yaratish" tugmasi bosilganda forma ma’lumotlari yo‘qolmasdan yangi variantlar generatsiya qilinadi.
5. "Postlar tarixi" ro‘yxatida oldingi postlar saqlanadi va har bir postni ro‘yxatning o‘zidan **bir marta bosish orqali** xotiraga nusxalash (vizual tasdiq bilan) to‘liq ishlaydi.
6. Ilova interfeysi `360px` kenglikdagi mobil ekranda gorizontal siljishlarsiz (horizontal scroll’siz) va qulay tugmalar bilan ishlaydi.

## 8. Rivojlantirish bosqichlari (Roadmap)

* **1-bosqich (MVP):** Kiritish formasi, platforma tanlagich, Gemini 3.5 Flash Lite API integratsiyasi, 3 ta variant generatsiyasi, yozuv mashinkasi effekti, qayta yaratish va `localStorage` asosidagi bir marta bosishda nusxalanadigan postlar tarixi.
* **2-bosqich (v1.1):** Post ohangini tanlash (Rasmiy, Do‘stona, Hazilomuz, Sotuv), til tanlash (O‘zbek, Rus, Ingliz) va tanlangan postni bevosita Telegram/Instagram ilovasiga "Ulashish" (`navigator.share` Web Share API) imkoniyati.
* **3-bosqich (v2.0):** Bulutli sinxronizatsiya, foydalanuvchi profili, post uchun sun’iy intellekt yordamida rasm/banner promptlarini yaratish va kontent-reja kalendari.
