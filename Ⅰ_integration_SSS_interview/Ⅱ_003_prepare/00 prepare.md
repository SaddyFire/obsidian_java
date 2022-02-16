##### 介绍下HashMap
![[Pasted image 20220213195738.png]]
	
	首先hashmap是一种key-value键值对形式的集合, 在1.7之前他的底层是数组加链表, 1.8之后他的底层是数组, 链表和红黑树. 
	在把数据存入hashmap中时, 会先通过该数据key的hash值进行哈希函数, 类似于对数字的取模取余, 如果得到的角标位置已经有数据了, 也就是hash冲突, 此时会以链表的形式挂下来.
	在1.7之前, 如果链表过长,就会导致每次查询要一个个撸下来, 而1.8之后采用红黑树方式的查询效率就会大大提高. 但是数据少的时候树的结点大小比链表要大, 所以在一个哈希桶数量大于8的时候才会变为红黑树, 小于6的时候会转回链表. 
	还有一点是1.7的hashmap是头插法,速度快,但是在多线程并发和数组扩容的时候会出现循环列表, 导致死锁问题, 所以在1.8之后的红黑树用尾插法, 能相对避免死锁. 在项目中的话, 如果有并发需求, 我们通常会采用concurrentHashmap, 他的底层是对hashmap的哈希桶上锁, 达到多线程安全.
	
##### 四大线程池

newSingleThreadExecutor
newFixedThreadPool
newCachedThreadPool
newScheduledThreadPool

##### 线程池七大参数
- 核心线程数
- 最大线程数
- 临时线程最大存活时间
- 时间单位
- 等待队列
- 创建线程工厂
- 任务拒绝策略


##### 介绍下JVM

	jvm是指Java virtual Machine, 在一个class文件加载到java虚拟机中, 要经历loading , linking, initializing, 
	其中首先虚拟机要将class文件load进classloader, 加载器会通过双亲委派模型, 从costumclassloader到appClassloader最终到bootstrap, 这样做最终要的原因是安全, 防止底层类库被篡改.同时如果经过了双亲委派模型最后也没找到该类, 则会报classnotfound异常. class文件load进内存之后会进行verification,preparation和resolution,进行字节码文件的解析, initializing是对静态变量的初始化赋值.
	java运行后会进入jvm运行时数据区,

##### JMM
JMM是抽象概念, main memory 和 working memory
多个线程共享的内存叫主内存, 在物理结构中叫堆heap
但是每个线程各自操作数据的时候, 会有一个各自的work memory , 操作的是从主内存中拷贝过来的副本, 而且所有操作是要在自己的工作内存进行

##### GC
假设一个场景, Student s1 = new Student();
Student s2 = new Student();
这个时候刚才的Student()对象就没有别人的指引, 也叫野指针,
java包含了编译型和解释型语言
垃圾回收器分为: 新生代 + 老年代 + 永久代（1.7）Perm Generation/ 元数据区(1.8) Metaspace
新生代 是 Eden + 2个suvivor区
第一次扫描garbage的时候新生代回收之后, 大多会被回收, 活着的就进入suvivor0
再次扫描, 或者的从eden 和s0 进入s1
再次扫描, eden 和s1 会重新进入s0
当达到一定程度之后, 会进入老年代

常见的GC回收器:
Serial 和 SerialOld
现在默认的回收器是Parallel Scavenge 和Parallel Old
ParNew和CMS
再后来就是GI

##### volatile关键字
volatile可以保证多线程之间数据的可见性和一定有序性,  因为变量被volatile修饰过后, 当这个数据要进行写操作, JVM会发送一条lock指令给CPU,CPU计算完数据之后会立刻把这个数据写回到主内存, 而且mesi协议会分别给每个线程各自的这个数据都打标记, 例如你的modify就是我的invalid, 如果有invalid标记那我就会回到主内存去读数据

##### JVM调优
-Xmx3350m: 设置JVM最大可用内存为3550M
-Xms3350m: 设置JVM初始内存为3550M
-Xmn2g: 设置年青代大小为2g 整个JVM内存大小=年青代+ 老年代+ 持久代大小, sum官方推荐配置整个堆的3/8
-Xss256k: 设置每个线程的栈大小, 根据应用线程所需要的内存大小调整, 满足需要的情况下, 越小能产生更多的线程

