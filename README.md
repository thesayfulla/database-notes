# Database Notes

A comprehensive collection of database concepts, architecture, and best practices. This repository contains detailed notes on key database topics including ACID properties, indexing, replication, and optimization techniques.

## Contents

### Core Concepts
- **ACID Properties**: Atomicity, Consistency, Isolation, Durability
- **Transactions**: What is a Transaction, Long-running Transaction Costs
- **Locks**: Shared vs Exclusive Locks, Two-Phase Locking (2PL) in PostgreSQL

### Indexing & Query Optimization
- **Indexing Fundamentals**: B-Tree vs B+Tree, Index types
- **Query Planning**: SQL query planner, optimizer, How Database Optimizers Decide to Use Indexes
- **Scan Types**: Bitmap index scan vs index scan vs table scan, Index Scan vs Index Only Scan
- **Index Design**: Key vs Non-key column database indexing, Create Index Concurrently

### Data Storage & Architecture
- **Storage Models**: Row based & Column based DB, Database Pages
- **Caching**: Memcached In-Memory Cache Architecture, Database connection pooling
- **Pagination**: Offset pagination is slow

### Scaling & Replication
- **Partitioning**: Vertical vs Horizontal Partitioning, Sharding
- **Replication**: Master-Standby Replication, Multi Master Replication
- **Advanced Topics**: MongoDB Clustered Collections, Database Cursors

## Folder Structure

```
├── Consistency/          # Consistency-related topics
├── Isolation/            # Isolation level concepts
├── Replication/          # Replication strategies
└── *.md                  # Individual topic files
```

## How to Use

- Start with **Menu - Index.md** for an overview
- Reference individual files for deep dives into specific topics
- Use folder collections for related concept groups

## License

These are study notes. Feel free to reference, share, and contribute improvements.

---

*Last updated: June 2026*
