# Multi-Master Replication

![Multi-Master Replication Architecture](./Replication/Pasted%20image%2020260529121159.png)
# Definition
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

```
Conflict Resolution
```

Example:

```
User updated same row on two masters
at same time
```

System must decide:

- last write wins?
- merge?
- reject?

### Common Issues

- Data conflicts
- Replication loops
- Complex consistency management

### Real Usage

```
Global applications
Multi-region systems
```

### Comparison

|Feature|Master-Standby|Multi-Master|
|---|---|---|
|Writes|One node|Multiple nodes|
|Complexity|Lower|Higher|
|Conflict Risk|Low|High|
|Scaling|Read scaling|Read + Write scaling|
