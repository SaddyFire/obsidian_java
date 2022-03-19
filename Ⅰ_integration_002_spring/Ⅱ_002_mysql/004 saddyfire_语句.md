```sql
-- order by 和 limit 搭配顺序
select device_id from user_profile order by 'device_id' desc limit 0,2

-- between 不用加 ()
select device_id, gender, age from user_profile where age between 20 and 23;


```