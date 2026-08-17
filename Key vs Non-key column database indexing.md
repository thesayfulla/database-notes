# Key vs Non-key Column Database Indexing

## Index nima?

Index — bu data’ni tez qidirish uchun yaratiladigan alohida sorted structure.

Misol:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Bu yerda Postgres email bo‘yicha sorted B-Tree yaratadi.

## Key column nima?

Key column — index sorting va searching’da qatnashadigan column.

Misol:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Bu yerda key column:

- email

Sababi: index email bo‘yicha sorted bo‘ladi.

Taxminan:

```text
a@gmail.com
b@gmail.com
c@gmail.com
```

### Key column vazifasi

Key column:

- search qiladi
- sorting qiladi
- B-Tree navigation qiladi
- WHERE uchun ishlaydi
- ORDER BY uchun ishlaydi
- JOIN uchun ishlaydi

## Index ichida nima saqlanadi?

Taxminan:

```text
email -> row pointer (TID)
```

Misol:

```text
a@gmail.com -> row 10
b@gmail.com -> row 25
```

Index actual data emas.

U faqat "value -> location" mapping saqlaydi.

### Query ishlashi

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

## Non-key column nima?

Non-key column — index ichida saqlanadigan, lekin sorting/search uchun ishlatilmaydigan column.

PostgreSQL’da INCLUDE orqali qo‘shiladi.

### INCLUDE misoli

```sql
CREATE INDEX idx_users_email
ON users(email)
INCLUDE(name, age);
```

Bu yerda key column:

- email

Non-key columns:

- name
- age

### INCLUDE qanday ishlaydi?

Index ichida taxminan:

```text
a@gmail.com -> Ali, 25
b@gmail.com -> Vali, 30
```

Lekin:

- name sorting qilmaydi
- age searching qilmaydi

Ular faqat extra payload.

### Nima uchun INCLUDE kerak?

Query:

```sql
SELECT name, age
FROM users
WHERE email = 'a@gmail.com';
```

Agar INCLUDE bo‘lmasa, `index -> heap(table)` borish kerak bo‘ladi.

INCLUDE bo‘lsa, kerakli data index ichida bo‘ladi.

## Index Only Scan

Agar kerakli column’larning hammasi index ichida bo‘lsa, Postgres heap’ga bormaydi.

Bu Index Only Scan deyiladi.

Bu juda tez ishlaydi.

## Composite key columns

```sql
CREATE INDEX idx_users
ON users(country, city);
```

Key columns:

1. country
2. city

Sorting `(country, city)` bo‘yicha bo‘ladi.

### Left-most prefix rule

`(country, city)` index quyidagi query’lar uchun yaxshi:

```sql
WHERE country = 'UZ'
```

yoki:

```sql
WHERE country = 'UZ'
AND city = 'Tashkent'
```

Lekin:

```sql
WHERE city = 'Tashkent'
```

uchun yaxshi emas.

Sababi: sorting country bilan boshlanadi.

## INCLUDE vs Composite Index

```sql
CREATE INDEX idx1
ON users(country, city);
```

Bu yerda:

- country search qiladi
- city ham search qiladi

```sql
CREATE INDEX idx2
ON users(country)
INCLUDE(city);
```

Bu yerda:

- country search qiladi
- city faqat payload

city sorting/search’da qatnashmaydi.

## Key column qachon ishlatiladi?

Ko‘p ishlatiladigan:

- WHERE
- JOIN
- ORDER BY

column’lar key column bo‘lishi kerak.

## INCLUDE qachon ishlatiladi?

Ko‘p SELECT qilinadigan, lekin filtering qilinmaydigan column’lar uchun ishlatiladi.

## Afzalliklari

Key columns:

- fast searching
- fast sorting

INCLUDE:

- heap access kamayadi
- Index Only Scan ishlaydi
- disk I/O kamayadi

## Kamchiliklari

INCLUDE juda ko‘p ishlatilsa:

- index kattalashadi
- RAM ko‘proq ishlatiladi
- INSERT/UPDATE sekinlashadi

## Real hayot analogiyasi

Key column:

- kutubxonadagi shelf tartibi

Non-key column:

- kitob ichidagi qo‘shimcha ma’lumot

## Qisqa xulosa

Key column:

- sorting
- searching
- B-Tree navigation

Non-key column:

- extra payload
- covering query
- Index Only Scan

## Muhim eslab qolish

B-Tree faqat key column bilan ishlaydi.

INCLUDE columns — "carry-along data", xolos.
