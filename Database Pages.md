# Database Pages

Type: Database internal

Databaselar ko‘pincha ma’lumotlarni saqlash uchun fixed-size pagelardan foydalanadi. Tables, collections, rows, columns, indexes, sequences, documents va boshqa obyektlar oxir-oqibat byte’lar ko‘rinishida page ichida saqlanadi. Shu orqali storage engine database frontend’dan ajraladi. Frontend data format va API bilan ishlasa, storage engine esa page’lar bilan ishlaydi. Bundan tashqari, hamma narsa page asosida bo‘lgani uchun data’ni read, write yoki cache qilish ancha osonlashadi.

Masalan, SQL Server’da page layout quyidagicha ishlaydi.

## A Pool of Pages

Databaselar read va write operatsiyalarini page’lar orqali bajaradi. Siz table’dan biror row o‘qisangiz, database avval shu row qaysi page ichida joylashganini topadi. Keyin shu page qaysi file ichida va diskdagi qaysi offset’da turganini aniqlaydi.

Shundan so‘ng database OS’dan:

- file nomi,
- offset,
- va page size

asosida ma’lumotni o‘qishni so‘raydi.

OS avval filesystem cache’ni tekshiradi. Agar kerakli data cache’da bo‘lmasa, diskdan page o‘qilib memory’ga yuklanadi va database’ga qaytariladi.

Database odatda o‘ziga maxsus memory pool ajratadi. Bu ko‘pincha:

- shared pool
- yoki buffer pool

deb ataladi.

Diskdan o‘qilgan page’lar shu buffer pool ichiga joylanadi.

Biror page memory’da paydo bo‘lgach:

- faqat kerakli row emas,
- balki shu page ichidagi boshqa row’lar ham memory’da bo‘ladi.

Bu ayniqsa index range scan paytida juda foydali.

Row qanchalik kichik bo‘lsa:

- bitta page ichiga ko‘proq row sig‘adi,
- bitta I/O operatsiyadan ko‘proq foyda olinadi.

### Write jarayoni

Write ham xuddi shunga o‘xshash ishlaydi.

User row update qilsa:

1. Database row joylashgan page’ni topadi.
2. Page buffer pool’ga yuklanadi.
3. Row memory ichida update qilinadi.
4. O‘zgarish haqida WAL (Write Ahead Log) yoziladi va diskka persist qilinadi.

Page darhol diskka flush qilinmasligi mumkin. U memory’da qoladi va yana boshqa write’larni ham qabul qiladi. Keyinchalik bir marta flush qilinadi. Bu esa I/O sonini kamaytiradi.

Delete va insert ham shu konsepsiyada ishlaydi, lekin implementation database’ga qarab farq qiladi.

---

# Page Content

Page ichida nima saqlash database design’iga bog‘liq.

## Row-store databases

Bunday database’lar row’larni barcha attribute’lari bilan ketma-ket page ichida saqlaydi.

Masalan:

| id | name | age |
| --- | --- | --- |
| 1 | Ali | 20 |

diskda ketma-ket yoziladi.

Bu OLTP workload uchun juda qulay, ayniqsa write workload’da.

---

## Column-store databases

Bu yerda esa ma’lumot column bo‘yicha saqlanadi.

Masalan:

```
id column:
1 2 3 4

name column:
Ali Vali Sami
```

Bu OLAP workload uchun juda foydali.

Masalan:

```
SELECT SUM(price)
```

faqat kerakli column o‘qiladi. Bitta page ichida aynan bitta column qiymatlari packed holda saqlangani uchun aggregate funksiyalar juda tez ishlaydi.

---

## Document databases

Document-based database’lar:

- document’larni compress qiladi,
- va page ichida saqlaydi.

Bu row-store’ga o‘xshaydi.

---

## Graph databases

Graph database’lar esa node va edge connectivity’ni page ichida shunday saqlaydiki:

- graph traversal samarali bo‘ladi,
- depth-first yoki breadth-first search optimallashtiriladi.

---

Asosiy maqsad:

> Page ichiga iloji boricha foydali ma’lumot joylash.
>

Agar siz kichik ish uchun juda ko‘p page o‘qiyotgan bo‘lsangiz:

- data modeling noto‘g‘ri bo‘lishi mumkin.

Data modeling juda underrated mavzu.

---

# Small vs Large Pages

## Small pages

Kichik page’lar:

- tezroq read/write qilinadi,
- ayniqsa page size storage block size’ga yaqin bo‘lsa.

