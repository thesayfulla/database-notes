# MongoDB Clustered Collections

**Clustered Collection** — bu hujjatlar (documents) diskda **ma’lum bir field bo‘yicha fizik tartibda saqlanadigan collection**.

Oddiy holatda MongoDB collection’larda:

- `_id` bo‘yicha **logical index** bor (primary key)
- lekin data diskda tartibsiz (heap-like storage)

Clustered collection’da esa:

- data **index tartibida diskka yoziladi**
- ya’ni **index = storage order**

---

### 🔹 Simple tushuncha

Oddiy collection:

```
Disk:[doc3] [doc1] [doc5] [doc2]
Index:_id → pointer to doc
```

Clustered collection:

```
Disk:[doc1] [doc2] [doc3] [doc5]
(Index order = physical order)
```

---

### 🔹 Qanday ishlaydi?

Clustered collection yaratishda:

- bitta field tanlanadi (`_id` yoki boshqa unique field)
- shu field bo‘yicha:
    - sorting
    - storage

Example:

```js
db.createCollection("orders", {  clusteredIndex: {    key: { createdAt: 1 },    unique: true  }})
```

Bu yerda:

- `createdAt` → **physical order**
- har bir yangi document → **to‘g‘ri joyga insert qilinadi**

---

### 🔹 Qachon ishlatish kerak?

**1. Time-series data**

- logs
- events
- audit trails

Sababi:

- data tabiatan sorted (`timestamp` bo‘yicha)

**2. Range queries tezlashadi**

```js
db.orders.find({  createdAt: { $gte: ..., $lte: ... }})
```

👉 Disk scan → sequential (SSD friendly)

---

### 🔹 Advantages

✔ Kamroq disk seek
✔ Range query juda tez
✔ Index + data birga → memory efficient
✔ Better compression (yaqin qiymatlar yonma-yon)

---

### 🔹 Disadvantages

❌ Insert qimmat (o‘rtaga insert bo‘lsa)
❌ Fieldni o‘zgartirib bo‘lmaydi (cluster key immutable)
❌ Faqat **bitta clustered index** bo‘ladi
❌ Noto‘g‘ri field tanlansa → performance yomonlashadi

---

### 🔹 Clustered vs Normal Collection

|Feature|Normal|Clustered|
|---|---|---|
|Storage order|Random|Sorted|
|Index|Separate|Built-in|
|Insert speed|Fast|Sometimes slower|
|Range query|Medium|Very fast|

---

### 🔹 Real-life analogy

- Normal collection → kitoblar stol ustida tartibsiz
- Clustered → kitoblar **alfavit bo‘yicha tokchada**

---

### 🔹 Important note

- MongoDB’da bu feature nisbatan **yangi (5.3+)**
- Default `_id` clustered bo‘lishi mumkin (internal optimization)
