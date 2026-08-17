# Sharding

Sharding — bu bitta table’ni bir nechta bo‘lakka bo‘lish, lekin endilikda server levelda.

Aytaylik, **users** degan table bor. Uni olib shard1 degan alohida serverdagi databasega, shard2 degan yana alohida databasega bo‘lib tashlaymiz.

Shardlarni aniqlash va connection yaratish uchun **hashing** ishlatamiz.
