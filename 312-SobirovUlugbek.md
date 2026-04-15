# Film Tavsiya Platformasi — Tizim Talablari va Arxitekturasi

> **Mavzu:** Film Tavsiya Platformasi  
> **Bosqichlar:** Talablar + Arxitektura

---

## 1-Bosqich. Talablar

### Loyiha Haqida Qisqacha

**Film Tavsiya Platformasi** — foydalanuvchilarga ularning tomosha tarixi, reytinglari va xohishlariga asoslangan holda filmlarni tavsiya qiladigan veb-xizmat. Foydalanuvchilar yangi filmlarni kashf etishi, tomosha ro'yxatini yaratishi va sun'iy intellekt yordamida shaxsiy tavsiyalar olishi mumkin.

---

### Funksional Talablar

| № | Talab | Tavsif |
|---|-------|--------|
| FT-1 | **Ro'yxatdan O'tish va Autentifikatsiya** | Foydalanuvchilar email/parol yoki OAuth (Google, GitHub) orqali ro'yxatdan o'tishi va tizimga kirishi mumkin bo'lishi kerak. Har bir foydalanuvchi o'z profilini boshqara olishi kerak. |
| FT-2 | **Film Qidirish va Ko'rish** | Foydalanuvchilar filmni nomi, janri, rejissyori, aktyori, chiqarilgan yili yoki reytingi bo'yicha qidira olishi kerak. Natijalarni filtrlash va saralash imkoniyati bo'lishi shart. |
| FT-3 | **Reyting va Sharh Tizimi** | Tizimga kirgan foydalanuvchilar filmlarga 1–10 ball bera olishi va ixtiyoriy matn sharhi yoza olishi kerak. Reytinglar tavsiyalar sifatini yaxshilash uchun ishlatilishi lozim. |
| FT-4 | **Shaxsiy Film Tavsiyalari** | Tizim har bir foydalanuvchi uchun tomosha tarixi, reytinglari va janr xohishlariga asoslanib shaxsiy film tavsiyalari tayyorlashi kerak (masalan, kollaborativ filtrlash yoki kontent asosidagi filtrlash algoritmi orqali). |
| FT-5 | **Tomosha Ro'yxatini Boshqarish** | Foydalanuvchilar filmlarni shaxsiy ro'yxatga qo'sha olishi, "ko'rildi" deb belgilashi va o'chirishi kerak. Ro'yxat barcha qurilmalar va sessiyalarda saqlanishi lozim. |

---

### Funksional Bo'lmagan Talablar

| № | Talab | Tavsif |
|---|-------|--------|
| NFT-1 | **Tezlik** | Tizim qidiruv natijalari va tavsiyalarni oddiy yuklanish sharoitida so'rovlarning 95% uchun **2 soniya ichida** qaytarishi kerak. Keshdan olingan tavsiyalar **500ms** dan tez bo'lishi lozim. |
| NFT-2 | **Masshtablilik** | Platforma kamida **100 000 bir vaqtda faol foydalanuvchi**ni qo'llab-quvvatlashi va trafik keskin oshganda (masalan, mashhur film chiqarilganda) gorizontal ravishda kengaya olishi kerak. |
| NFT-3 | **Mavjudlik** | Tizim yiliga **99.9% ishlash vaqtini** (yiliga ≤ 8.7 soat ishlamay qolish) ta'minlashi shart. Autentifikatsiya va tavsiya xizmatlari kabi muhim komponentlar zaxira mexanizmlariga ega bo'lishi kerak. |
| NFT-4 | **Xavfsizlik** | Barcha foydalanuvchi ma'lumotlari uzatishda (TLS 1.3) va saqlashda (AES-256) shifrlangan bo'lishi kerak. Autentifikatsiya muddati cheklangan JWT tokenlardan foydalanishi lozim. Tizim keng tarqalgan zaifliklardan (OWASP Top 10: XSS, SQL Injection, CSRF va boshqalar) himoyalangan bo'lishi shart. |
| NFT-5 | **Qo'llab-Quvvatlanish** | Kod bazasi aniq mas'uliyatlar ajratilgan modul arxitekturasiga rioya qilishi kerak. Har bir xizmatning birlik testi qamrovi **≥ 80%** bo'lishi lozim. Tizim rolling update yoki blue-green deployment orqali to'xtovsiz yangilanishlarni qo'llab-quvvatlashi kerak. |

---

## 2-Bosqich. Arxitektura

Film Tavsiya Platformasi uchun ikkita arxitektura varianti taklif etiladi.

---

## A Variant — Monolit Arxitektura (Qatlamli MVC)

### Tizim Diagrammasi

