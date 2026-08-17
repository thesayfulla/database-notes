# SQL Query Planner & Optimizer

**EXPLAIN** — bu keyword orqali biz o‘zimiz yozgan querylar qanday ishlayotganini ko‘ra olamiz.

**Misol:**

```sql
EXPLAIN SELECT * FROM table;
```

Natijada qancha worker ishlayotgani, index bormi yoki sequential ma’lumotlar fetch bo‘lib kelayotgani, shu bilan birga column size (width) qancha ekanligi kabi ma’lumotlar keladi.
