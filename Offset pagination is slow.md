# Offset Pagination is Slow

`OFFSET` sekin bo‘lishining asosiy sababi:

```
database rowlarni skip qila olmaydi,
ularni avval o‘qib chiqishga majbur
```

---

# Misol

```sql
SELECT * FROM postsORDER BY id LIMIT 10 OFFSET 1000000;
```

Siz o‘ylaysiz:

```
database 1,000,001-rowdan boshlaydi
```

Lekin aslida:

```
1 -> 2 -> 3 -> ... -> 1,000,000
```

hammasini yurib chiqadi.

Keyin:

- birinchi millionini tashlaydi
- keyingi 10 tasini qaytaradi

---

# Nega Index yordam bermaydi?

Chunki `OFFSET`:

```
"qancha row tashlash kerak?"
```

degan concept.

B+Tree index:

```
"qayerdan boshlash kerak?"
```

ni yaxshi bajaradi.

Lekin:

```
1000000 ta row skip qil
```

degan operation index uchun ham qimmat.

---

# Internally nima bo‘ladi?

Tasavvur:

```
ORDER BY idLIMIT 10 OFFSET 1000000
```

PostgreSQL roughly:

```
1. index scan boshlaydi2. 1 million tuple o‘qiydi3. discard qiladi4. keyingi 10 tasini beradi
```

---

# EXPLAIN ANALYZE’da ko‘rinishi

```sql
EXPLAIN ANALYZE SELECT * FROM posts ORDER BY id LIMIT 10 OFFSET 1000000;
```

Ko‘pincha:

```
Rows Removed by Limit: 1000000
```

ko‘rasiz.

Bu juda katta red flag.

---

# Performance Cost

## OFFSET kichik

```
OFFSET 10
```

cheap.

---

## OFFSET katta

```
OFFSET 5000000
```

CPU:

- ko‘p ishlaydi

Disk:

- ko‘p o‘qiydi

Memory:

- ko‘proq ishlatiladi

Latency:

- oshadi

---

# Real Production Problem

Infinite scroll:

```
page=10000
```

API:

```
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

---

# Cursor Pagination nega tez?

```sql
WHERE id > 1000000 ORDER BY idLIMIT 10
```

B+Tree index:

```
to‘g‘ridan-to‘g‘ri kerakli joyga sakraydi
```

Million row discard qilmaydi.

---

# Visual

## OFFSET

```
start ↓
1 2 3 4 5 6 7 8 ... 1000000
					↑  finally
```

---

## Cursor

```
jump directly here↓
1000000 1000001 1000002
```

---

# PostgreSQL’da yanada yomon bo‘ladigan holatlar

## Large rows

Agar row katta bo‘lsa:

- JSONB
- TEXT
- TOAST data

skip qilish ham qimmatlashadi.

---

## JOIN bilan

```sql
SELECT * FROM posts JOIN users ... LIMIT 10 OFFSET 100000;
```

join result ham materialize bo‘lishi mumkin.

Juda expensive.

---

# Rule

## OFFSET ishlatish mumkin

- admin panel
- page 1-10
- small dataset

---

## OFFSET avoid qilish kerak

- infinite scroll
- social feed
- chat app
- million+ rows
- realtime systems

---

# Best Practice

Cursor pagination:

```sql
WHERE created_at < last_seen_created_at
```

yoki:

```sql
WHERE id > last_id
```

Bu production standard hisoblanadi.
