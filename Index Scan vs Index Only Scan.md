# Index Scan vs Index Only Scan

## 1. Index Scan nima?

Index Scan’da:

1. Postgres index’dan row location’ni topadi
2. Keyin heap(table)ga boradi
3. Actual row’ni o‘qiydi

Flow:

```text
Index -> Heap(Table)
```

### Misol

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email TEXT,
    name TEXT
);

CREATE INDEX idx_users_email
ON users(email);
```

Query:

```sql
SELECT *
FROM users
WHERE email = 'a@gmail.com';
```

### Nima bo‘ladi?

Postgres:

1. Index’dan `email = 'a@gmail.com'` ni topadi
2. TID (row pointer) oladi — masalan `'a@gmail.com' -> row 500`
3. Heap(table)ga boradi
4. Full row’ni o‘qiydi

### EXPLAIN

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'a@gmail.com';
```

Natija:

```text
Index Scan using idx_users_email
```

### Nega heap’ga boradi?

Chunki index ichida faqat `email -> row location` bor.

Lekin query `SELECT *` qilyapti. Actual data heap’da.

### Index Scan muammosi

Agar ko‘p row qaytsa:

```text
index -> heap
index -> heap
index -> heap
```

random disk access ko‘payadi.

Bu qimmat.

## 2. Index Only Scan nima?

Index Only Scan’da Postgres heap’ga umuman bormaydi.

Barcha kerakli data index ichida bo‘ladi.

### Misol

```sql
CREATE INDEX idx_users_email_include
ON users(email)
INCLUDE(name);
```

Query:

```sql
SELECT email, name
FROM users
WHERE email = 'a@gmail.com';
```

### Nima bo‘ladi?

Kerakli data:

- email
- name

ikkalasi ham index ichida mavjud.

Shuning uchun heap access kerak emas.

Flow:

```text
Index
```

### EXPLAIN

```sql
EXPLAIN ANALYZE
SELECT email, name
FROM users
WHERE email = 'a@gmail.com';
```

Natija:

```text
Index Only Scan using idx_users_email_include
```

## Asosiy farq

```text
Index Scan:      Index -> Heap
Index Only Scan: Index only
```

### Nima uchun "Only"?

Chunki:

- heap read yo‘q
- table access yo‘q

Faqat index ishlatiladi.

## Qachon Index Only Scan ishlaydi?

2 ta shart kerak:

1. Kerakli column’larning hammasi index ichida bo‘lishi kerak
2. Visibility Map clean bo‘lishi kerak

### Visibility Map nima?

Postgres MVCC ishlatadi.

Har row:

- visiblemi?
- deletedmi?
- updatedmi?

tekshiriladi.

Ba’zan Postgres baribir heap’ga borib visibility check qiladi.

### VACUUM ahamiyati

VACUUM ishlasa, visibility map yangilanadi.

Shunda Index Only Scan ko‘proq effective ishlaydi.

## INCLUDE nima uchun muhim?

INCLUDE extra payload saqlaydi.

Misol:

```sql
CREATE INDEX idx_orders
ON orders(user_id)
INCLUDE(total_price, created_at);
```

Query:

```sql
SELECT total_price, created_at
FROM orders
WHERE user_id = 10;
```

Bu query Index Only Scan bo‘lishi mumkin.

## Performance farqi

Index Scan:

- heap access bor
- random I/O ko‘p
- sekinroq

Index Only Scan:

- heap access yo‘q
- kamroq disk read
- tezroq

## Real hayot analogiyasi

### Index Scan

Telefon kitobidan address topib, keyin uyga borish.

### Index Only Scan

Telefon kitobining o‘zida kerakli ma’lumot bor. Uyga borish kerak emas.

## Composite index misoli

```sql
CREATE INDEX idx_products
ON products(category_id)
INCLUDE(name, price);
```

Query:

```sql
SELECT name, price
FROM products
WHERE category_id = 1;
```

Bu Index Only Scan bo‘lishi mumkin.

## SELECT * muammosi

`SELECT *` ko‘pincha Index Only Scan’ni yo‘q qiladi.

Chunki barcha column’lar index ichida bo‘lmaydi.

## Best practice

Agar:

- WHERE bir column’da
- SELECT boshqa kichik column’larda

bo‘lsa, INCLUDE ishlatish yaxshi.

## Qisqa xulosa

Index Scan:

- index + heap
- actual row heap’dan olinadi

Index Only Scan:

- faqat index
- heap access yo‘q

## Muhim formula

```text
Index Scan:      Index -> Heap
Index Only Scan: Index only
```
