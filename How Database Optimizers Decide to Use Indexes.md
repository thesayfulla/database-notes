# How Database Optimizers Decide to Use Indexes

Database optimizer (query planner)
har query uchun eng arzon execution plan tanlashga harakat qiladi.

Optimizer:

- index ishlatadimi
- table scan qiladimi
- bitmap scan qiladimi

shuni cost orqali hisoblaydi.

---

# Optimizer nima qiladi?

Misol:

SELECT *
FROM users
WHERE email = 'a@gmail.com';

Optimizer o‘ylaydi:

"Index ishlatish arzonmi
yoki full table scan?"

---

# Optimizer asosiy maqsadi

Eng kichik:

- CPU cost
- Disk I/O
- Memory usage

bilan query’ni bajarish.

---

# Optimizer nimaga qaraydi?

Asosiy factor’lar:

1. Table size
2. Selectivity
3. Statistics
4. Random vs Sequential I/O
5. Rows estimate
6. Index correlation
7. Visibility
8. Cost parameters

---

# 1. Table Size

Agar table kichkina bo‘lsa:

Seq Scan
ko‘pincha tezroq.

Sababi:

Butun table’ni o‘qish arzon.

---

# Misol

100 row table.

Index ishlatish:

- index read
- heap lookup

qiladi.

Bu ba’zan full scan’dan qimmat.

---

# 2. Selectivity

Eng muhim concept.

Selectivity =
query qancha row qaytaradi.

---

# High Selectivity

Kam row qaytadi.

Misol:

WHERE id = 10

yoki:

WHERE email = 'x'

Bu yaxshi candidate:
→ Index Scan

---

# Low Selectivity

Ko‘p row qaytadi.

Misol:

WHERE is_active = true

Agar 95% row active bo‘lsa:

Index foydasiz.

→ Seq Scan

---

# Nega?

Index ishlatilsa:

index -> heap
index -> heap
index -> heap

million marta random access bo‘ladi.

Seq Scan arzonroq chiqadi.

---

# 3. Statistics

Postgres statistics saqlaydi.

ANALYZE ishlaganda:

- distinct values
- null count
- histogram
- data distribution

yig‘iladi.

---

# Misol

country column:

UZ = 90%
US = 5%
JP = 5%

Query:

WHERE country = 'UZ'

→ Seq Scan ehtimoli katta.

---

# ANALYZE muhimligi

Agar statistics eski bo‘lsa:

optimizer noto‘g‘ri plan tanlashi mumkin.

---

# 4. Random vs Sequential I/O

Disk uchun:

Sequential read
Random read’dan arzonroq.

---

# Index Scan

Random I/O ko‘p qiladi.

---

# Seq Scan

Sequential I/O ishlatadi.

---

# Bitmap Scan

Ikkalasining o‘rtasi.

---

# 5. Rows Estimate

Optimizer oldindan taxmin qiladi:

"Nechta row qaytadi?"

---

# Misol

Estimated rows:
10

→ Index Scan

---

Estimated rows:
5 million

→ Seq Scan

---

# 6. Correlation

Data table’da qanday joylashganini bildiradi.

---

# High Correlation

Agar table:

ORDER BY created_at

bo‘yicha insert qilingan bo‘lsa,

created_at index
heap bilan yaxshi correlated bo‘ladi.

Index Scan tez ishlaydi.

---

# Low Correlation

Heap random joylashgan bo‘lsa:

Index Scan ko‘p random I/O qiladi.

---

# 7. Visibility Map

Index Only Scan uchun muhim.

Agar page:
"all-visible"

bo‘lsa,

heap access kerak bo‘lmaydi.

---

# VACUUM roli

VACUUM visibility map’ni yangilaydi.

Bu:
Index Only Scan’ni tezlashtiradi.

---

# 8. Cost Parameters

Postgres cost model ishlatadi.

Muhim parameter’lar:

- random_page_cost
- seq_page_cost
- cpu_tuple_cost
- cpu_index_tuple_cost

---

# random_page_cost

Random disk access narxi.

Katta bo‘lsa:
optimizer index’dan qo‘rqadi.

---

# SSD vs HDD

HDD:
random I/O juda qimmat.

SSD:
random I/O ancha arzon.

Shuning uchun SSD’da
Index Scan ko‘proq foydali.

---

# Real Example

Table:
10 million row

Query:

SELECT *
FROM orders
WHERE order_id = 100;

→ Index Scan

Sababi:
1 row qaytadi.

---

# Boshqa Example

SELECT *
FROM orders
WHERE status = 'completed';

Agar:
90% completed bo‘lsa

→ Seq Scan

---

# Bitmap Scan qachon?

Agar:

10-20% row qaytsa

planner ko‘pincha:

Bitmap Index Scan
+
Bitmap Heap Scan

tanlaydi.

---

# EXPLAIN ishlatish

Optimizer qarorini ko‘rish uchun:

EXPLAIN ANALYZE

ishlatiladi.

---

# Misol

EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'x';

---

# Natija

Index Scan using idx_users_email

yoki:

Seq Scan on users

yoki:

Bitmap Heap Scan

---

# Noto‘g‘ri plan sabablari

1. Eski statistics
2. Yomon index
3. Low selectivity
4. Wrong cost settings
5. Data skew

---

# Data Skew

Data notekis taqsimlangan bo‘lsa.

Misol:

90% UZ
10% others

Optimizer ba’zan noto‘g‘ri estimate qiladi.

---

# Optimizer hech qachon "hardcoded" emas

U:

- cost hisoblaydi
- taxmin qiladi
- eng arzon plan’ni tanlaydi

Bu probabilistic system.

---

# Mental Model

Optimizer har query uchun:

"Qaysi yo‘l kamroq disk va CPU ishlatadi?"

degan savolni yechadi.

---

# Qisqa Xulosa

Optimizer quyilarga qaraydi:

- table size
- selectivity
- statistics
- random vs sequential I/O
- estimated rows
- correlation
- visibility map
- cost settings

---

# Muhim Eslab Qol

Index mavjudligi
uni ishlatadi degani emas.

Optimizer uchun:

"Index ishlatish arzonmi?"

shu savol eng muhim.