##### 谈谈对spring的理解
spring广义上来说是一个完整的生态, spring对项目的开发有超强的拓展性,spring家族让开发进入了春天时代. 侠义的spring框架核心主要是ioc和aop, ioc是指inversion of control 也就是控制反转, 本质是一种思想, 是一种程序之间的解耦思想,将对象的创建权力反转给spring. ioc也是工厂模式, 单例模式, 模板方法, 观察者设计模式的一种实现. 
aop是aspect 

##### spring的bean生命周期
首先spring是一个轻量级框架, 帮我们简化开发, 核心功能是ioc和aop
bean的生命周期只是ioc创建对象的一个流程,里面大体来说可以包含实例化, 初始化, 使用, 销毁
如果说细粒度的的话, 通过反射的方式读取到`beandefinition`的相关信息,把这些对象创建出来, 然后通过反射的方式加载类, 创建对象, 此时的创建只是在堆内存中申请了一块内存空间, 
所以此时的值只是默认值,接下来会通过一个方法, 叫`populateBean()`方法来给当前属性赋值, 会调用相应set()方法完成属性的赋值, 但是这部分只会完成自定义属性的操作, 还有一部分属性叫做容器属性, 是spring框架给我们定义的自定义对象, 所以这时候还会调用一个`aware()`接口, 通过这个接口可以帮我们判断是否能进行属性值的设置工作, 所以相当于populatebean()和invokeAwareMethods()方法都完成了属性的赋值工作, 一个是自定义属性的赋值, 一个是容器属性的赋值, 当这个时候完成之后, 正常情况下bean对象已经可以正常使用了
但是这里spring设计的时候考虑到了一个特点, 比如拓展性, 当bean对象创建过后可能会对bean对象有一些扩展的过程, 比如aop的实现, 其实就是对我们生成的bean对象有一定的拓展操作, `PostProcessor`接口里面有after()before()方法来进行增强, aop的底层动态代理也是在这里实现的
当这些步骤都完成之后, 就可以完成bean的创建了, 这是我可以通过Context.getBean()来调取.
当最后关闭容器的时候, 会调用close()方法, 把对象销毁掉
![[Pasted image 20220215163227.png]]
![[Pasted image 20220215163013.png]]

##### spring 的 factorybean 和 beanfactory
beanfactory是spring容器的入口, 定义了一些列的接口规范, 对象的创建经过一个完整的复杂生命周期过程, 可以说是一种流水线的工作流程

factorybean更像是一种定制化, 可以自己定义FactoryBean来实现Bean的特殊要求, 自己可以自定义, 不需要完全遵循bean的生命周期

##### spring是怎么解决循环依赖的/为什么要使用三级缓存来解决循环依赖问题
循环依赖是指A对象里有个属性是b, B对象里有个属性是a,
此时这是一个闭环问题, 如果要解决这个问题我们要想, 从形成环的那个步骤把这个闭环干掉
假设持有了某一个对象的引用, 能否再后续步骤的时候通过该引用给对象的赋值属性
这里我们可以定义一个概念: 按照对象的状态进行分类, 分为成品, 半成品, 一个是完成品和半成品
所以这是我就可以再闭环链路的判断容器是否有A或者B的那个节点打开, 虽然没有成品, 但是有半成品, 有地址值
还有一个概念是三级缓存, 就是三个map结构, 就是说我前面说的一旦我初始化A或者B, 我就立刻把这个半成品对象放到map里, 那这个时候在每次判断我有没有A或者B的时候, 我都有了半成品, 这个环一走, 我都能进行赋值
所以其实本质上最后能解决循环依赖问题的, 是spring的生命周期中, 实例化和初始化时分开的
![[Pasted image 20220215224553.png]]

##### 接口抽象类的差别
- 接口: 自上向下: 定义空方法, 空约束,先定义规范,然后实现具体方法
- 抽象类: 自下向上: 我已经现有n种不同的子类, 然后抽取公共部分


