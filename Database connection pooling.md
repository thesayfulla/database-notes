Har requestda yangi DB connection ochiladi va ish tugagach yopiladi.

```
Request -> Open Connection -> Query -> Close Connection
```

### Muammo

Connection ochish juda qimmat operation:

- TCP handshake
- Authentication
- SSL/TLS setup
- Backend process/thread creation
- Memory allocation

Shuning uchun har requestda open/close qilish latency va CPU xarajatini oshiradi.

### Real example

```python
# BAD
def get_user(id):    
	conn = psycopg.connect(...)    
	cur = conn.cursor()    
	cur.execute("SELECT * FROM users WHERE id=%s", (id,))    
	conn.close()
```

1000 request/sec bo‘lsa:

- 1000 ta connection create
- 1000 ta destroy

Bu database’ni o‘ldirib qo‘yishi mumkin.

---

# Connection Pooling

Pooling’da connection oldindan ochilib turadi va qayta ishlatiladi.

```
Request 
-> Take existing connection from pool 
-> Query         
-> Return connection back to pool
```

Connection yopilmaydi, reuse qilinadi.

---

## Architecture

```
App 
├── Conn 1 
├── Conn 2 
├── Conn 3 
└── Conn 4Requests kelganda:available connection olinadi
```

---

# Nega pooling juda muhim?

### 1. Performance

Connection reuse qilinadi.

Latency kamayadi.

### 2. Database protection

PostgreSQL’da har connection:

- process yaratadi
- memory yeydi

10000 ta connection → RAM explosion.

Pool limit qo‘yadi.

Masalan:

- appda 5000 user
- DB’da faqat 50 connection

---

### 3. Better concurrency

Requestlar navbat bilan connection ishlatadi.

---

# Real production example

### Without pool

```
Nginx -> Django -> PostgreSQL
```

1000 concurrent request → 1000 DB connection.

Postgres sekinlashadi.

---

### With pool

```
Nginx -> Django -> PgBouncer -> PostgreSQL
```

1000 request:

- PgBouncer faqat 50 connection ushlab turadi.

Qolgan requestlar reuse qiladi.

---

# Pooling turlari

## 1. Application-level pool

Library ichida.

Misollar:

- SQLAlchemy pool
- Django persistent connections
- psycopg pool

---

## 2. External pooler

Alohida service.

Eng mashhuri:

- PgBouncer

Bu production’da juda ko‘p ishlatiladi.

---

# PgBouncer nima qiladi?

```
App connections  
--->  PgBouncer  ---> PostgreSQL
```

Client connection va actual DB connection’ni ajratadi.

---

# Pooling modes (PgBouncer)

## Session pooling

Connection clientniki bo‘lib qoladi session davomida.

Safe, lekin kam efficient.

---

## Transaction pooling

Transaction tugashi bilan connection qaytariladi.

Eng mashhur mode.

Performance juda yaxshi.

---

## Statement pooling

Har statementdan keyin release.

Judayam aggressive.

Ko‘p applar bilan incompatible.

---

# Qachon pooling kerak?

Deyarli har production system’da.

Ayniqsa:

- Django
- FastAPI
- Node.js
- Go microservices
- High traffic APIs

---

# Qachon oddiy open/close yetadi?

- CLI script
- Small cron job
- One-time migration
- Local toy project

---

# Muhim production muammolari

## 1. Connection leak

Connection poolga qaytmay qoladi.

```python
conn = pool.getconn() # forgot:pool.putconn(conn)
```

Natija:

- pool exhaustion
- app hangs

---

## 2. Idle connections

Juda ko‘p idle connection RAM yeydi.

---

## 3. Long transactions

Pooldagi connection uzoq vaqt band bo‘lib qoladi.

Boshqa requestlar kutadi.

---

# PostgreSQL’da nega connection qimmat?

Postgres:

- thread emas
- har connection uchun alohida process yaratadi

Shuning uchun:

- memory expensive
- context switch expensive

MySQL’da bu biroz yengilroq.

---

# Qisqa taqqoslash

|Feature|Open/Close|Pooling|
|---|---|---|
|Performance|Sekin|Tez|
|Resource usage|Yuqori|Past|
|Scalability|Yomon|Yaxshi|
|Production readiness|Yomon|Standard|
|Setup|Oson|Biroz murakkab|

---

# Modern stack

Ko‘p production stack:

```
App -> PgBouncer -> PostgreSQL
```

Bu deyarli standard architecture hisoblanadi.