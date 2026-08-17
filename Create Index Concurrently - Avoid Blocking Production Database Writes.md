# Create Index Concurrently

Production database’da oddiy `CREATE INDEX` juda xavfli bo‘lishi mumkin.

Sababi: u table’ni write uchun lock qiladi.

Natijada:

- INSERT to‘xtaydi
- UPDATE to‘xtaydi
- DELETE to‘xtaydi

Production traffic block bo‘lishi mumkin.

Shuning uchun PostgreSQL’da `CREATE INDEX CONCURRENTLY` mavjud.

## Oddiy CREATE INDEX muammosi

Misol:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Postgres ACCESS EXCLUSIVE emas, lekin write operation’larni block qiladigan lock oladi.

Natija:

- INSERT kutadi
- UPDATE kutadi
- DELETE kutadi

Katta table’da bu bir necha minut davom etishi mumkin.

## Production’da nima bo‘ladi?

Tasavvur qil: 50 million row table.

`CREATE INDEX` 10 minut ishladi.

Bu vaqt ichida:

- API sekinlashadi
- requests queue bo‘ladi
- timeout chiqishi mumkin

## CREATE INDEX CONCURRENTLY nima qiladi?

Misol:

```sql
CREATE INDEX CONCURRENTLY idx_users_email
ON users(email);
```

Bu table write operation’larini block qilmaydi.

Production system ishlashda davom etadi.

## Muhim farq

Oddiy CREATE INDEX:

- writes block bo‘ladi

CREATE INDEX CONCURRENTLY:

- writes davom etadi

## Qanday ishlaydi?

CONCURRENTLY index’ni bir nechta phase’da yaratadi.

Taxminan:

1. Table scan qiladi
2. Index build qiladi
3. O‘zgargan row’larni qayta sync qiladi
4. Index valid bo‘ladi

## Nega sekinroq?

Chunki:

- bir necha marta scan qiladi
- extra bookkeeping qiladi
- concurrent changes’larni kuzatadi

Shuning uchun CONCURRENTLY oddiy CREATE INDEX’dan sekinroq.

## Lekin asosiy foyda

Production traffic to‘xtamaydi.

Bu eng muhim advantage.

## Muhim cheklov

CONCURRENTLY transaction ichida ishlamaydi.

### Xato misol

```sql
BEGIN;

CREATE INDEX CONCURRENTLY idx_users_email
ON users(email);

COMMIT;
```

Error:

```text
ERROR: CREATE INDEX CONCURRENTLY cannot run inside a transaction block
```

### To‘g‘ri ishlatish

```sql
CREATE INDEX CONCURRENTLY idx_users_email
ON users(email);
```

Bitta statement sifatida ishlaydi.

## Lock hali ham bormi?

Ha.

Lekin juda kichik va qisqa lock’lar. Writes to‘liq block bo‘lmaydi.

## Real hayotiy analogiya

Oddiy CREATE INDEX:

Yo‘lni butunlay yopib, yangi asfalt qilish.

CREATE INDEX CONCURRENTLY:

Mashinalar yurayotgan paytda, yon tomondan ehtiyotkorlik bilan ishlash.

## CREATE UNIQUE INDEX CONCURRENTLY

Unique index uchun ham ishlaydi.

Misol:

```sql
CREATE UNIQUE INDEX CONCURRENTLY idx_users_email
ON users(email);
```

### Muhim muammo

Agar concurrent insert vaqtida duplicate data kirsa, unique index creation fail bo‘lishi mumkin.

## Failure holati

Agar build vaqtida error chiqsa, INVALID index qolib ketishi mumkin.

Ko‘rish:

```sql
SELECT *
FROM pg_indexes
WHERE tablename = 'users';
```

yoki:

```text
\d users
```

### INVALID index muammosi

Agar failed bo‘lsa, index mavjud, lekin usable emas.

### Tozalash

```sql
DROP INDEX CONCURRENTLY idx_users_email;
```

### Nega DROP ham CONCURRENTLY?

Oddiy DROP INDEX query’larni block qilishi mumkin.

CONCURRENTLY esa minimal blocking bilan ishlaydi.

## Resource cost

CONCURRENTLY:

- ko‘proq CPU
- ko‘proq disk I/O
- ko‘proq vaqt

ishlatadi.

## Qachon ishlatish kerak?

Production environment’da deyarli har doim.

### Qachon oddiy CREATE INDEX ishlatish mumkin?

- local development
- maintenance window
- kichik table
- traffic yo‘q payt

## Large table example

Table: 500 million rows.

Oddiy CREATE INDEX:

- write traffic’ni muzlatib qo‘yishi mumkin

CREATE INDEX CONCURRENTLY:

- sekinroq
- lekin service ishlashda davom etadi

## PostgreSQL ichki mexanizmi

CONCURRENTLY 2 ta table scan qiladi.

Sababi: build vaqtida o‘zgargan row’larni ham ushlashi kerak.

## Performance tradeoff

Oddiy CREATE INDEX:

- tezroq
- lekin blocking

CONCURRENTLY:

- sekinroq
- lekin non-blocking

## EXPLAIN bilan bog‘liq emas

CREATE INDEX CONCURRENTLY query plan emas.

Bu DDL operation.

## Best practice

Production’da CREATE INDEX CONCURRENTLY default tanlov bo‘lishi kerak.

## Qisqa xulosa

CREATE INDEX:

- tezroq
- write block qiladi

CREATE INDEX CONCURRENTLY:

- sekinroq
- write block qilmaydi
- production-safe

## Muhim eslab qol

Production database’da oddiy CREATE INDEX katta outage sababchisi bo‘lishi mumkin.