##### 谈一下SpringMVC的加载过程
SpringMVC是指spring module view controller, 也叫模型视图控制器, 把web层进行解耦, 简化开发
首先springMVC的入口是DispatcherServlet(`package org.springframework.web.servlet`)前端控制器, 其中有核心方法`initStrategies()`初始化了映射处理器, `doDispatch() `处理请求方法, 并且获取到`HandlerAdapter`, 也就是处理器适配器, 最终调用了后端处理器`handle()`方法处理请求, 并返回`ModelAndView`模型视图

##### SpringBoot的工作原理/自动配置/SPI机制是怎么样子的？
首先SPI 是Service Provider Interface, 是一种服务发现机制, springboot的底层也使用到了spi机制
`@SpringBootApplication`  -> `@EnableAutoConfiguration` -> `@import` 中导入的类会被加载到spring IOC容器中 ->  最终会被spring反射调用`classLoader`类中的的getResourcees() 把`META-INFO`下的`spring.factories`文件里的类load进jvm


##### 谈一谈mysql
讲mysql我想从首先mysql是一个关系型数据库, 关系型数据库和nosql在本质的区别是一个偏于存储硬盘, 一个偏于存储在内存, 因为是基于硬盘存储.
讲到mysql要聊到mysql的存储引擎, 在mysql5.5之后, 默认引擎改成了innodb, 之前是myisom, 存储引擎是一种存储数据文件的形式, 源文件里面innodb是2个, myisom是3个, 因为innodb采用了聚簇索引的方式, 当然他们两个底层都是b+树, 但是innodb把行数据放在了b+树的叶子节点. myisom的磁盘块放的是指针和数据行的地址. 
因为有了聚簇索引这里有牵扯到一个概念叫回表, B+树的聚簇索引表一定会有一张, 可以是主键, 可以是唯一键, 如果都没有就会默认给你生成一个6字节的rowid, 所以比方`select * from xx where name='zhangsan'` 如果我name是索引, 那我会先在name索引表里查到这行数据的聚簇索引id, 然后再回聚簇索引里查这个行数据. 讲到这又有一个索引覆盖, 就是我不回表, 直接从name里面查id 和name, 这样就不用回表


##### 谈一谈mysql的事务
首先mysql 的 事务是关系型数据库和nosql数据库的根本区别, 它可以用来维护数据操作的安全, 能够保证一系列操作的要么完全成功, 要么完全失败
事务的四大特性是ACID, mysql中只有innodb是支持事务的

数据库优化
1.  数据表优化, 磁盘块每个磁盘块大小16kb, 表中的主键要越小越好, 因此主键用varchar和int, 如果数据小于4个字节, 用varchar, 大于4个字节, 用int  
2. 在满足需求的情况下,尽量主键自增, 因为这样在插入数据时不会对B+树中的磁盘块分裂
3. 使用前缀索引(通过索引的选择性来构建索引), 可以让b+树存储更多的数据, 口语说是索引长度要适当取值

##### mysql语句的优化
首先关于mysql数据优化之前,我们要明确musql的常用几种数据存储引擎, 在mysql5.5之后, 默认的存储引擎是innodb, innodb的底层
-   `select * from actor where actor_id = 1 or actor_id = 2`  
    `select * from actor where actor_id in (1,2)`  
    `select * from actor where actor_id = 1 union all select * from actor where actor_id = 2`  
    union all or 都能使用索引, 推荐使用in, union会分两步执行  
    or单列索引可以走, 组合索引不会走,所以推荐in
	
- `select id from t where num in(1,2,3)` 对于连续的数值, 能用between就不用in select id from t where num between 1 and 3
-  `select id from t where num/2=100` 应改为: `select id from t where num=100*2` 避免表字段尽量不要用表达式, 把计算放到业务层, 这会导致直接放弃索引
- 范围列都可以用索引 范围条件是:  但是只能用一个范围
- `select * from user where phone = 123`  
    `select * from user where phone='123'`  
    强制类型转换会全表扫描
