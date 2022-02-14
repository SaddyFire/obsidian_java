##### 数据类型
int bigint double char varchar date

##### 存储引擎
InnoDB MyISAM MEMORY
MyISAM不支持事务和外键

##### 聚簇索引、回表、索引覆盖(联合索引)、最左匹配原则
- 聚簇索引: 
存储引擎表示不同的数据在磁盘的不同组织形式, 但是底层数据结构都是B+树
是否是聚簇索引取决于数据是否跟索引放在一起
他只有一个聚簇索引, 向innodb插入数据的时候, 必须要包含一个索引的key值, 这个索引可以是主键, 可以是唯一键, 如果都没有, 那么就是一个自生成6字节的rowid
但是如果他有多个聚簇索引, 那数据要放很多份, 因此只能有一个聚簇索引, 但是可以有很多非聚簇索引

myisam是非聚簇索引的, 他的b+数里面磁盘块放的式指针和数据行的地址

- 回表: 
假设有一张表, 有id , name, age, gender 四个字段, id 式主键, name是索引列
`select * from table where name=zhangsan`
先根据name查询id, 再根据id查询整行的记录, 走了2课B+树, 此时这种现象叫做回表
当根据普通索引查询到聚簇索引的key值之后,再根据key值再根据聚簇索引获取所有行记录

- 索引覆盖: 
`select * from table where name=zhangsan`
根据name可以直接查询到id , name两个列的值, 直接返回即可, 不需要从聚簇索引查询任何数据, 此时叫做索引覆盖

- 最左匹配: 
主键一般是一个列, 可能还有联合主键, 索引也会有组合索引
假设有一张表, 有id , name, age, gender 四个字段, id 式主键, name, age是组合索引
组合索引使用的时候必须先匹配name, 然后匹配age
`select * from table where name=? and age=?`  用
`select * from table where name=?`  用
`select * from table where age=?`  不用
`select * from table where name=? and age=?`  用  mysql内部有优化器, 会调整对应的顺序

- 组合索引(联合索引)结构:

- 索引下推


##### 脏读、幻读、不可重复读
- **脏读(Dirty Read)**: 是指在一个事务处理过程中读取了另一个未提交的事务中的数据 , 导致两次查询结果不一致。
	解决方案: 修改时加排他锁(写锁),直到事务提交后再释放,读取时加共享锁(读锁),其他事务只能读不能更.
- **幻读(Phantom Read):** 某记录是否存在，不存在，准备插入此记录，但执行 insert 时发现此记录已存在，无法插入。或不存在执行delete删除，却发现删除成功。
- **不可重复度(Non-repeatable read)**: 

**ACID** 原子性, 一致性, 隔离性, 持久性
**A** Atomicity
**C**	Consistency
**I**	 Isolation
**D**  Durability

![[Pasted image 20220214220536.png]]

##### mysql优化
- `select id from t where num=10 or num=20` 尽量少用or `select id from t where num=10 union all select id from t where num=20` 因为使用or会放弃索引引擎
- `select id from t where num in(1,2,3)` 对于连续的数值, 能用between就不用in select id from t where num between 1 and 3
- ` select id from t where name like '%abc%' ` 如果要进行大量首字母进行模糊匹配的要考虑用es
- `select id from t where num/2=100` 应改为: `select id from t where num=100*2 `避免表字段尽量不要用表达式, 把计算放到业务层, 这会导致直接放弃索引
- `select count(distinct left(city,3)/count(*) as sel3`使用前缀索引(通过索引的选择性来构建索引), 可以让b+树存储更多的数据, 口语说是索引长度要适当取值
	![[Pasted image 20220215003739.png]]


- `select retal_id, staff_id from retal where retal_date='2005-05-25' order by inventory_id, customer_id `整体利用组合索引 
   `retal_id, staff_id from retal where retal_date='2005-05-25' order by inventory_id, customer_id desc`
   使用索引扫描来排序(不懂)

- `select * from actor where actor_id = 1 or actor_id = 2`
  `select * from actor where actor_id in (1,2)`
  `select * from actor where actor_id = 1 union all select * from actor where actor_id = 2`
  union all or 都能使用索引, 推荐使用in, union会分两步执行
  or单列索引可以走, 组合索引不会走,所以推荐in
- 范围列都可以用索引 范围条件是: 见图, 但是只能用一个范围
![[Pasted image 20220215005702.png]]
- `select * from user where phone = 123`
   `select * from user where phone='123'`
   强制类型转换会全表扫描
- 更新 十分频繁, 数据区分度不高的字段上不建议建立索引, 因为数据库要维护,类似于性别类区分不大的属性,建立索引没有意义, bu'n, 区分度在80%以上可以建立
![[Pasted image 20220215010019.png]]

1. 数据表优化, 磁盘块每个磁盘块大小16kb, 表中的主键要越小越好, 因此主键用varchar和int, 如果数据小于4个字节, 用varchar, 大于4个字节, 用int
mysql锁

在满足需求的情况下,尽量主键自增, 因为这样在插入数据时不会对B+树中的磁盘块分裂



