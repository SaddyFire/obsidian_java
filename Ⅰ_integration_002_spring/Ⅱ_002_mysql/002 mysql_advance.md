## day02 MySQL高级

### 1. 约束 (constraint)

(建议用navicat)

非空约束(not null) , 唯一约束(unique) , 主键约束(primary key) , 检查约束(check) , 默认约束(default), `外键约束(foreign key)`, 自增(auto_increment)

```sql
-- 员工表
CREATE TABLE emp (
  id INT PRIMARY KEY auto_increment, -- 员工id，主键且自增长
  ename VARCHAR(50) NOT NULL UNIQUE, -- 员工姓名，非空并且唯一
  joindate DATE NOT NULL , -- 入职日期，非空
  salary DOUBLE(7,2) NOT NULL , -- 工资，非空
  bonus DOUBLE(7,2) DEFAULT 0 -- 奖金，如果没有奖金默认为0
);
```

### 3. 多表查询 √

#### 3.1 联表
```sql
-- 子查询
select c_name,grade from score where stu_id = (SELECT id from student where name = '李四' );
#隐式连接
select u.id, u.name, u.age, o.number from user u,orderlist o where u.id = o.uid;
-- ------------------------------------------------------

#显式内连接
select u.id,name,age,number from user u join orderlist o on u.id = o.uid;

-- 左/右显
select u.id,name,age,number from user u left/right join orderlist o on u.id = o.uid;

-- ****************************************************
15.查询同时参加计算机和英语考试的学生的信息
-- 分组,having,count,子查询
-- 选择查询学生参加 计算机和英语的考试数
select * from student where id in (select stu_id from score where c_name in ('计算机','英语') group by stu_id HAVING count(*) = 2)

-- ****************************************************
-- 三表查询,等级匹配
-- 查询员工姓名，工资，职务名称，职务描述，部门名称，部门位置，工资等级
select e.ename,e.salary, j.jname,j.description,d.dname,d.loc ,s.grade from emp e join dept d on e.dept_id = d.did join job j on j.id = e.job_id join salarygrade s on s.losalary <= e.salary <= s.hisalary;

-- ****************************************************
-- 查询出部门编号、部门名称、部门位置、部门人数
select d.did,d.dname,d.loc, count(*) count from dept d join emp e on d.did = e.dept_id group by d.did;
```

#### 3.2 union
```sql
(SELECT * from student  ORDER BY score DESC limit 0,2 ) UNION all ( SELECT * from student  ORDER BY score2 DESC limit 0,2)
```

#### 3.3 些许注意点
```sql

```
