# Two-Phase Locking (2PL) in PostgreSQL

`Two-Phase Locking (2PL)` — bu transaction davomida lock’larni boshqarish qoidasi.

Maqsad:
- data consistency saqlash
- race condition oldini olish
- concurrent transaction’larni xavfsiz ishlatish

---

# 2PL qanday ishlaydi?

2 ta phase bor:

## 1. Growing Phase

Transaction:

- yangi lock olishi mumkin
- lekin release qila olmaydi

Misol:

```sql
BEGIN;

SELECT * FROM users WHERE id=1 FOR UPDATE;
UPDATE users SET balance=500 WHERE id=1;
```

Bu yerda lock olinmoqda.

---

## 2. Shrinking Phase

Transaction:

- lock release qiladi
- yangi lock ola olmaydi

Bu phase odatda:

```sql
COMMIT;

-- yokiROLLBACK;
```

dan keyin boshlanadi.

---

# PostgreSQL’da 2PL bormi?

Ha, lekin:

PostgreSQL toza “strict 2PL database” emas.

U:

- `MVCC (Multi-Version Concurrency Control)`
- - locking

kombinatsiyasidan foydalanadi.

Ammo:

- `FOR UPDATE`
- `FOR SHARE`
- table locks
- explicit locks

2PL behavior beradi.

---

# Real Example

## Bank transfer

### Transaction A

```sql
BEGIN;
SELECT * FROM accounts WHERE id=1 FOR UPDATE;
UPDATE accountsSET balance = balance - 100WHERE id=1;

-- hali commit yo'q
```

Bu row lock bo‘ladi.

---

### Transaction B

```sql
UPDATE accounts SET balance = balance + 50 WHERE id=1;
```

Transaction B kutadi.

Sabab:

- Transaction A lock ushlab turibdi
- commit bo‘lmaguncha release bo‘lmaydi

Bu strict 2PL behavior.

---

# Strict 2PL nima?

Oddiy 2PL:

- lock transaction o‘rtasida ham release bo‘lishi mumkin

Strict 2PL:

- barcha write lock’lar COMMIT gacha ushlanadi

PostgreSQL write lock’larda asosan strict behavior qiladi.

---

# PostgreSQL’da Lock Types

## Row-Level Locks

```sql
FOR UPDATE FOR SHAREFOR NO KEY UPDATE FOR KEY SHARE
```

---

## Table Locks

```sql
LOCK TABLE users IN ACCESS EXCLUSIVE MODE;
```

---

# Muammo: Deadlock

2PL’ning eng katta muammosi.

## Misol

### Transaction A

```
UPDATE users SET ... WHERE id=1;
```

### Transaction B

```
UPDATE users SET ... WHERE id=2;
```

Keyin:

### A

```
UPDATE users SET ... WHERE id=2;
```

### B

```
UPDATE users SET ... WHERE id=1;
```

Natija:

- A B’ni kutadi
- B A’ni kutadi

=> deadlock

PostgreSQL bittasini o‘ldiradi:

```
ERROR: deadlock detected
```

---

# Deadlock oldini olish

## 1. Bir xil order ishlatish

Har doim:

```
id ASC
```

bo‘yicha update qilish.

---

## 2. Transaction’ni qisqa saqlash

Yomon:

```sql
BEGIN; UPDATE ...sleep(20) COMMIT;
```

---

## 3. Keraksiz lock olmaslik

`FOR UPDATE` ni faqat kerak bo‘lsa ishlatish.

---

# MVCC vs 2PL

| MVCC                   | 2PL                     |
| ---------------------- | ----------------------- |
| Readers block qilmaydi | Readers kutishi mumkin  |
| Snapshot ishlatadi     | Lock ishlatadi          |
| PostgreSQL default     | Traditional DB approach |
| High concurrency       | Simpler consistency     |

---

# PostgreSQL’da qachon 2PL’ni his qilasiz?

Ko‘pincha:

- `SELECT ... FOR UPDATE`
- inventory systems
- bank systems
- queue workers
- booking systems

---

# Real Production Examples

## 1. Seat Booking

```sql
SELECT * FROM seats WHERE id=10 FOR UPDATE;
```

Bir vaqtning o‘zida 2 ta user bir seat ololmaydi.

---

## 2. Queue Worker

```sql
SELECT * FROM jobs
WHERE status='pending'
FOR UPDATE SKIP LOCKED
LIMIT 1;
```

Worker’lar bir job’ni 2 marta ishlamaydi.

---

## 3. Money Transfer

Balance consistency uchun row lock kerak.

---

# Muhim Idea

PostgreSQL:

- pure lock-based DB emas
- MVCC asosiy model
- lekin locking hali ham juda muhim
- ayniqsa write conflict’larda

`FOR UPDATE` ishlatganingizda, siz amalda 2PL behavior’ga yaqinlashasiz.
