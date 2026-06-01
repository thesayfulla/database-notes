## Nima o'zi?

**Memcached** — bu juda sodda va tez ishlaydigan **distributed in-memory key-value cache**.

Asosiy maqsadi:

- Database yukini kamaytirish
- Tez-tez o'qiladigan ma'lumotlarni RAM'da saqlash
- Query natijalarini cache qilish

---

## Arxitektura

```
                +------------+
                |   Client   |
                +------------+
                       |
                       v
              +----------------+
              |   Memcached    |
              |   (RAM only)   |
              +----------------+
                       |
                 Cache Miss
                       |
                       v
              +----------------+
              |   PostgreSQL   |
              +----------------+
```

Flow:

```
1. Client -> GET user:123
2. Memcached:
      Found? -> Return data
      Not Found? -> DB query
3. PostgreSQL returns data
4. Store into Memcached
5. Return to Client
```

---

## Data Structure

Memcached faqat:

```
Key -> Value
```

Misol:

```
"user:1"        -> JSON
"product:100"   -> JSON
"article:55"    -> HTML
```

Database kabi:

- JOIN yo'q
- Transaction yo'q
- Secondary index yo'q
- Query language yo'q

---

## Internal Architecture

```
+--------------------+
| Network Thread     |
+--------------------+
          |
          v
+--------------------+
| Worker Threads     |
+--------------------+
          |
          v
+--------------------+
| Hash Table         |
+--------------------+
          |
          v
+--------------------+
| Memory Slabs       |
+--------------------+
```

---

## 1. Hash Table

Key hash qilinadi:

```
"user:123"
      |
      v
hash()
      |
      v
Bucket #42
```

Shuning uchun:

```
GET O(1)
SET O(1)
DELETE O(1)
```

o'rtacha holatda.

---

## 2. Slab Allocator

Memcached har bir object uchun `malloc()` qilmaydi.

Buning o'rniga RAM'ni bloklarga bo'ladi.

```
RAM
 |
 +-- Slab Class 64B
 |
 +-- Slab Class 128B
 |
 +-- Slab Class 256B
 |
 +-- Slab Class 512B
 |
 +-- Slab Class 1KB
```

Misol:

```
Value size = 90B
```

↓

```
128B slabga joylanadi
```

Bu:

- fragmentationni kamaytiradi
- allocationni tezlashtiradi

---

## 3. LRU Eviction

RAM to'lib qolsa:

```
Least Recently Used
```

element o'chiriladi.

```
Old Item      ↓Evicted
```

Misol:

```
RAM Full

A (last access 1h ago)
B (last access 10m ago)
C (last access now)

A deleted
```

---

## Distributed Architecture

Bir nechta Memcached node:

```
           Client
              |
     +--------+--------+
     |                 |
     v                 v
 Node1            Node2```

```

Odatda:

```
Consistent Hashing
```

ishlatiladi.

```
user:1 -> Node1
user:2 -> Node2
user:3 -> Node1
```

---

## Cache Aside Pattern

Eng mashhur pattern:

```
GET user
   |
   v
Cache?
   |
 +----+
 |Yes |
 +----+
   |
 Return

 No
   |
   v
Database
   |
   v
Put Cache
   |
Return
```

Django'da ham ko'pincha shu ishlatiladi.

---

## Memcached vs Redis

|Feature|Memcached|Redis|
|---|---|---|
|RAM only|✅|✅|
|Persistence|❌|✅|
|Transactions|❌|✅|
|Replication|❌|✅|
|Data structures|Key-Value|Ko'p|
|Pub/Sub|❌|✅|
|Simplicity|Juda sodda|Murakkabroq|
|Speed|Juda tez|Juda tez|

---

## Qachon ishlatiladi?

Yaxshi mos keladi:

- User profile cache
- Session cache
- API response cache
- HTML page cache
- Expensive query cache

Mos emas:

- Permanent storage
- Banking transactionlar
- Queue systems
- Analytics data

---

## 1 jumlalik xulosa

**Memcached — bu RAM ichida hash table va slab allocator yordamida ishlaydigan, persistence va transaction'larsiz, juda tez key-value cache serveridir.**