- 更新十分频繁, 数据区分度不高的字段上不建议建立索引, 因为数据库要维护,类似于性别类区分不大的属性,建立索引没有意义, 不能有效过滤数据, 区分度在80%以上可以建立



##### mysql索引语句创建
create index 索引名 on 表名(列名)

##### 分布式事务
**TCC方案** 
比如A账户给B账户转钱, 
Try阶段: 先把两个银行账户中的资金冻结住
Confirm: A减钱 和 B加钱, 如果中间
Cancel阶段: 如果任何一个账户操作失败, 就要回滚进行补偿
但是这种方案业务耦合太多, 每个业务的try, confirm, cancel都不一样, 所以只有跟钱相关的分布式事务, 要么全部成功, 要么全部自动回滚, 严格保证资金的正确性, 用TCC方案, 维护难
**可靠消息最终一致性方案**







##### RabbitMQ使用场景
关注好友里面如果相互关注的话是会默认添加好友, 业务里是要额外查一下双方是否相互关注, 我是把加好友加到mq里了, 因为之前做过一个业务, 也是类似这种模式, 当时

##### RabbitMQ避免重复消费
- 采用乐观锁保证幂等, 在数据库中添加一个版本字段, 如果消费过后就改掉, 做个匹配
- 如果这个消息时insert操作, 就做一个唯一键, 然后异常捕获一下
- 如果是set就不用解决


##### 死信队列的场景


##### RabbitMQ优势
解耦, 异步, 消峰

##### 高并发怎么处理
![[Pasted image 20220215193256.png]]
首先要提前预估用户的并发量
1. **微服务架构拆分**
2. (读请求)先用缓存, 把页面浏览的信息用minIO或者redis抗压,这些主要是用来读的一些属性, 可以数据库写一份, 缓存写一份, 大量请求直接先到缓存
3. 写请求 -> 用mq异步处理,这是为了处理消峰, 消峰一过, 再快速的把这个请求消化掉, 如果说还是消化不掉, 进行分库分表, 每个库承担压力
4. 这种情况下, 如果缓存失效, 缓存又会有定时的写缓存, 这时候要进行数据库读写分离主库, 缓存从从库里读数据
5. 如果是搜索请求, 一开始数据量很小, 可以自己基于lucene包封装搜索引擎, 如果数据量越来越大的话, 单机lucene撑不了这么搞得并发, 这时候要用es分布式集群, 因为es本身就是分布式, 加上他的存储海量数据的特性


##### reid分布式事务锁
首先redis是一种nosql, 是为了解决海量数据走数据库会很容易崩塌的这种问题, **Redis有5种主要数据类型**：string、hash、list(有序、可重复)、set(无序、不可重复)、zset(不可重复，基于score实现排序)
Redis之所以读取速度快是因为他基于内存操作,数据结构key-value相对简单,单线程操作不会有cpu上下文的切换,多路的I/O复用

Redis分布式锁的加锁, 本质上是加一个key,value，其实就是给Key键设置一个值（SET lock_key random_value NX PX 5000，NX表示键不存在时才设置），其他进程执行前会判断Redis中这个Key是否有值，如果发现这个Key有值了，就说明已有其他进程在执行，则循环等待，超时则获取失败
锁信息一定要设置过期时间, 不然万一要是redis挂了就会成死锁了

##### redis的持久化
**RDB:** Redis Database Backup file
也被叫做Redis数据快照。简单来说就是把内存中的所有数据都记录到磁盘中。当Redis实例故障重启后，从磁盘读取快照文件，恢复数据。快照文件称为RDB文件，默认是保存在当前运行目录。

**AOF:** Append Only File(追加文件)  -->  bgrewriteaof
Redis处理的每一个写命令都会记录在AOF文件，可以看做是命令日志文件。

##### 什么是缓存穿透、缓存雪崩、缓存击穿？
**缓存穿透**是指反复查询不存在的数据, 导致数据库压力过大, 这是恶意. 
**缓存雪崩**指某一个时间段内, 缓存集中的过期失效, 导致大量请求过来, 压力都集中到数据库
**缓存击穿**指将一个热点key的redis缓存失效, 导致大量请求瞬间集中到数据库