```
┌─────────────────────────────────────────────────────────────┐
│                      MIJOZ QATLAMI                          │
│              Brauzer / Mobil Ilova (React SPA)              │
└──────────────────────────┬──────────────────────────────────┘
                           │  HTTPS
┌──────────────────────────▼──────────────────────────────────┐
│                   MONOLIT ILOVA SERVERI                     │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                  TAQDIMOT QATLAMI                     │  │
│  │        REST API Kontrollerlar / Ko'rinish Shablonlari │  │
│  └───────────────────────┬───────────────────────────────┘  │
│                          │                                  │
│  ┌───────────────────────▼───────────────────────────────┐  │
│  │               BIZNES MANTIQ QATLAMI                   │  │
│  │  AuthService │ FilmService │ TavsiyaService           │  │
│  │  FoydalanuvchiService │ ReyingService │ RoʼyxatService│  │
│  └───────────────────────┬───────────────────────────────┘  │
│                          │                                  │
│  ┌───────────────────────▼───────────────────────────────┐  │
│  │             MA'LUMOTLARGA KIRISH QATLAMI              │  │
│  │             ORM (Hibernate / Sequelize)               │  │
│  └───────────────────────┬───────────────────────────────┘  │
└──────────────────────────┼──────────────────────────────────┘
                           │
         ┌─────────────────┼──────────────┐
         ▼                 ▼              ▼
   ┌───────────┐    ┌────────────┐  ┌──────────┐
   │PostgreSQL │    │   Redis    │  │  Fayl    │
   │ (Asosiy   │    │  (Kesh)    │  │Saqlash   │
   │    MB)    │    │            │  │  (S3)    │
   └───────────┘    └────────────┘  └──────────┘
```

### Afzalliklari

- **Ishlab chiqish va joylashtirishning soddaligi** — barcha kod bitta repozitoriyada; `docker-compose up` buyrug'i hamma narsani ishga tushiradi.
- **Operatsion xarajatlar past** — xizmatlar orasidagi tarmoq, xizmat topish yoki tarqatilgan kuzatuv boshqaruviga ehtiyoj yo'q.
- **Nosozliklarni tuzatish oson** — bitta stek orqali muammoni loglarda kuzatish qulay.
- **Boshlang'ich ishlab chiqish tezligi** — MVP ni tez qurmoqchi bo'lgan kichik jamoalar uchun mos.
- **Ichki chaqiruvlarda kam kechikish** — barcha xizmatlar o'rtasidagi aloqa xotirada amalga oshadi, tarmoq yuklanishini kamaytiradi.

### Kamchiliklari

- **Masshtablash qiyin** — faqat tavsiya mexanizmi yuklanib qolgan bo'lsa ham, butun ilova birlikda kengaytirilishi kerak.
- **Mustahkam bog'liqlik** — bir modulda (masalan, tavsiyalarda) xato butun ilovani ishdan chiqarishi mumkin.
- **Sekin joylashtirish** — har qanday kichik o'zgarish uchun butun ilovani qayta joylashtirishga to'g'ri keladi.
- **Texnologiyalarga bog'liqlik** — barcha modullar bir xil til va freymvorkdan foydalanishi shart.
- **Katta jamoa uchun qiyin** — ishlab chiquvchilar soni oshgach, kod ziddiyatlari ko'payadi.

---

## B Variant — Mikroservislar Arxitekturasi

### Tizim Diagrammasi

```
┌─────────────────────────────────────────────────────────────┐
│                      MIJOZ QATLAMI                          │
│              Brauzer / Mobil Ilova (React SPA)              │
└──────────────────────────┬──────────────────────────────────┘
                           │  HTTPS
┌──────────────────────────▼──────────────────────────────────┐
│                       API SHLYUZI                           │
│       (Tezlik Cheklash · Auth Tekshirish · Yo'naltirish)    │
└───┬──────────┬──────────┬──────────┬──────────┬─────────────┘
    │          │          │          │          │
    ▼          ▼          ▼          ▼          ▼
┌───────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌──────────────┐
│ Auth  │ │Foyda-  │ │ Film   │ │Reyting │ │   Tavsiya    │
│Xizmat │ │lanuvchi│ │Xizmati │ │Xizmati │ │   Xizmati    │
│       │ │Xizmati │ │        │ │        │ │ (ML Engine)  │
└───┬───┘ └───┬────┘ └───┬────┘ └───┬────┘ └──────┬───────┘
    │         │          │          │              │
    ▼         ▼          ▼          ▼              ▼
┌───────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌──────────────┐
│Postgre│ │Postgre │ │Postgre │ │Postgre │ │ Redis Kesh   │
│  SQL  │ │  SQL   │ │  SQL   │ │  SQL   │ │ + Vektor MB  │
│(Foyda)│ │(Profil)│ │(Filmlar│ │(Reyt.) │ │              │
└───────┘ └────────┘ └────────┘ └────────┘ └──────────────┘

              ┌──────────────────────────────┐
              │        Xabar Brokeri         │
              │      (Kafka / RabbitMQ)      │
              │  Hodisalar: FoydalanuvchiBaho│
              │  FilmKo'rildi va boshqalar   │
              └──────────────────────────────┘

              ┌──────────────────────────────┐
              │   Konteyner Orkestrasiyasi   │
              │         (Kubernetes)         │
              └──────────────────────────────┘
```

