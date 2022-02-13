##### SringBoot的核心注解是哪个？它主要由哪几个注解组成？
``@SpringBootApplication`
该注解包含 
`@SpringBootConfiguration`
`@EnableAutoConfiguration`
`@ComponentScan`

##### SpringBoot的工作原理/自动配置/SPI机制是怎么样子的？
`@SpringBootApplication`  -> `@EnableAutoConfiguration` -> `@import` 中导入的类会被加载到spring IOC容器中 ->  `SpringFactoriesLoader` 反射出`META-INFO`下的`spring.factories`文件


#####   什么是CAP定理？
Consistency(一致性)
Availability(可用性)
Partition tolerance(分区容错性)

##### BASE理论
Basically Available(基本可用, 必要时部分可用, 成全一致性),
Soft state(软状态)
Eventually consistent(最终一致性)

##### 什么是面向服务架构—SOA？
Service-Oriented Architecture(面向服务架构),将应用程序的不同功能单元进行拆分, 定义接口和契约(contract)连接彼此

#####  SpringCloud和Dubbo对比
1. 首先SpringCloud与Dubbo都是实现微服务有效的工具
2. Dubbo只是实现了服务治理，而SpringCloud子项目分别覆盖了微服务架构下的众多部件
3. Dubbo使用RPC通讯协议（传输层，基于TPC协议) ，SpirngCloud使用RESTful完成通信（应用层，基于HTTP协议)，Dubbo效率略高于SpringCloud

##### SpringCloud常用组件
Eureka
Ribbon Fiegn
Hystrix
Gateway
Spring Cloud Config

##### SpringCloud ALibaba常用组件
Sentinel
Nacos
RocketMQ(我们使用RabbitMQ)
Dubbo
Seata 分布式事务


##### seata


**C**	Consistency
**A**	Availability
**P**	Partition Tolerance

**BA**	Basically Available
**S**	 Soft State
**E**	 Eventually Consistent

**TC**		Transaction Coordinator
事务协调器，维护全局事务的运行状态，负责协调并驱动全局事务的提交或回滚。

**TM**		Transaction Manager
控制全局事务的边界，负责开启一个全局事务，并最终发起全局提交或全局回滚的决议。

**RM**		Resource Manager
控制分支事务，负责分支注册、状态汇报，并接收事务协调器的指令，驱动分支（本地）事务的提交和回滚。
##### Seata的实现原理
Seata将一个本地事务做为一个分布式事务分支，所以若干个分布在不同微服务中的本地事务共同组成了一个全局事务，结构如下。
![[Pasted image 20220212205758.png]]
TM先向TC申请开启一个全局事务，TM同意后会给分布式事务开启全局唯一事务ID，每个成员事务会找TC注册自己的事务分支，TC会将唯一ID给每一个分支事务，所有分支收到ID后，TM会向TC发起全局提交或全局回滚的决议，TC就会向所有RM发送最终提交还是回滚的指令（undo log实现），所有RM接受指令执行。


**XA**   -->  X/Open DTP, Distributed Transaction Processing
**AT**	 -->  undo-log
**TCC** -->  Try Confirm Cancel
空回滚、业务悬挂

