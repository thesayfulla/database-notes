# Memcached In-Memory Cache Architecture

## Nima o'zi?

**Memcached** — bu juda sodda va tez ishlaydigan **distributed in-memory key-value cache**.

Asosiy maqsadi:

- Database yukini kamaytirish
- Tez-tez o'qiladigan ma'lumotlarni RAM'da saqlash
- Query natijalarini cache qilish

## Arxitektura

```text
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

```text
1. Client -> GET user:123
2. Memcached:
      Found? -> Return data
      Not Found? -> DB query
3. PostgreSQL returns data
4. Store into Memcached
5. Return to Client
```

## Data structure

Memcached faqat:

```text
Key -> Value
```

Misol:

```text
"user:1"        -> JSON
"product:100"   -> JSON
"article:55"    -> HTML
```

Database kabi:

- JOIN yo'q
- Transaction yo'q
- Secondary index yo'q
- Query language yo'q

## Internal architecture

```text
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

### 1. Hash Table

Key hash qilinadi:

```text
"user:123"
      |
      v
hash()
      |
      v
Bucket #42
```

Shuning uchun o'rtacha holatda:

```text
GET    O(1)
SET    O(1)
DELETE O(1)
```

### 2. Slab Allocator

Memcached har bir object uchun `malloc()` qilmaydi.

Buning o'rniga RAM'ni bloklarga bo'ladi.

```text
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

```text
Value size = 90B
        ↓
128B slabga joylanadi
```

Bu:

- fragmentationni kamaytiradi
- allocationni tezlashtiradi

### 3. LRU Eviction

RAM to'lib qolsa, **Least Recently Used** element o'chiriladi.

Misol:

```text
RAM Full

A (last access 1h ago)   <- evicted
B (last access 10m ago)
C (last access now)
```

## Distributed architecture

Bir nechta Memcached node:

```text
           Client
              |
     +--------+--------+
     |                 |
     v                 v
   Node1             Node2
```

Odatda **consistent hashing** ishlatiladi:

```text
user:1 -> Node1
user:2 -> Node2
user:3 -> Node1
```

## Cache aside pattern

Eng mashhur pattern:

```text
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

## Memcached vs Redis

| Feature         | Memcached   | Redis       |
| --------------- | ----------- | ----------- |
| RAM only        | ✅          | ✅          |
| Persistence     | ❌          | ✅          |
| Transactions    | ❌          | ✅          |
| Replication     | ❌          | ✅          |
| Data structures | Key-Value   | Ko'p        |
| Pub/Sub         | ❌          | ✅          |
| Simplicity      | Juda sodda  | Murakkabroq |
| Speed           | Juda tez    | Juda tez    |

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

## 1 jumlalik xulosa

**Memcached — bu RAM ichida hash table va slab allocator yordamida ishlaydigan, persistence va transaction'larsiz, juda tez key-value cache serveridir.**
