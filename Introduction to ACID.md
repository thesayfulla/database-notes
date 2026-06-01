Type: ACID
### ACID bu nima?

ACID — bu **tranzaksiyalar** (transaction) ishonchli ishlashi uchun kerak bo‘ladigan 4 ta xususiyatlar to‘plami:

- **A — Atomicity (Atomiklik)**
- **C — Consistency (Izchillik)**
- **I — Isolation (Izolyatsiya)**
- **D — Durability (Barqarorlik)**

PostgreSQL kabi RDBMS’larda ACID, odatda, bank o‘tkazmalari, buyurtma yaratish, balans yangilash kabi “yarim bajarilsa zarar bo‘ladigan” operatsiyalar uchun juda muhim.

---

## 1) Atomicity (Atomiklik)

Tranzaksiya ichidagi amallar **hammasi birga muvaffaqiyatli yakunlanadi yoki hammasi bekor qilinadi**.

- Agar tranzaksiya o‘rtasida xatolik chiqsa — **ROLLBACK**.
- Hech qanday “yarimta o‘zgarish” qolmasligi kerak.

Misol (pul o‘tkazish):

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT; -- xato bo‘lsa ROLLBACK
```

---

## 2) Consistency (Izchillik)

Tranzaksiya natijasida ma’lumotlar bazasi **qoidalar va cheklovlarni (constraints)** buzmasligi kerak.

Masalan:

- `PRIMARY KEY` takrorlanmasin
- `FOREIGN KEY` bog‘lanishlar buzilmasin
- `CHECK`, `NOT NULL`, `UNIQUE` shartlar saqlansin

Qisqacha: tranzaksiya **valid holatdan valid holatga** olib o‘tadi.

---

## 3) Isolation (Izolyatsiya)

Bir vaqtda ishlayotgan tranzaksiyalar bir-biriga xalaqit bermasligi kerak — natija **go‘yoki ketma-ket** bajarilgandek chiqadi.

### PostgreSQL’da isolation level’lar

- **Read Committed (default)**: har bir statement yangi snapshot oladi; boshqa tranzaksiyaning commit bo‘lgan o‘zgarishi keyingi statement’da ko‘rinishi mumkin.
- **Repeatable Read**: tranzaksiya bo‘yi bitta snapshot; bir xil SELECT’lar bir xil natija qaytaradi.
- **Serializable**: eng kuchli; PostgreSQL SSI (Serializable Snapshot Isolation) orqali “ketma-ket bajarilgandek” kafolat beradi.

### Klassik muammolar (anomalies)

- Dirty read (PostgreSQL’da amalda bo‘lmaydi)
- Non-repeatable read
- Phantom read

---

## 4) Durability (Barqarorlik)

Tranzaksiya **COMMIT** bo‘lgandan keyin natija saqlanib qolishi kerak — hatto server o‘chib qolsa ham.

PostgreSQL’da buni asosan:

- **WAL (Write-Ahead Log)** ta’minlaydi: avval log yoziladi, keyin data sahifalar yoziladi.

Amaliy sozlama:

- `synchronous_commit = on` (default): kuchliroq durability, lekin sekinroq bo‘lishi mumkin.
- `synchronous_commit = off`: tezroq, lekin crash bo‘lsa oxirgi commit’lar yo‘qolishi ehtimoli bor.

---

## Tez eslab qolish uchun

- **Atomicity**: “hammasi yoki hech narsa”
- **Consistency**: “qoidalar buzilmaydi”
- **Isolation**: “parallel bo‘lsa ham aralashmaydi”
- **Durability**: “commit yo‘qolmaydi”