# Offset Pagination is Slow

`OFFSET` sekin bo‘lishining asosiy sababi:

```text
database rowlarni skip qila olmaydi,
ularni avval o‘qib chiqishga majbur
```

## Misol

```sql
SELECT * FROM posts
ORDER BY id
LIMIT 10 OFFSET 1000000;
```

Siz o‘ylaysiz:

```text
database 1,000,001-rowdan boshlaydi
```

Lekin aslida:

```text
1 -> 2 -> 3 -> ... -> 1,000,000
```

hammasini yurib chiqadi.

Keyin:

- birinchi millionini tashlaydi
- keyingi 10 tasini qaytaradi

## Nega index yordam bermaydi?

Chunki `OFFSET` — "qancha row tashlash kerak?" degan concept.

B+Tree index esa "qayerdan boshlash kerak?" degan savolni yaxshi bajaradi.

Lekin "1000000 ta row skip qil" degan operation index uchun ham qimmat.

## Internally nima bo‘ladi?

Tasavvur:

```sql
ORDER BY id
LIMIT 10 OFFSET 1000000
```

PostgreSQL roughly:

```text
1. index scan boshlaydi
2. 1 million tuple o‘qiydi
3. discard qiladi
4. keyingi 10 tasini beradi
```

## EXPLAIN ANALYZE’da ko‘rinishi

```sql
EXPLAIN ANALYZE
SELECT * FROM posts
ORDER BY id
LIMIT 10 OFFSET 1000000;
```

Ko‘pincha:

```text
Rows Removed by Limit: 1000000
```

ko‘rasiz.

Bu juda katta red flag.

## Performance cost

### OFFSET kichik

```sql
OFFSET 10
```

cheap.

### OFFSET katta

```sql
OFFSET 5000000
```

- CPU: ko‘p ishlaydi
- Disk: ko‘p o‘qiydi
- Memory: ko‘proq ishlatiladi
- Latency: oshadi

## Real production problem

Infinite scroll:

```text
page=10000
```

API:

```sql
OFFSET 99990
```

Har request:

- katta scan qiladi
- database pressure oshadi

Natija:

- slow API
- high CPU
- cache miss
- replication lag

## Cursor pagination nega tez?

```sql
SELECT * FROM posts
WHERE id > 1000000
ORDER BY id
LIMIT 10;
```

B+Tree index to‘g‘ridan-to‘g‘ri kerakli joyga sakraydi.

Million row discard qilmaydi.

## Visual

### OFFSET

```text
start ↓
1 2 3 4 5 6 7 8 ... 1000000
                    ↑ finally
```

### Cursor

```text
jump directly here ↓
                   1000000 1000001 1000002
```

## PostgreSQL’da yanada yomon bo‘ladigan holatlar

### Large rows

Agar row katta bo‘lsa:

- JSONB
- TEXT
- TOAST data

skip qilish ham qimmatlashadi.

### JOIN bilan

```sql
SELECT * FROM posts
JOIN users ...
LIMIT 10 OFFSET 100000;
```

join result ham materialize bo‘lishi mumkin.

Juda expensive.

## Rule

### OFFSET ishlatish mumkin

- admin panel
- page 1-10
- small dataset

### OFFSET avoid qilish kerak

- infinite scroll
- social feed
- chat app
- million+ rows
- realtime systems

## Best practice

Cursor pagination:

```sql
WHERE created_at < last_seen_created_at
```

yoki:

```sql
WHERE id > last_id
```

Bu production standard hisoblanadi.
