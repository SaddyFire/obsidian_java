```sql
-- regexp 匹配
select prod_name,prod_desc from Products
where prod_desc REGEXP 'toy';
-- locate 函数
select prod_name , prod_desc from Products
# where prod_desc like '%toy%';
# where instr(prod_desc , 'toy') > 0;
# having instr(prod_desc , 'toy');
having locate('toy' , prod_desc);



-- case 条件函数 及其他写法(@-1)
select (case when age >= 25 then '25岁及以上' else '25岁以下' end) as age_cut, count(id) as number 
from user_profile group by age_cut
-- if -- -- -- -- -- 
select if(age>=25,'25岁及以上','25岁以下') as age_cut, count(id) as number
from user_profile group by age_cut

-- case 条件函数 (@-2)
select device_id , gender , 
(case when age >= 25 then '25岁及以上' when age >= 20 then '20-24岁' when age < 20 then '20岁以下' else '其他' end ) as age_cut
from user_profile

-- 日期函数
select day(date) as day, count(id) as question_cnt
from question_practice_detail where month(date) = 8 and year(date) = 2021 group by date

-- 日期函数2  date_add(qpd1.date1,interval 1 day) = qpd2.date2
select count(date2) / count(date1) as avg_ret from 
(select distinct device_id , date as date1 from question_practice_detail) 
as qpd1 
left join (select distinct device_id , date as date2 from question_practice_detail as qpd2) 
as qpd2 
on qpd1.device_id = qpd2.device_id and date_add(qpd1.date1,interval 1 day) = qpd2.date2


-- if 函数条件 @-3
select if(profile like '%female', 'female', 'male') as gender,
count(device_id) as number 
from user_submit
group by gender

-- 切割、截取、删除、替换
-- -- replace() -- --
select device_id , 
replace(blog_url,'http:/url/','') as user_name
from user_submit 
-- -- trim() -- --
select device_id , 
trim('http:/url/'from blog_url) as user_name
from user_submit 
-- -- substring_index -- --(-1是指从后面开始)
select device_id , 
substring_index(blog_url,'/',-1) as user_name
from user_submit 
-- -- 多重截取 -- --
select substring_index(substring_index(profile,',',-2),',',1) as age , 
count(device_id) 
from user_submit 
group by age


-- 求复旦大学的每个用户在8月份练习的总题目数和回答正确的题目数情况，请取出相应明细数据，对于在8月份没有练习过的用户，答题数结果返回0.
-- 此处要注意 month(qpd.date) = 8 的条件要加在join中, 原因见下图
select up.device_id, up.university , 
    count(qpd.question_id) as question_cnt,
    sum(if(qpd.result='right',1,0)) as right_question_cnt
from question_practice_detail as qpd 
right join user_profile as up on qpd.device_id = up.device_id and month(qpd.date) = 8
where up.university = '复旦大学' 
group by up.device_id

-- 浙江大学的用户在不同难度题目下答题的正确率情况  avg
select qd.difficult_level , (avg(if(qpd.result = 'right',1,0))) as correct_rate
from question_practice_detail as qpd 
left join user_profile as up on qpd.device_id = up.device_id
left join question_detail as qd on qd.question_id = qpd.question_id
where up.university = '浙江大学'
group by qd.difficult_level
order by correct_rate


-- DATEDIFF函数(待更新)

```
[[002 达梦语法]]

![[Pasted image 20220320000453.png]]

![[Pasted image 20220320000421.png]]
![[Pasted image 20220320003348.png]]
![[Pasted image 20220320130848.png]]