#####  MyBatis编程步骤是什么样的？

1. 创建SqlSessionFactory

2. 通过SqlSessionFactory创建SqlSession

3. 通过sqlsession执行数据库操作

4. 调用session.commit()提交事务

5. 调用session.close()关闭会话


**Dubbo三大核心能力**
1. 面向接口的远程方法调用(核心功能)
2. 只能容错和负载均衡
3. 服务自动注册和发现



**zookeeper概念**
Zookeeper 是 Apache Hadoop 项目下的一个子项目，是一个树形目录服务。
Zookeeper 是一个分布式的、开源的分布式应用程序的协调服务。
功能: 配置管理、分布式锁、集群管理

MQ
解耦, 消峰, 异步

##### 为什么用消息队列MQ, 消息队列的好处, 消息队列的坏处
案例: A系统要发送B/C/D系统, 如果这时候又有E系统也要让A发送数据, 如果D系统又不需要A调用, 这就导致A的上层系统有各种耦合
如果用了MQ, 再来其他系统, 直接定义MQ去消费就行了, 系统A就不再考虑再维护这个代码
这是MQ发布和订阅消息的模型, **Pub/Sub模型**



