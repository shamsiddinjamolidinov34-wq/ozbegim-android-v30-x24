# Fokus va navigatsiya yo'riqnomasi (O'zbegim 0.9.15, v30)

Bu hujjat uch narsani aytadi: (1) teginib o'rganish va haqiqiy amalni bajarish o'rtasidagi muvozanat qanday qoidalarga tayanadi,
(2) v29 dagi qaysi nuqsonlar topildi va qanday tuzatildi, (3) tuzatishlarni telefonda qanday tekshirish kerak.

**Muhim:** bu muhitda Gradle bilan qurib ham, telefonda sinab ham bo'lmaydi. Kod kompilyator darajasida (yangi xato yo'qligi) va
mantiqiy modellar bilan tekshirildi. Sabablar jurnal va kodni o'qish orqali topilgan; ularning hech biri telefonda tasdiqlanmagan.
3-bo'limdagi sinov tartibi shuning uchun yozilgan: haqiqiy hukmni telefon beradi.

## 1. Qanday ishlaydi va muvozanat qoidalari

Teginib o'rganish yoqiq paytda har bir teginish ikki bosqichli ishdir:

1. **Tanlash.** Barmoq element ustida yuradi, tizim `TYPE_VIEW_HOVER_ENTER` yuboradi, O'zbegim elementga **fokus** beradi va o'qiydi.
   Bu bosqichda hech narsa bosilmaydi.
