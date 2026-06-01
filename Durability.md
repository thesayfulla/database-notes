# Durability

**Durability —** bu o'zbekchda chidamlilik degani bo'ladi ammo bu unchalik ham to'g'ri emas ammo qisman to'g'ri , Bu nima degani o'zi biz databazaga ma’lumotni yozib bo'lganimizdan so'ng svet o'chib qolsa databaza qulasa hamma uni qayta yoqqanimda ma’lumotlar shu yerda turishi kerak. Masalan siz databazaga tranzaksiyani commit qildingiz ammo shu paytda hamma joyda internet uzildi ammo internet tiklangandan so'ng u ma'lumotlar turgan bo'lishi kerak chunki ular diskda saqlanadi.

### **Durability texnikalari**

1. **WAL** **(write ahead log) —** bu holatda har bir o`zgarish birinchi bo`lib WAL ga boradi va bu bizga ma’lumotda bir xillikni taminlashga yordam beradi , ya’ni biz buni hali databazaga yozishga ulgurmadik ammo biror holat bo`lsa (databaza qulasa , svet o`chib qolsa) ma’lumotni qayta tiklay olamiz va avvalgidek o`zgartira olamiz .Buning usulning asosiy yechimi qattiq diskga to`g`ridan to`g`ri ko`p malumot(indexes , data files, columns, rows) yozish juda ko`p vaqt oladi , va bizga yaxshiroq yo`l kerak DBMS o`zgarishlarni compress qilingan versiyasini WAL da saqlaydi va u databazaga nimani qaysi qiymatga o`zgartirishni aytib turadi .
2. **Asynchronous snapshot** — bu holatda biz ma’lumotlarni snapshot holida xotirada saqlaymiz va asynchronous holatda uni orqa tarafdan diska yozib boramiz .
3. **AOF** — bu ham WAL ga o`xshaydi yani u o`zgarishlarni oldin logga yozib xotiradagai o`zgarishlarni keyin boshlaydi AOFda esa bu o`zgarishlar avval xotirada amalga oshiriladi va keyin loglarda yozib qo`yiladi (redis ishlatadi ).

OS Cache muammosi - buni hal qilish uchun biza **fsync** ni ishlatamiz
