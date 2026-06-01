# PostgreSQL Scan Types

## 1. Seq Scan (Table Scan)

Postgres table’dagi barcha row’larni boshidan oxirigacha o‘qiydi.

### Qanday ishlaydi?

- Hamma page sequential o‘qiladi
- Har bir row WHERE condition bilan tekshiriladi

### Afzalligi

- Sequential read tez
- Katta amount data uchun yaxshi

### Qachon ishlatiladi?

- Table kichkina bo‘lsa
- Ko‘p row qaytsa
- Index foydasiz bo‘lsa

### Misol

```sql
SELECT * FROM users WHERE is_active = true;
```

Agar row’larning katta qismi active bo‘lsa:
→ Seq Scan

### Mental model

"Hamma narsani birma-bir tekshirish"

---

## 2. Index Scan

Postgres index orqali kerakli row location’ni topadi,
keyin heap(table)dan row’ni olib keladi.

### Qanday ishlaydi?

1. Index’dan TID topiladi
2. Heap’ga boriladi
3. Actual row olinadi

### Afzalligi

- Juda selective query uchun tez
- Kam row qaytsa yaxshi

### Kamchiligi

- Random I/O ko‘p qiladi
- Har row uchun boshqa page’ga sakrashi mumkin

### Qachon ishlatiladi?

- WHERE id = 10
- WHERE email = 'x'

### Misol

```sql
CREATE INDEX idx_users_email ON users(email);
```

```sql
SELECT * FROM users
WHERE email = 'a@gmail.com';
```

→ Index Scan

### Mental model

"Avval address topiladi,
keyin uyga boriladi"

---

## 3. Bitmap Index Scan

Index Scan bilan Seq Scan orasidagi hybrid usul.

### Qanday ishlaydi?

1. Index’dan matching row’lar topiladi
2. Bitmap yaratiladi
3. Heap page’lar tartibli o‘qiladi

### Afzalligi

- Random I/O kamayadi
- Page’lar grouped holda o‘qiladi

### Qachon ishlatiladi?

- O‘rtacha amount row qaytsa
- Juda kam ham emas
- Juda ko‘p ham emas

### Misol

```sql
SELECT *
FROM orders
WHERE status = 'pending';
```

→ Bitmap Index Scan
→ Bitmap Heap Scan

### Mental model

"Oldin kerakli page’larni yozib olish,
keyin tartib bilan o‘qish"

---

# Heap nima?

Postgres’da actual data heap’da saqlanadi.

Index:
value -> TID

xolos.

Keyin heap’dan actual row olinadi.

---

# Index Only Scan

Agar kerakli column’larning hammasi index ichida bo‘lsa,
heap’ga borilmaydi.

### Misol

```sql
SELECT email
FROM users
WHERE email = 'x';
```

→ Index Only Scan

### Afzalligi

- Juda tez
- Heap access yo‘q

---

# Qisqa taqqoslash

Seq Scan

- Sequential read
- Katta result uchun yaxshi

Index Scan

- Random read
- Juda kam row uchun yaxshi

Bitmap Scan

- Mixed approach
- O‘rtacha row uchun yaxshi

Index Only Scan

- Heap access yo‘q
- Eng tez variantlardan biri

---

# Planner nimalarga qaraydi?

- random_page_cost
- seq_page_cost
- statistics
- rows estimate
- table size
- selectivity
- correlation
- visibility map
