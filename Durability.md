# Durability

**Durability —** bu o‘zbekchada chidamlilik degani bo‘ladi, ammo bu unchalik ham to‘g‘ri emas, qisman to‘g‘ri. Bu nima degani o‘zi: biz databazaga ma’lumotni yozib bo‘lganimizdan so‘ng svet o‘chib qolsa, databaza qulasa, uni qayta yoqqanimizda ma’lumotlar shu yerda turishi kerak. Masalan, siz databazaga tranzaksiyani commit qildingiz, ammo shu paytda hamma joyda internet uzildi — internet tiklangandan so‘ng u ma’lumotlar turgan bo‘lishi kerak, chunki ular diskda saqlanadi.

### Durability texnikalari

1. **WAL (write ahead log)** — bu holatda har bir o‘zgarish birinchi bo‘lib WAL ga boradi va bu bizga ma’lumotda bir xillikni ta’minlashga yordam beradi. Ya’ni biz buni hali databazaga yozishga ulgurmadik, ammo biror holat bo‘lsa (databaza qulasa, svet o‘chib qolsa) ma’lumotni qayta tiklay olamiz va avvalgidek o‘zgartira olamiz. Bu usulning asosiy sababi: qattiq diskka to‘g‘ridan to‘g‘ri ko‘p ma’lumot (indexes, data files, columns, rows) yozish juda ko‘p vaqt oladi, va bizga yaxshiroq yo‘l kerak. DBMS o‘zgarishlarning compress qilingan versiyasini WAL da saqlaydi va u databazaga nimani qaysi qiymatga o‘zgartirishni aytib turadi.
2. **Asynchronous snapshot** — bu holatda biz ma’lumotlarni snapshot holida xotirada saqlaymiz va asynchronous holatda uni orqa tarafdan diskka yozib boramiz.
3. **AOF** — bu ham WAL ga o‘xshaydi. WAL o‘zgarishlarni oldin logga yozib, xotiradagi o‘zgarishlarni keyin boshlaydi; AOF da esa bu o‘zgarishlar avval xotirada amalga oshiriladi va keyin loglarga yozib qo‘yiladi (Redis ishlatadi).

OS cache muammosi — buni hal qilish uchun **fsync** ni ishlatamiz.
