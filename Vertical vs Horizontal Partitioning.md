Type: Partitioning
## Horizontal Partitioning

Table row’lar bo‘yicha bo‘linadi.
### Example

```
orders_2024
orders_2025
orders_2026
```

Har partition’da bir xil columns bo‘ladi.

---

## Qachon ishlatiladi?

- Huge rows
- Time-series data
- Logs / events / orders

---

## Afzalliklari

- Faster queries
- Easier archive/delete
- Better scalability

---

## Kamchiliklari

- Cross-partition query qimmat
- Setup murakkabroq

---

## Vizual

![https://images.openai.com/static-rsc-4/A98mAGlqF_Ijxq4L-mKny4LNAQG1Ksoy0oHPZpYJbHVuJK6HtyrMqD3_0ujL0QuqQhzsX7yXIP73jMMUBiaLo9_4VILcXRr-w46Li75cJJJLJk-2AUeCaVXMEcYVG_gfMzGblEBd98rYjV-dBmbxULz06RzzmRXQC6mXr4HXwhc5ImtwFgOfjrbPGWxjYSGR?purpose=fullsize](https://images.openai.com/static-rsc-4/EWipxEHAzhyZY9Hxp8euvw49uG03oFRVcw5Xozj0VenQPvMyixw3suGvtjk7MqGsq5XcvQCKedf9Y-UAq-FPzh0GrXh_39BpZ4WGcQHsPNznMI2wSqUTFM4AqHO4uYrpBSTSJ4aH3Qtdmqfe4gV7bImwLYrPnFAJF0FMjln7lG0?purpose=inline)

---

# Vertical Partitioning

Table column’lar bo‘yicha bo‘linadi.

### Example

### users_main

```
id
username
email
```

### users_profile

```
bio
avatar
settings
```

---

## Qachon ishlatiladi?

- Wide tables
- Large TEXT/JSON/BLOB
- Rarely used columns

---

## Afzalliklari

- Smaller row size
- Better cache usage
- Faster reads

---

## Kamchiliklari

- JOIN kerak bo‘ladi
- Query complexity oshadi

---

## Vizual

![https://images.openai.com/static-rsc-4/m-QJ92Cdir-rovpZEdGikwQeNJLh2QyhmDxdl12byQoV4mYz9Iui70dbPRqNHowPI_Rod6_3beNCjSrezXKUcU8xZaWFcS8dYVjqgRs2TaWiuZFcKmjuy4g4ZGrHgQ2mWNYinv4EoUEttrG2ye2nSpK7fGDaVmHOjVUjlrAhIpOCFCuy0S69T6JvR97U3dW7?purpose=fullsize](https://images.openai.com/static-rsc-4/OpyNsGcIAiBJuYwXvq-iOO-gyctm8pVwbXNjOuNpLWDY14wfmBVlrNMTmPEOSTUVj-ex6m5qKX3pPUMbQKwPzNiURKk8ZBMYP24ADkASyiITnIIHgfFEETW9Lm_11X1Odd9qjGNTxqZdevnKRg83YRjxPx4k5ZwsDLY9A9H-L1U?purpose=inline)

---

# Farqi

| Horizontal             | Vertical                   |
| ---------------------- | -------------------------- |
| Rows split             | Columns split              |
| Huge data volume       | Huge row size              |
| `orders_2025`          | `users_profile`            |
| Mostly time/date based | Mostly heavy columns based |