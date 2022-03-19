```sql
-- order by 和 limit 搭配顺序
select device_id from user_profile order by 'device_id' desc limit 0,2

-- between 不用加 ()
select device_id, gender, age from user_profile where age between 20 and 23;

-- 1. count和()中间不能有空格   2. 保留小数用round()
select count(gender) as male_num ,round(avg(gpa),1) as avg_gpa from user_profile where gender = 'male'

-- 先进行 '性别' 和 '大学' 分组, 然后计算平均
select gender, university, 
count(id) as user_num , 
round(avg(active_days_within_30),1) as avg_active_day,
round(avg(question_cnt),1) as avg_question_cnt
from user_profile
group by gender ,university


```