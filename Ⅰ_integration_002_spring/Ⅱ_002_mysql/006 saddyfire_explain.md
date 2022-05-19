## 1. 索引




## 2. 优化
```sql
/*正常写法*/
DELETE from activity where id not in ( SELECT id from activity_data);

/*优化后写法*/
DELETE from activity where id not in (select * from (SELECT id from activity_data) t);
```