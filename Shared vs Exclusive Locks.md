# Shared vs Exclusive Locks

Type: Concurrency Control

## Shared Lock (S Lock)

- Read (`SELECT`) uchun ishlatiladi
- Bir nechta transaction shared lock olishi mumkin
- Write (`UPDATE/DELETE`) bloklanadi
- Concurrent reads mumkin

```sql
SELECT * FROM users
WHERE id=1
FOR SHARE;
```

---

## Exclusive Lock (X Lock)

- Data o‘zgartirish uchun ishlatiladi
- Faqat bitta transaction exclusive lock oladi
- Read va write operation’larni bloklaydi

```sql
SELECT * FROM users
WHERE id=1
FOR UPDATE;
```

yoki

```sql
UPDATE users
SET balance = balance - 100
WHERE id=1;
```

---

## Real Example

### Shared Lock

- 1000 user balansni ko‘ryapti
- Hamma faqat read qilyapti
- Shared lock yetarli

### Exclusive Lock

- Ikki user bir vaqtda pul yechmoqda
- Race condition oldini olish uchun row lock qilinadi

---

## Short Table

| Lock | Read | Write | Multiple Transactions |
| --- | --- | --- | --- |
| Shared | ✅ | ❌ | ✅ |
| Exclusive | ❌ | ❌ | ❌ |
