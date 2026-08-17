# What is Database Cursors?

Database cursor — query natijasini **row-by-row** o‘qish uchun ishlatiladigan object.

Cursor bo‘lmasa database barcha rowlarni birdan yuboradi.

Cursor bilan esa rowlar asta-sekin olinadi.

## Simple idea

Without cursor:

```sql
SELECT * FROM users;
```

Barcha ma’lumot birdan keladi.

With cursor:

```text
Open cursor
Fetch first row
Fetch next row
Fetch next row
...
Close cursor
```

Rowlar bittadan process qilinadi.

## Why use cursor?

### Good for

- katta dataset
- memory tejash
- streaming data
- batch processing
- background jobs

### Bad for

- oddiy CRUD
- kichik querylar
- high-performance OLTP system

Ko‘pincha set-based SQL cursor’dan tezroq ishlaydi.

## Example (PostgreSQL)

```sql
BEGIN;

DECLARE user_cursor CURSOR FOR
SELECT id, name FROM users;

FETCH NEXT FROM user_cursor;
FETCH NEXT FROM user_cursor;

CLOSE user_cursor;

COMMIT;
```

## Cursor lifecycle

```text
DECLARE -> OPEN -> FETCH -> CLOSE
```

## Types of database cursors

| Type         | Description                        |
| ------------ | ---------------------------------- |
| Forward-only | faqat oldinga yuradi               |
| Scrollable   | oldinga va orqaga yuradi           |
| Read-only    | row update qilolmaydi              |
| Updatable    | current row update qilishi mumkin  |

## Server-side vs client-side cursor

| Feature      | Server-side Cursor          | Client-side Cursor        |
| ------------ | --------------------------- | ------------------------- |
| Storage      | database server             | application memory        |
| Fetching     | row-by-row                  | hammasi birdan            |
| Memory Usage | kam memory                  | ko‘p memory               |
| Performance  | katta dataset uchun yaxshi  | kichik query uchun yaxshi |
| Network      | ko‘p fetch request          | bitta katta response      |
| Use Case     | streaming large data        | normal query              |

### Server-side cursor

Cursor state database server ichida saqlanadi.

Application rowlarni asta-sekin oladi.

```text
App -> FETCH 100 rows
App -> FETCH next 100 rows
```

Advantages:

- memory kam ishlatiladi
- katta dataset bilan ishlay oladi
- ETL va batch processing uchun yaxshi

Disadvantages:

- DB resource uzoq band bo‘ladi
- long transaction paydo bo‘lishi mumkin
- network round-trip ko‘payadi

Real examples:

- millionlab row export qilish
- analytics pipeline
- data migration

### Client-side cursor

Application barcha rowlarni avval memory’ga yuklaydi.

```sql
SELECT * FROM users;
```

Keyin local iteratsiya qiladi.

Advantages:

- oddiy
- DB call kam
- kichik querylarda tez

Disadvantages:

- RAM ko‘p ishlatadi
- katta dataset uchun yomon

Real examples:

- admin panel
- dashboard
- oddiy web request

## Real production example

### Without cursor

```python
logs = db.query("SELECT * FROM logs")
```

Muammo:

- 10 million row RAM’ga yuklanadi
- application crash bo‘lishi mumkin

### With server-side cursor

```python
for row in cursor.fetchmany(1000):
    process(row)
```

Natija:

- rowlar stream bo‘lib keladi
- memory stabil qoladi

## Important note

Cursor ko‘pincha set-based SQL’dan sekinroq.

Bad:

```text
for each row:
    update row
```

Better:

```sql
UPDATE users
SET active = false
WHERE last_login < NOW() - INTERVAL '1 year';
```

Database engine set operation uchun optimize qilingan.

## Quick summary

```text
Cursor = row-by-row processing tool

Server-side cursor:
    DB state saqlaydi
    Huge dataset uchun yaxshi

Client-side cursor:
    App state saqlaydi
    Small query uchun yaxshi

Set-based SQL > Cursor
ko‘p production systemlarda
```
