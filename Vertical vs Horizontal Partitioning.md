# Vertical vs Horizontal Partitioning

## Horizontal Partitioning

Table row’lar bo‘yicha bo‘linadi.

### Example

```text
orders_2024
orders_2025
orders_2026
```

Har partition’da bir xil columns bo‘ladi.

### Qachon ishlatiladi?

- Huge rows
- Time-series data
- Logs / events / orders

### Afzalliklari

- Faster queries
- Easier archive/delete
- Better scalability

### Kamchiliklari

- Cross-partition query qimmat
- Setup murakkabroq

### Vizual

![Horizontal partitioning](https://images.openai.com/static-rsc-4/EWipxEHAzhyZY9Hxp8euvw49uG03oFRVcw5Xozj0VenQPvMyixw3suGvtjk7MqGsq5XcvQCKedf9Y-UAq-FPzh0GrXh_39BpZ4WGcQHsPNznMI2wSqUTFM4AqHO4uYrpBSTSJ4aH3Qtdmqfe4gV7bImwLYrPnFAJF0FMjln7lG0?purpose=inline)

## Vertical Partitioning

Table column’lar bo‘yicha bo‘linadi.

### Example

users_main:

```text
id
username
email
```

users_profile:

```text
bio
avatar
settings
```

### Qachon ishlatiladi?

- Wide tables
- Large TEXT/JSON/BLOB
- Rarely used columns

### Afzalliklari

- Smaller row size
- Better cache usage
- Faster reads

### Kamchiliklari

- JOIN kerak bo‘ladi
- Query complexity oshadi

### Vizual

![Vertical partitioning](https://images.openai.com/static-rsc-4/OpyNsGcIAiBJuYwXvq-iOO-gyctm8pVwbXNjOuNpLWDY14wfmBVlrNMTmPEOSTUVj-ex6m5qKX3pPUMbQKwPzNiURKk8ZBMYP24ADkASyiITnIIHgfFEETW9Lm_11X1Odd9qjGNTxqZdevnKRg83YRjxPx4k5ZwsDLY9A9H-L1U?purpose=inline)

## Farqi

| Horizontal             | Vertical                   |
| ---------------------- | -------------------------- |
| Rows split             | Columns split              |
| Huge data volume       | Huge row size              |
| `orders_2025`          | `users_profile`            |
| Mostly time/date based | Mostly heavy columns based |
