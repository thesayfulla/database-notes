## B-Tree

- Data internal node va leaf node’larda saqlanishi mumkin
- Search ba’zan leaf’gacha bormaydi
- Har node:
    - key
    - child pointer
    - data(pointer) saqlaydi
- Range query uchun uncha optimal emas
- Strukturasi murakkabroq

### Misol

```
        [20]
       /    \
   [10]    [30,40]
```

Data yuqoridagi node’larda ham bo‘lishi mumkin.

---

## B+ Tree

- Internal node’lar faqat key saqlaydi
- Actual data faqat leaf node’da bo‘ladi
- Barcha search leaf node’gacha tushadi
- Leaf node’lar linked list kabi bog‘langan
- Range query juda tez ishlaydi

### Misol

```
        [20]
       /    \
    [10]   [30]

Leaf:
[1,5,10] -> [20,25] -> [30,40]
```

---

# Asosiy farqlar

| Feature | B-Tree | B+ Tree |
| --- | --- | --- |
| Data qayerda saqlanadi | Hamma node’larda | Faqat leaf’da |
| Search | Erta tugashi mumkin | Har doim leaf’gacha |
| Range query | Sekinroq | **Juda tez(chunki linkedin bo’lib aynan kerakli ma’lumotlar yana bog’langan)** |
| Leaf node link | Yo‘q | Bor |
| Disk performance | Yaxshi | Juda yaxshi |
| DB index uchun | Kam ishlatiladi | Eng ko‘p ishlatiladi |

---

# Qachon ishlatiladi?

## B-Tree

- Point lookup muhim bo‘lsa
- Kichik sistemalarda

## B+ Tree

- Database index
- File system
- Range query
- Sorting
- Large scale storage

---

# Real Hayot

## B+ Tree ishlatadigan DB’lar

- PostgreSQL
- MySQL
- Oracle Database

Masalan:

```sql
SELECT * FROM users
WHERE age BETWEEN 20 AND 30;
```

B+ Tree leaf node’lari linked bo‘lgani uchun juda tez ishlaydi.

---

# Qisqa Xulosa

```
B-Tree  = data everywhere
B+ Tree = data only in leaves
```

```
B-Tree  → point lookup
B+ Tree → databases & range queries
```
