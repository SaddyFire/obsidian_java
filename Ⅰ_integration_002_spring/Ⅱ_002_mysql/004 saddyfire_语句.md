```sql
-- order by 和 limit 搭配顺序
select device_id from user_profile order by 'device_id' desc limit 0,2

-- between 不用加 ()
select device_id, gender, age from user_profile where age between 20 and 23;

-- 1. count和()中间不能有空格   2. 保留小数用round()
select count(gender) as male_num ,round(avg(gpa),1) as avg_gpa from user_profile where gender = 'male'
```