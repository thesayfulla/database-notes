# Multi-Master Replication

![Multi-Master replication architecture](./Replication/Pasted%20image%2020260529121159.png)

### Definition

- Multiple databases can WRITE
- Each master replicates changes to others
- Every node can handle:
    - READ
    - WRITE

### Advantages

- High Availability
- Better write scaling
- Multiple active regions/datacenters

### Problems

```text
Conflict Resolution
```

Example:

```text
User updated same row on two masters
at same time
```

System must decide:

- last write wins?
- merge?
- reject?

### Common issues

- Data conflicts
- Replication loops
- Complex consistency management

### Real usage

```text
Global applications
Multi-region systems
```

### Comparison

| Feature       | Master-Standby | Multi-Master          |
| ------------- | -------------- | --------------------- |
| Writes        | One node       | Multiple nodes        |
| Complexity    | Lower          | Higher                |
| Conflict Risk | Low            | High                  |
| Scaling       | Read scaling   | Read + Write scaling  |