2. **Bajarish.** Foydalanuvchi **istalgan joyga ikki marta bosadi**, amal aynan fokusdagi (eng so'nggi eshitilgan) elementda bajariladi.
   Ikki marta bosishni kim bajarishi sozlanadi: Tafsilotlar > Bosish va navigatsiya ("Android tizimi bajaradi" yoki "O'zbegim bajaradi").

Muvozanat shu besh qoidaga tayanadi. Har bir tuzatish ulardan birini himoya qiladi:

| # | Qoida | Buzilganda foydalanuvchi nimani sezadi |
|---|-------|----------------------------------------|
| 1 | **Fokus faqat foydalanuvchining o'zi yoki yangi ekran uchun siljiydi.** Ilovaning tasodifiy hodisasi fokusni tortib olmaydi. | "Fokus mo'ljaldagi tugmadan qochib ketdi" |
| 2 | **Ikki marta bosish nishoni = oxirgi eshitilgan element.** Eskirgan yoki boshqa ekrandagi element nishon bo'lmaydi. | "Nomini eshitdim, lekin boshqa narsa bosildi" |
| 3 | **Bir ikki marta bosish = bitta amal.** Takror kelgan juftlik o'tkazib yuboriladi. | Kalit ikki marta almashib, holat o'zgarmay qoladi |
| 4 | **Bosish natijasi eshitiladi.** Holat yoki tugma nomi o'zgarsa, aytiladi. | "Amal bajarilmadi" deb qayta-qayta bosish |
| 5 | **Asosiy oqim band bo'lmaydi.** Hodisa ishlovi qisqa: barmoq ostidagi elementga fokus kechikmaydi. | Eshitish kechikadi, ikki marta bosish eski fokusga tushadi |

## 2. Topilgan nuqsonlar va tuzatishlar

Ishonch darajasi: **K** = kodni o'qib aniqlangan, **J** = jurnal ham ko'rsatadi, **T** = telefonda tasdiqlangan (hozircha hech biri).

| # | Nuqson | Sabab | Tuzatish | Fayl | Ishonch |
|---|--------|-------|----------|------|---------|
| 1 | Fokus klaviatura yoki tizim paneli ustidan "qochadi" | `maybeInitialFocus` oyna hodisasi kelgan har safar, fokus boshqa oynada (klaviatura, holat paneli, pastki tugmalar) bo'lsa ham, uni faol oynaning eng yuqori elementiga tortib olardi | Fokus klaviatura va tizim oynalarida turgan bo'lsa, barmoq tegib turgan bo'lsa yoki faol oyna klaviatura bo'lsa, siljitilmaydi | `ReaderService.maybeInitialFocus`, `focusInForeignWindow` | K |
| 2 | Dialog yoki menyu yopilgach fokus ekranning eng tepasiga sakraydi | Yopilgan oynadan qaytganda har doim birinchi elementga o'tilardi | Har oyna uchun oxirgi fokus eslab qolinadi; boshqa oynadan qaytilsa va element hali ko'rinsa, fokus unga qaytadi | `rememberWindowFocus`, `rememberedFocusFor` | K |
| 3 | Eski ekrandagi element bosilib ketadi | "Yumshoq fokus" (tizim fokus bermagan, o'zimiz o'qigan element) 120 soniya saqlanardi; ilovaning manbasiz fokus hodisasi uni tozalamasdi; ekran almashganda ham qolardi | Muddat 45 soniya; manbasiz fokus hodisasi tozalaydi; ekran almashganda yangilab bo'lmaydigan yumshoq fokus tashlanadi | `onFocused`, `currentFocus`, `dropStaleSoftFocus` | K |
| 4 | Kalit ikki marta almashadi, tugma ikki marta bosiladi | "O'zbegim bajaradi" rejimida ikki marta bosish takror kelgan: jurnalda 63 ta bosish orasidagi 62 ta oraliqning 5 tasi 350 ms dan qisqa (72, 268, 269, 280 va 316 ms); odam uchun bunday tez takror deyarli mumkin emas | 350 ms ichida kelgan ikkinchi juftlik o'tkazib yuboriladi (`dt_dup`) | `handleGesture` | J |
| 5 | Ikki marta bosish hali qayta ishlanmagan oxirgi teginishga tushmaydi | Barmoq tez ko'tarilganda oxirgi hover hodisasi navbatda turgan bo'lishi mumkin | Ikki marta bosishdan oldin kutayotgan hover yakunlanadi | `activateByDoubleTap` | K |
| 6 | Bosdim, hech narsa eshitilmadi, shuning uchun qayta bosdim | Tugma nomi o'zgarsa ("Pauza" -> "Ijro"), O'zbegim faqat holat o'zgarganda gapirardi. Jurnalda YouTube'da 25 ta bosish bor, ulardan 12 tasi oldingisidan 1,5 soniya ichida (qayta bosish ehtimoli) | Bosishdan keyin 2,5 soniya ichida bosiladigan elementning o'z nomi o'zgarsa, yangi nom aytiladi (TalkBack kabi). Vaqt yoki hisoblagich kabi o'zi o'zgaradigan matn aytilmaydi | `checkStateChange`, `labelKey` | J |
| 7 | Eshitish va fokus kechikadi | Hodisa ishlovi daraxtni qayta-qayta yurardi (`related`/`isInside`): jurnalda 9 ta `SLOW_HANDLER`, eng uzuni 364 ms; ular hover ishlovini kutdirgan bo'lishi mumkin | Oyna raqami bo'yicha tez rad etish, rejalashtirilgan tekshiruv paytida yurmaslik, yurish chuqurligi 12 | `related`, `scheduleStateCheck`, `isInside` | K, J qisman |
| 8 | Select to Speak yoqilsa O'zbegim jim qoladi | Google paketidagi har qanday xizmat "TalkBack" hisoblanardi | Qaror xizmat (paket + sinf) bo'yicha; yordamchi xizmatlar hisobga olinmaydi (5-bo'lim) | `ReaderRules`, `OtherReaders` | K |
| 9 | Bosh ekrandagi eslatma | O'zbegim boshqa ekran o'quvchi bilan o'zi almashinadi, eslatma eskirgan | Olib tashlandi (maket va 4 tildagi matn) | `activity_main.xml`, `strings.xml` | |

## 3. Telefonda sinash tartibi

Har sinovdan keyin natijani yozib boring. Sinovlarni **ikkala rejimda** (Tafsilotlar > Bosish va navigatsiya) takrorlang: 5, 6 va 7-sinovlar rejimga bog'liq.

1. **TalkBack bilan almashinuv.** O'zbegim yoqiq, TalkBack o'chiq. Sozlamalardan TalkBack'ni yoqing.
   Kutiladi: bir-uch soniyada "Boshqa ekran o'quvchi ishni boshladi, men ekranni o'qishni to'xtatdim", keyin faqat TalkBack gapiradi.
   TalkBack'ni o'chiring. Kutiladi: bir necha soniyada "Men ekranni o'qishni boshladim, boshqa ekran o'quvchi dastur o'chirilganligi uchun".
2. **Select to Speak.** TalkBack o'chiq, Select to Speak'ni yoqing. Kutiladi: O'zbegim to'xtamaydi, odatdagidek o'qiydi.
3. **Klaviatura.** Xabar yozish maydonini oching, klaviatura chiqsin, tugmalarni barmoq bilan tekshiring, bir necha soniya kuting (ilova
   takliflar chiqarishi mumkin). Kutiladi: fokus klaviaturada qoladi, ilovaning yuqori qismiga sakramaydi.
4. **Dialog yoki menyu yopilishi.** "Qo'shimcha" (uch nuqta) menyusini oching, Orqaga bilan yoping. Kutiladi: fokus uch nuqta tugmasiga
   qaytadi, ekranning birinchi elementiga emas.
5. **Orqaga qaytish.** Sozlamalarda biror qatorni oching, Orqaga bosing. Kutiladi: fokus ochgan qatoringizda.
6. **Bosish natijasi.** YouTube'da pauza/ijro tugmasini ikki marta bosing; Wi-Fi kalitini ham. Kutiladi: yangi nom ("Ijro") yoki holat aytiladi.
7. **Takror.** "O'zbegim bajaradi" rejimida kalitni tez-tez ikki marta bosing. Kutiladi: har juftlik bitta almashtirish, tez takror o'tkazib yuboriladi.
8. **Oxirgi eshitilgan.** A elementga tegib eshiting, tezda B ga o'ting, eshiting, barmoqni ko'tarib ikki marta bosing. Kutiladi: B bosiladi.
9. **Pastki tugmalar.** Pastki (Orqaga, Bosh ekran) tugmalarni teginib toping, YouTube'da video o'ynab turganda. Kutiladi: fokus tugmada qoladi.
10. **Jurnal.** Sinovdan keyin "Xatolar jurnali" ekranidan ulashing (4-bo'lim bo'yicha o'qiladi).

## 4. Jurnalni o'qish (`GESTURE_PATH`)

| Yozuv | Ma'nosi |
|-------|---------|
| `dt_click`, `dt_global`, `dt_touch` | "O'zbegim bajaradi" rejimida bosish yo'li: element bosildi, tizim tugmasi bajarildi yoki sun'iy tegish yuborildi |
| `sys_click` (yangi) | "Android tizimi bajaradi" rejimida ilova bosish hodisasini yubordi. `rel=true`: bosilgan element fokusdagi element bilan qarindosh. `rel=false` ko'p bo'lsa, bosish eshitilgan elementga tushmayapti, ya'ni fokus qochgan. Jurnal hajmi cheklangan: `rel=false` 0,7 soniyada, `rel=true` 15 soniyada bir martadan ko'p yozilmaydi, shuning uchun yozuvlar sonini bosishlar soni deb o'qimang |
| `dt_dup` (qora quti) | Takror ikki marta bosish o'tkazib yuborildi |
| `SLOW_HANDLER` | Hodisa ishlovi 120 ms dan uzoq. Soni kamayganini solishtiring: v29 jurnalida 9 ta, eng uzuni 364 ms |

Eslatma: v29 jurnalida bosish yozuvlari (`dt_*`) bor, demak u paytda rejim "O'zbegim bajaradi" bo'lgan. "Tizimga berilganda ham qiyin" degan
kuzatuvingizni tizim rejimi jurnali hali ko'rsatmaydi (v29 da bunday bosish yozilmasdi). Shu sababli `sys_click` qo'shildi: keyingi
jurnalni tizim rejimida yozing.

## 5. Boshqa ekran o'quvchi bilan almashinish qoidalari

- **Xizmat bo'yicha qaror** (paket nomi bo'yicha emas). Ekran o'quvchi hisoblanadi: TalkBack (sinf nomi `TalkBackService`), Samsung TalkBack va
  Voice Assistant, Jieshuo, Dianming: ular to'xtatilgan (suspend) bo'lsa ham; noma'lum xizmat esa faqat hozir teginib o'rganishni talab qilsagina.
- **Hisobga olinmaydi:** Select to Speak, Switch Access, Voice Access, Accessibility Menu, Braille xizmatlari, Universal Switch, Assistant menu.
- **Yoqilganda tezroq:** sozlamalarda yoqilgan, lekin tizim hali ulamagan ma'lum ekran o'quvchi 15 soniyagacha hisobga olinadi (tez to'xtash uchun).
  15 soniyada ham ulanmasa, u ishlamayapti deb hisoblanadi va O'zbegim jim qolmaydi.
- **O'chirilganda tezroq:** sozlamalardan olib tashlangan xizmat tizimdan hali uzilmagan bo'lsa ham hisobga olinmaydi.
- O'zbegim kutish rejimidan chiqqanda ikki marta bosish sozlamasini qayta moslaydi (kutish paytida o'zgartirilgan bo'lishi mumkin).

## 6. Cheklovlar va keyingi qadamlar (halol)

- Bu muhitda Gradle qurilmasi va telefon yo'q: **qurilmadagi xatti-harakat tasdiqlanmagan.** Qurishda xato chiqsa, xato matnini yuboring.
- Ikki marta bosish standarti "Android tizimi bajaradi" bo'lib qoldi. Qaysi rejim sizning telefoningizda qulay ekanini 3-bo'limdagi sinovlar ko'rsatadi.
- O'zgartirilmadi: `dt_touch` sun'iy tegish yo'li (teginib o'rganishni vaqtincha o'chiradi); TTS `ERROR_OUTPUT` (-5) xatosi (jurnalda 1 marta);
  bosish tasdig'i uchun tovush yoki titrash qo'shilmadi; hover ishlovi asosiy oqimda qoldi.
- Agar sinovdan keyin ham fokus qochsa, jurnal va qaysi sinov (raqami) va rejimni yuboring: keyingi tuzatish shu ma'lumotga tayanadi.
