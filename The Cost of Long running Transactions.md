# The Cost of Long-running Transactions

Postgres’da (yoki boshqa har qanday database’da) uzoq davom etgan va oxirida failed bo‘lgan update transaction juda katta cost keltirib chiqarishi mumkin.

Postgres’da har bir DML transaction row’ga tegsa, o‘sha row’ning yangi version’i yaratiladi. Agar row index’larda referenced bo‘lsa, index’lar ham yangi tuple id bilan update qilinishi kerak bo‘ladi. Ba’zi optimization’lar mavjud, masalan HOT (Heap Only Tuple). HOT holatida index update qilinmaydi. Lekin bu faqat row joylashgan page’da yetarli bo‘sh joy bo‘lsa ishlaydi (fill factor < 100%).

Agar millionlab row’larni update qilgan uzun transaction rollback bo‘lsa, transaction yaratgan barcha yangi row version’lar invalid holatga o‘tadi va yangi transaction’lar tomonidan o‘qilmasligi kerak bo‘ladi.

Buni hal qilishning bir nechta yo‘li bor:

- rollback vaqtida barcha dead row’larni eager tarzda tozalash,
- yoki lazy tarzda keyinroq cleanup qilish,
- yoki table’ni lock qilib, database restart bo‘lguncha cleanup qilish.

Postgres lazy approach’dan foydalanadi. Buning uchun vacuum degan command mavjud va u periodik ravishda ishlaydi. Vacuum dead row’larni olib tashlashga va page ichidagi space’ni bo‘shatishga harakat qiladi.

Dead row’larni qoldirishning zarari nimada?

Bu correctness muammosi emas. Transaction’lar dead row’larni o‘qimaslikni biladi, chunki ular row’ni yaratgan transaction state’ini tekshiradi — committed yoki rolled back ekanligini.

Lekin bu check qimmat operation hisoblanadi. Database har safar row’ni o‘qiganda transaction state’ini tekshirishi kerak bo‘ladi.

Yana bir muammo shundaki, dead row’lar live row’lar bilan bir xil disk page ichida yashaydi. Bu esa IO efficiency’ni yomonlashtiradi.

Masalan:

- bitta page’da 1000 ta row bor,
- ulardan faqat 1 tasi live,
- 999 tasi dead.

Database baribir butun page’ni disk’dan o‘qiydi, lekin foydali ma’lumot sifatida atigi 1 ta row oladi.

Bu holat ko‘p takrorlansa:

- ko‘proq IO bo‘ladi,
- ko‘proq disk read bo‘ladi,
- performance sekinlashadi.

Ba’zi boshqa database’lar eager approach’dan foydalanadi va rollback to‘liq tugamaguncha database’ni ishga tushirishga ham ruxsat bermaydi. Bunda undo log’lardan foydalaniladi.

Qaysi approach to‘g‘ri?

Eng qiziq joyi shuki — hech biri mutlaqo to‘g‘ri yoki noto‘g‘ri emas.

Bular engineering decision’lar.

Hammasi fundamental tradeoff’lar.

Muhimi:

- system qanday ishlashini tushunish,
- qaysi approach nima cost keltirishini bilish,
- va requirement’ga qarab to‘g‘ri tanlov qilish.
