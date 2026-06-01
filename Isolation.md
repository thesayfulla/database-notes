Type: ACID
Isolation — bir nechta transaction bir vaqtda ishlaganda, ular bir-biriga xalaqit bermasligi kerak.

Maqsad:
- race condition oldini olish
- inconsistent data chiqmasligi
- parallel transactionlarni xavfsiz boshqarish

Reading turlari:

- Dirty reads — transaction hali commit bo‘lmasidan turib o‘zgarishlarni ko‘rish, ya’ni hech qanday snapshot yaratilmaydi.

![Screenshot 2026-05-17 at 12.07.25.png](Screenshot_2026-05-17_at_12.07.25.png)

Dirty reads muammosini yechish uchun ushbu komandadan foydalanamiz, lekin PostgreSQL’da bu default bo‘ladi:

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

- Non repeatable reads — bu yerda COMMIT ishlagan bo‘ladi, lekin oxirgi queryda baribir xatoliklar yuzaga kelaveradi.

![Screenshot 2026-05-17 at 12.13.58.png](Screenshot_2026-05-17_at_12.13.58.png)

- Phantom read — bir transaction biror shart (masalan, `WHERE`) bilan qatorlarni o‘qiydi, ikkinchi transaction esa shu oraliqda shu shartga mos keladigan yangi qator(lar)ni INSERT qiladi yoki mavjudlarini shunday UPDATE qiladi. Birinchi transaction o‘sha queryni qayta bajarganda natijalar soni o‘zgarib ketadi ("phantom" qatorlar paydo bo‘ladi).
Odatda buni `REPEATABLE READ` yoki `SERIALIZABLE` izolatsiya darajalari oldini oladi. PostgreSQL’da `REPEATABLE READ` snapshot asosida ishlaydi va phantom read’larni amalda bloklaydi; eng qat’iy daraja `SERIALIZABLE` bo‘lib, kerak bo‘lsa conflict bo‘lganda transaction’ni qayta urinishga majbur qiladi.

![image.png](Database%20notes/Isolation/image.png)

### Isolation — transactionlar ishlayotgan paytda isolation level’lar

- Read Uncommitted — Isolation deyarli yo‘q. Tashqaridan bo‘layotgan har qanday o‘zgarish transactionga ko‘rinadi, commit qilingan bo‘lsa ham, qilinmagan bo‘lsa ham.
- Read Committed — Transaction ichidagi har bir query boshqa transactionlarning faqat commit qilingan o‘zgarishlarini ko‘radi.
- Repeatable Read — Transaction query orqali biror row’ni o‘qisa, transaction tugaguncha o‘sha row o‘zgarmagan holatda ko‘rinadi.
- Snapshot — Transaction ichidagi query transaction boshlangan vaqtgacha commit qilingan o‘zgarishlarnigina ko‘radi. Bu xuddi o‘sha paytdagi database snapshotiga o‘xshaydi.
- Serializable — Transactionlar xuddi bittadan ketma-ket ishlayotgandek boshqariladi.
- Har bir DBMS isolation level’larni turlicha implement qiladi.