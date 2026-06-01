Type: #ACID

Atomicity — asosiy g‘oya: barcha transaction amallari **bajarilsin** yoki **umuman bajarilmasin.**

Aytaylik, bizda 100 ta query ishga tushishi kerak. Shulardan bittasi ham fail bo‘lsa, barcha query **ROLLBACK** bo‘lishi kerak.

Successful transaction paytida, tasavvur qiling, database qulab tushdi. Shunday holatlarda ham query **ROLLBACK** bo‘lishi kerak.

Adashib database restart bo‘lib ketsa ham, transactionga tegishli barcha narsani clean up qilishimiz kerak.