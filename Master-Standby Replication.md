![[Pasted image 20260529120508.png]]Bitta asosiy **master** bo'ladi, u asosan DDLs uchun ishlatilinadi. lekin qolgan **standby**larni esa shunchaki reading operation uchun ishlatsak bo'ladi holos.
### CORE LOGIC:
- Master (Primary) = main database
- Standby (Replica) = copied database
- PostgreSQL uses WAL replication
- Data changes stream from Primary → Replica
- Replica usually handles READ queries
- Used for:
    - High Availability
    - Read Scaling
    - Backup

### Async Replication

```
Primary COMMIT -> User OK -> Replica later
```

- Faster
- Possible small data loss

### Sync Replication

```
Primary -> Replica ACK -> COMMIT
```

- Safer
- Slower

### Common Problem

```
Replication Lag
```

Replica can be behind primary for a short time.