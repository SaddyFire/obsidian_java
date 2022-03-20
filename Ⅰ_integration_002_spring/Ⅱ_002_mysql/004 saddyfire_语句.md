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

-- join连接 / 外连接查询转换
select up.university,qd.difficult_level,
count(qpd.question_id) / count(distinct qpd.device_id) as avg_answer_cnt
from question_practice_detail as qpd
left join user_profile as up on qpd.device_id = up.device_id
left join question_detail as qd on qpd.question_id = qd.question_id
where up.university = '山东大学'
group by qd.difficult_level;

SELECT t1.university,t3.difficult_level,
COUNT(t2.question_id) / COUNT(DISTINCT (t2.device_id)) as avg_answer_cnt
from user_profile as t1, question_practice_detail as t2, question_detail as t3
WHERE t1.university = '山东大学' 
and t1.device_id = t2.device_id 
and t2.question_id = t3.question_id
GROUP BY t3.difficult_level;

-- case 条件函数 及其他写法(@-1)
select (case when age >= 25 then '25岁及以上' else '25岁以下' end) as age_cut,
count(id) as number 
from user_profile
group by age_cut
-- if -- -- -- -- -- 
select if(age>=25,'25岁及以上','25岁以下') as age_cut,
count(id) as number
from user_profile
group by age_cut

-- case 条件函数 (@-2)
select device_id , gender , 
(case when age >= 25 then '25岁及以上' 
     when age >= 20 then '20-24岁'
     when age < 20 then '20岁以下'
     else '其他' end ) as age_cut
from user_profile

-- 日期函数
select day(date) as day,
count(id) as question_cnt
from question_practice_detail
where month(date) = 8 and year(date) = 2021
group by date

-- if 函数条件 @-3
select if(profile like '%female', 'female', 'male') as gender,
count(device_id) as number 
from user_submit
group by gender

-- 切割、截取、删除、替换
-- -- 替换 -- --
select device_id , 
replace(blog_url,'http:/url/','') as user_name
from user_submit 
-- -- 删除 -- --
select device_id , 
trim('http:/url/'from blog_url) as user_name
from user_submit 
-- -- 字段切割 -- --(-1是指从后面开始)
select device_id , 
substring_index(blog_url,'/',-1) as user_name
from user_submit 
-- -- 多重截取 -- --
select substring_index(substring_index(profile,',',-2),',',1) as age , 
count(device_id) 
from user_submit 
group by age

```

![[Pasted image 20220320000453.png]]

![[Pasted image 20220320000421.png]]
![[Pasted image 20220320003348.png]]