# Indexing

**Index** — [Database](https://wiki.42.uz/Database.md) tezligining sirli kaliti. Kitobning oxiridagi ko‘rsatkich kabi.

```text
Indekssiz: SELECT * FROM users WHERE ism = 'Ali'
  -> Hamma satrni o'qiydi (Sequential scan)

Indeks bilan:
  -> O'sha satrni topadi (Index lookup)
```

Million qatorda farq: **5 sekund vs 5 millisekund**.

### Index turlari

| Tur           | Tavsif                                                |
| ------------- | ----------------------------------------------------- |
| **B-Tree**    | Eng keng tarqalgan (default)                          |
| **Hash**      | Faqat `=` uchun                                       |
| **GIN**       | Massiv, [JSON](https://wiki.42.uz/JSON.md) (Postgres) |
| **GIST**      | Geo, range                                            |
| **BRIN**      | Katta jadvallar uchun                                 |
| **Full-text** | Matn qidirish                                         |
