# Row-based & Column-based Databases

### Row based DB
- Ma’lumotlar row (satr) bo‘yicha saqlanadi.
- Bitta user haqidagi barcha columnlar bir joyda turadi.
- INSERT, UPDATE, DELETE juda tez ishlaydi.
- CRUD workload uchun juda qulay.
- OLTP systems uchun ideal.
- Misollar: PostgreSQL, MySQL, Oracle.

### Column based DB
- Ma’lumotlar column bo‘yicha saqlanadi.
- Har bir column alohida saqlanadi.
- Analytics querylar juda tez ishlaydi.
- Aggregation va reporting uchun kuchli.
- OLAP systems uchun ideal.
- Writing juda sekin
- Misollar: ClickHouse, BigQuery, Snowflake.

### OLTP

- Online Transaction Processing.
- Ko‘p write va real-time transactionlar bo‘ladi.
- Small querylar juda ko‘p ishlaydi.
- Banking, ecommerce, messaging kabi systemlarda ishlatiladi.
- Odatda Row based DB bilan ishlaydi.

### OLAP

- Online Analytical Processing.
- Katta analytics va report querylar ishlaydi.
- Millionlab row ustida aggregation qilinadi.
- BI, dashboard, data warehouse uchun ishlatiladi.
- Odatda Column based DB bilan ishlaydi.
