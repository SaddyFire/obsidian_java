##### redis的持久化
RDB: Redis Database Backup file
AOF: Append Only File  -->  bgrewriteaof

##### 什么是缓存穿透、缓存雪崩、缓存击穿？
缓存穿透是指反复查询不存在的数据, 导致数据库压力过大. 
