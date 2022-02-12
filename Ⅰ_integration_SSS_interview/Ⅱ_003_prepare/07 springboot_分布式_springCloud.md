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
