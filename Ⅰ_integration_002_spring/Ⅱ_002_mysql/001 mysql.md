## 1 DDL
**Data Definition Language** : 数据定义语言

## 2 DML
**Data Manipulation Language** : 数据操作语言

```sql
insert into stu (id,name) values (1,'张三');	-- 给指定列添加数据

insert into stu values(id,name...)	-- 给全部列表添加数据

insert into stu(id,name) values (1,'张三')	-- 批量添加数据

update stu set sex = '女'... where name = '张三';		-- 修改表数据

delete from stu where name = '张三';		-- 删除数据
```

## 3 DQL
**Data Query Language** : 数据查询语言


##### 3.3.4 聚合函数

- (null值不参与所有聚合函数运算)

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
select sex, avg(math),count(*) from stu where math > 70 group by sex having count(*)  > 2;		-- 查询男同学和女同学各自的数学平均分，以及各自人数，要求：分数低于70分的不参与分组，分组之后人数大于2个的
```

> where > 聚合函数 > group by


##### `3.3.7 好题积累`

```sql
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