Lekin:

- page header metadata overhead oshib ketadi.

---

## Large pages

Katta page’lar:

- metadata overhead’ni kamaytiradi,
- page split’larni kamaytiradi.

Lekin:

- cold read qimmatlashadi,
- write ham qimmatroq bo‘ladi.

---

Storage industry bu muammoni hal qilish uchun:

- Zoned Storage,
- NVMe KV namespaces

kabi texnologiyalar ustida ishlamoqda.

---

Turli database’larning default page size’lari:

| Database | Default Page Size |
| --- | --- |
| PostgreSQL | 8KB |
| MySQL InnoDB | 16KB |
| MongoDB WiredTiger | 32KB |
| SQL Server | 8KB |
| Oracle | 8KB |

Default setting’lar ko‘p holatda yetarli bo‘ladi, lekin workload’ga qarab tuning qilish muhim.

---

# How page are stored on Disk

Page’larni diskda saqlashning turli usullari mavjud.

Oddiy usullardan biri:

- har bir table uchun alohida file yaratish,
- page’larni fixed-size array sifatida saqlash.

Misol:

```
Page0 | Page1 | Page2 | Page3
```

Agar bizga page X kerak bo‘lsa:

- file nomi table’dan olinadi,
- offset = X * PAGE_SIZE,
- read length = PAGE_SIZE.

---

Misol:

Page size = 8KB.

Page 2 dan 9 gacha o‘qish kerak bo‘lsa:

```
offset = 2 * 8192 = 16384
length = 8 * 8192 = 65536 bytes
```

Database shu offset’dan boshlab kerakli byte’larni o‘qiydi.

Lekin har bir database implementation’i boshqacha bo‘lishi mumkin.

---

# Postgres Page Layout

PostgreSQL default holatda **8KB** page ishlatadi.

Page quyidagi qismlardan iborat:

---

## Page header — 24 bytes

Bu page metadata’si.

Unda:

- free space,
- page state,
- boshqa metadata

saqlanadi.

Header fixed-size — 24 byte.

---

## ItemIds — har biri 4 byte

Bu actual tuple emas.

Bu:

- offset:length pointer array.

Har bir ItemId:

- tuple qayerda joylashganini,
- va tuple size’ini

ko‘rsatadi.

---

## HOT Optimization

Bu pointer’lar sabab PostgreSQL HOT (Heap Only Tuple) optimization qila oladi.

Agar row update qilinsa:

- yangi tuple yaratiladi,
- agar yangi tuple eski page ichiga sig‘sa,
- eski ItemId yangi tuple’ga redirect qilinadi.

Natijada:

- index’larni update qilish shart bo‘lmaydi,
- eski tuple id ishlashda davom etadi.

Bu juda kuchli optimization.

---

## Tanqid (Criticism)

Har bir ItemId 4 byte joy oladi.

Agar 1000 ta item bo‘lsa:

```
1000 * 4 = 4000 bytes
```

deyarli 4KB faqat pointer’larga ketadi.

Bu yarim page degani.

---

# Row vs Tuple vs Item

Bu tushunchalar farqli.

## Row

User ko‘radigan logical data.

---

## Tuple

Row’ning physical version’i.

---

## Item

Page ichidagi tuple entry.

---

Bitta row uchun:

- bir nechta tuple bo‘lishi mumkin.

Masalan:

- 1 active tuple,
- 7 eski MVCC tuple,
- 2 dead tuple.

MVCC sabab eski transaction’lar eski tuple’larni ko‘rishi mumkin.

---

# Items — variable length

Bu qismda actual tuple’lar joylashadi.

Tuple’lar ketma-ket saqlanadi.

---

# Special — variable length

Bu qism asosan B+Tree index leaf page’larda ishlatiladi.

Unda:

- previous page pointer,
- next page pointer

kabi ma’lumotlar saqlanadi.

---

# Summary

Database ichidagi hamma narsa oxir-oqibat page’larda yashaydi:

- tables,
- indexes,
- sequences,
- rows,
- documents.

Bu abstraction database’ga:

- data bilan bir xil usulda ishlash,
- caching,
- efficient read/write,
- buffer management

imkonini beradi.

Har bir database:

- page layout,
- storage format,
- disk organization

bo‘yicha farq qiladi.

Lekin asosiy g‘oya bir xil:

> Data fixed-size page’lar ichida saqlanadi va database shu page’lar bilan ishlaydi.
>
