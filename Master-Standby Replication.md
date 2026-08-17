# Master-Standby Replication

![Master-Standby replication architecture](./Replication/Pasted%20image%2020260529120508.png)

Bitta asosiy **master** bo'ladi, u asosan DDL’lar uchun ishlatiladi. Qolgan **standby**’larni esa shunchaki reading operation uchun ishlatsak bo'ladi, xolos.

### Core logic

- Master (Primary) = main database
- Standby (Replica) = copied database
- PostgreSQL uses WAL replication
- Data changes stream from Primary → Replica
- Replica usually handles READ queries
- Used for:
    - High Availability
    - Read Scaling
    - Backup

### Async replication

```text
Primary COMMIT -> User OK -> Replica later
```

- Faster
- Possible small data loss

### Sync replication

```text
Primary -> Replica ACK -> COMMIT
```

- Safer
- Slower

### Common problem

```text
Replication Lag
```

Replica can be behind primary for a short time.