### Afzalliklari

- **Mustaqil masshtablash** — eng ko'p hisoblash talab qiladigan Tavsiya Xizmati GPU tugunlarida boshqa xizmatlarni buzmasdan alohida kengaytirilishi mumkin.
- **Xatoliklarni izolyatsiya qilish** — Reyting Xizmati ishlamay qolsa, film qidirish va tavsiyalar ishlashda davom etadi.
- **Texnologik erkinlik** — ML mexanizmi Python (scikit-learn, PyTorch) da, API shlyuzi esa Node.js da yozilishi mumkin.
- **Tezroq joylashtirish** — alohida xizmatlar mustaqil joylashtirilishi mumkin, bu zero-downtime CI/CD jarayonlarini ta'minlaydi.
- **Jamoa mustaqilligi** — turli jamoalar turli xizmatlarni boshqara oladi, muvofiqlashtirish xarajatlari kamayadi.

### Kamchiliklari

- **Yuqori operatsion murakkablik** — Kubernetes, xizmat tarmog'i, tarqatilgan kuzatuv (Jaeger), markazlashtirilgan loglash (ELK Stack) va xizmat topish mexanizmlari talab etiladi.
- **Kechikish oshadi** — xizmatlar orasidagi tarmoq qo'ng'iroqlari ichki chaqiruvlarga nisbatan ortiqcha vaqt oladi.
- **Tarqatilgan tizim qiyinchiliklari** — yakuniy izchillik (eventual consistency), tranzaksiyalar uchun saga naqshlari va circuit breaker mexanizmlari amalga oshirilishi lozim.
- **Infratuzilma xarajatlari yuqori** — ko'proq konteynerlar, ma'lumotlar bazalari va orkestrasiya vositalari, ayniqsa dastlabki bosqichlarda, bulut xarajatlarini oshiradi.
- **Nosozliklarni tuzatish murakkab** — bitta foydalanuvchi so'rovini 5+ xizmat bo'ylab kuzatish korrelyatsiya IDlari va tarqatilgan kuzatuv vositalarini talab qiladi.

---

## Taqqoslash va Tavsiya

| Mezon | A Variant (Monolit) | B Variant (Mikroservislar) |
|-------|--------------------|-----------------------------|
| Ishlab chiqish tezligi | ✅ Tez | ⚠️ Dastlab sekinroq |
| Masshtablilik | ❌ Cheklangan | ✅ A'lo darajada |
| Nosozliklarga chidamlilik | ❌ Yagona muvaffaqiyatsizlik nuqtasi | ✅ Izolyatsiyalangan xatolar |
| Operatsion murakkablik | ✅ Past | ❌ Yuqori |
| Jamoa hajmiga mosligi | Kichik (1–5 dasturchi) | O'rta-Katta (5+ dasturchi) |
| Joylashtirish moslashuvchanligi | ❌ To'liq qayta joylashtirish | ✅ Xizmat bo'yicha joylashtirish |
| MVP uchun tavsiya | ✅ Ha | ⚠️ Ortiqcha murakkab |
| Kengayish uchun tavsiya | ❌ Yo'q | ✅ Ha |

### ✅ Tavsiya Etilgan Variant: **B Variant — Mikroservislar Arxitekturasi**

**Asoslanish:**

Film Tavsiya Platformasining komponentlari tubdan farqli hisoblash profillariga ega. **Tavsiya Mexanizmi** CPU/GPU talab qiladigan va katta foydalanuvchi-element matritsalarini qayta ishlashi kerak, **Film Katalogi** xizmati esa oddiy ko'p o'qiladigan so'rovlarni bajaradi. Monolit arxitekturada faqat ML mexanizmi yuklanib qolsa ham, butun ilova birlikda kengaytirilishi kerak — bu ham qimmat, ham samarasiz.

Bundan tashqari, platforma vaqt o'tishi bilan foydalanuvchi bazasini oshirishi kutilmoqda (NFT-2: 100 000+ bir vaqtda faol foydalanuvchi). Mikroservislar Kubernetes HPA (Gorizontal Pod Avtomasshtablovchi) orqali alohida xizmatlarni avtomatik kengaytirish imkonini beradi.

Nihoyat, **nosozliklarga chidamlilik** talabi (NFT-3: 99.9% ishlash vaqti) mikroservislar bilan ancha osonroq ta'minlanadi — Reyting Xizmatidagi nosozlik film qidirish yoki foydalanuvchi autentifikatsiyasini ishdan chiqarmaydi.

> **Eslatma:** Dastlabki ishlab chiqish (MVP bosqichi) uchun **modul monolit** bilan boshlash va trafik oshgani sayin xizmatlarni bosqichma-bosqich ajratib chiqarish tavsiya etiladi. Bu A variantning ishlab chiqish tezligini B variantning keyingi masshtablilik imkoniyatlari bilan birlashtiradi.
