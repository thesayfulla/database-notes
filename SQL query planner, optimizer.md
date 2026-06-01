# SQL Query Planner & Optimizer

**EXPLAIN -** bu keyword orqali biz doim o’zimiz yozgan querylarni qanday ishlayotganini ko’ra olamiz

**Misol:**

```sql
EXPLAIN SELECT * FROM table;
```

Natijada qancha workerlar ishlayotgani, index bormi yoki sequantial ma’lumotlar fetch bo’lib kelayotgani shu bilan birga column size(width) qancha ekanligi kabi ma’lumotlar keladi.
