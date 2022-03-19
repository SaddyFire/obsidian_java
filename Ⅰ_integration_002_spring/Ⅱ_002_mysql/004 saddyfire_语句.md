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

-- 先分组 后进行条件查询 使用having 同时 注意having的位置
select university, 
round(avg(question_cnt),3) as avg_question_cnt, 
round(avg(answer_cnt),3) as avg_answer_cnt
from user_profile
group by university
having avg_question_cnt < 5 or avg_answer_cnt < 20

-- 先进行 group by 后 order by
select up.university , 
count(qpd.question_id)/count(distinct qpd.device_id)
from user_profile as up join question_practice_detail as qpd on up.device_id = qpd.device_id
group by up.university
order by university

-- 用户信息表：user_profile
-- 题库练习明细表：question_practice_detail
-- question_detail
select up.university , qd.difficult_level, 
count(up.device_id)/ count(distinct qpd.device_id) as avg_answer_cnt
from question_practice_detail as qpd 
left join  user_profile as up on qpd.device_id = up.device_id 
left join question_detail as qd on qpd.question_id = qd.question_id
group by up.university , qd.difficult_level
```