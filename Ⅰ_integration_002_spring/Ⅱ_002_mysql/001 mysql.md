## 1 DDL
**Data Definition Language** : 数据定义语言
</br>

## 2 DML
**Data Manipulation Language** : 数据操作语言

```sql
insert into stu (id,name) values (1,'张三');	-- 给指定列添加数据

insert into stu values(id,name...)	-- 给全部列表添加数据

insert into stu(id,name) values (1,'张三')	-- 批量添加数据

update stu set sex = '女'... where name = '张三';		-- 修改表数据

delete from stu where name = '张三';		-- 删除数据
```
</br>

## 3 DQL
**Data Query Language** : 数据查询语言

##### 3.3.1 基本查询

```sql
select * from homework01;	-- 全部查询
SELECT pid,pname from homework01;	-- 基本查询
SELECT pid,pname,price+10,category_name FROM homework01 ; -- 运算查询
select distinct * from homework01;		-- 去重查询

SELECT count(*) FROM 
(SELECT DISTINCT user_id FROM tb_log WHERE log_time > '2020-09-15') 
as id_class -- 去重计数查询

select name,math as 数学成绩,english as 英文成绩 from stu;	-- 起别名
```

##### 3.3.2 条件查询(where)

```sql
SELECT 字段列表 FROM 表名 WHERE 条件列表;		-- 条件查询
select * from homework01 where price BETWEEN 200 and 1000;	-- 区间查询
select * from stu where age in (18,20 ,22);	-- 选择查询
select * from stu where age not in (18,20 ,22);	-- 选择查询
#模糊查询
select * from stu where name like '马%';		-- 模糊查询'_花%''%德%'
select * from emp where ename not like '%R%';
select * from stu where category_name is null; -- null查找
select * from stu where category_name is not null; -- null查找
```

##### 3.3.3 排序查询(order by)

```sql
select * from stu order by age ASC;		-- 默认升序
select * from stu order by math desc ;
select * from stu order by math desc , english asc ;	-- 升序
```

##### 3.3.4 聚合函数

*(null值不参与所有聚合函数运算)*

```sql
select count(*) from stu;		-- 统计所有数据
select max(math) from stu;		-- 查询(math)最大值
select sum(math) from stu;		-- 查询(math)总和
select avg(math) from stu;		-- 查询(math)平均分
select min(english) from stu;	-- 查询(english)最小值
```

##### 3.3.5 分组查询(group by)

where… group by… having…

```sql
select sex, avg(math) from stu group by sex;	-- 男同学和女同学各自的数学平均分

select sex, avg(math),count(*) from stu group by sex;	-- 查询男同学和女同学各自的数学平均分, 以及各自人数

select sex, avg(math),count(*) from stu where math > 70 group by sex;		-- 查询男同学和女同学各自的数学平均分，以及各自人数，要求：分数低于70分的不参与分组

select sex, avg(math),count(*) from stu where math > 70 group by sex having count(*)  > 2;		-- 查询男同学和女同学各自的数学平均分，以及各自人数，要求：分数低于70分的不参与分组，分组之后人数大于2个的
```

> where > 聚合函数 > group by

##### 3.3.6 分页查询(limit)

```sql
select * from stu limit 0,3;	-- 从0开始查询,查询3条数据
select * from stu limit 0,3;	-- 每页显示3条数据,查询第2页数据
                    -- 起始索引 = (当前页码 - 1) * 每页显示的条数
```

##### `3.3.7 好题积累`

```sql
#查询结果是表达式（运算查询）：将所有商品的价格+10元进行显示.
SELECT pid,pname,price+10,category_name FROM homework01 ;
----------------------------------------------------------
#查询商品价格是200或800或者2000的所有商品
select * from homework01 where price in (200 ,800, 2000); 
----------------------------------------------------------
select * from homework02 where sid not in ('S_1001','S_1002','S_1003');
----------------------------------------------------------
#11.名字中不包含R字符的员工信息。
select * from emp where ename not like '%R%';
----------------------------------------------------------
#14.计算员工的日薪(按30天)。
select EName,sal/30 as 日薪 from emp ;
----------------------------------------------------------

```

## 4 DCL
**Data Control Language** : 数据控制语言




