Type: Indexing
# Index nima?

---

Index — bu data’ni tez qidirish uchun yaratiladigan
alohida sorted structure.

Misol:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Bu yerda Postgres email bo‘yicha
sorted B-Tree yaratadi.

---

# Key Column nima?

Key column — index sorting va searching’da
qatnashadigan column.

Misol:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Bu yerda:

Key Column:

- email

Sababi:
index email bo‘yicha sorted bo‘ladi.

Taxminan:

```
[a@gmail.com](mailto:a@gmail.com)
```

```
[b@gmail.com](mailto:b@gmail.com)
```

```
[c@gmail.com](mailto:c@gmail.com)
```

---

# Key Column vazifasi

Key column:

- search qiladi
- sorting qiladi
- B-Tree navigation qiladi
- WHERE uchun ishlaydi
- ORDER BY uchun ishlaydi
- JOIN uchun ishlaydi

---

# Index ichida nima saqlanadi?

Taxminan:

email -> row pointer (TID)

Misol:

[a@gmail.com](mailto:a@gmail.com) -> row 10
[b@gmail.com](mailto:b@gmail.com) -> row 25

Index actual data emas.

U faqat:
"value -> location"

mapping saqlaydi.

---

# Query ishlashi

```sql
SELECT *
FROM users
WHERE email = 'a@gmail.com';
```

Postgres:

1. Index’dan email topadi
2. Row location oladi
3. Heap(table)ga boradi
4. Actual row’ni o‘qiydi

---

# Non-Key Column nima?

Non-key column —
index ichida saqlanadigan,
lekin sorting/search uchun ishlatilmaydigan
column.

PostgreSQL’da INCLUDE orqali qo‘shiladi.

---

# INCLUDE misoli

```sql
CREATE INDEX idx_users_email
ON users(email)
INCLUDE(name, age);
```

Bu yerda:

Key Column:

- email

Non-Key Columns:

- name
- age

---

# INCLUDE qanday ishlaydi?

Index ichida taxminan:

[a@gmail.com](mailto:a@gmail.com) -> Ali, 25
[b@gmail.com](mailto:b@gmail.com) -> Vali, 30

Lekin:

- name sorting qilmaydi
- age searching qilmaydi

Ular faqat extra payload.

---

# Nima uchun INCLUDE kerak?

Query:

```sql
SELECT name, age
FROM users
WHERE email = 'a@gmail.com';
```

Agar INCLUDE bo‘lmasa:

index -> heap(table)

borish kerak bo‘ladi.

INCLUDE bo‘lsa:

kerakli data index ichida bo‘ladi.

---

# Index Only Scan

Agar kerakli column’larning hammasi
index ichida bo‘lsa:

Postgres heap’ga bormaydi.

Bu:

Index Only Scan

deyiladi.

Bu juda tez ishlaydi.

---

# Composite Key Columns

CREATE INDEX idx_users
ON users(country, city);

Key Columns:

1. country
2. city

Sorting:

(country, city)

bo‘yicha bo‘ladi.

---

# Left-most Prefix Rule

(country, city)

index quyidagi query’lar uchun yaxshi:

WHERE country = 'UZ'

yoki:

WHERE country = 'UZ'
AND city = 'Tashkent'

Lekin:

WHERE city = 'Tashkent'

uchun yaxshi emas.

Sababi:
sorting country bilan boshlanadi.

---

# INCLUDE vs Composite Index

CREATE INDEX idx1
ON users(country, city);

Bu yerda:

- country search qiladi
- city ham search qiladi

---

CREATE INDEX idx2
ON users(country)
INCLUDE(city);

Bu yerda:

- country search qiladi
- city faqat payload

city sorting/search’da qatnashmaydi.

---

# Key Column qachon ishlatiladi?

Ko‘p ishlatiladigan:

- WHERE
- JOIN
- ORDER BY

column’lar key column bo‘lishi kerak.

---

# INCLUDE qachon ishlatiladi?

Ko‘p SELECT qilinadigan,
lekin filtering qilinmaydigan
column’lar uchun ishlatiladi.

---

# Afzalliklari

Key Columns:

- fast searching
- fast sorting

INCLUDE:

- heap access kamayadi
- Index Only Scan ishlaydi
- disk I/O kamayadi

---

# Kamchiliklari

INCLUDE juda ko‘p ishlatilsa:

- index kattalashadi
- RAM ko‘proq ishlatiladi
- INSERT/UPDATE sekinlashadi

---

# Real Hayot Analogiyasi

Key Column:

- kutubxonadagi shelf tartibi

Non-Key Column:

- kitob ichidagi qo‘shimcha ma’lumot

---

# Qisqa Xulosa

Key Column:

- sorting
- searching
- B-Tree navigation

Non-Key Column:

- extra payload
- covering query
- Index Only Scan

---

# Muhim Eslab Qolish

B-Tree faqat key column bilan ishlaydi.

INCLUDE columns:
"carry-along data"

xolos.