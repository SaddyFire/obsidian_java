## 1.依赖(父工程)
```xml
<!--AMQP依赖，包含RabbitMQ-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

## 2.publisher
*<u>application.yml配置类</u>*
```yml
logging:
  pattern:
    dateformat: MM-dd HH:mm:ss:SSS
spring:
  rabbitmq:
    host: 139.224.230.186 # 主机名
    port: 5672 # 端口
    virtual-host: / # 虚拟主机
    username: root # 用户名
    password: root # 密码
```

*<u>测试类</u>*
```java
@Autowired  
private RabbitTemplate rabbitTemplate;

/**
 * workQueue
 * 向队列中不停发送消息，模拟消息堆积。
 */
@Test
public void testWorkQueue() throws InterruptedException {
	// 队列名称
	String queueName = "simple.queue";
	// 消息
	String message = "hello, message_";
	for (int i = 1; i <= 50; i++) {
		// 发送消息
		rabbitTemplate.convertAndSend(queueName, message + i);
		Thread.sleep(20);
	}
}
```

## 3.consumer
*<u>模拟两个消费者接收</u>*
```java
@RabbitListener(queues = "simple.queue")
public void listenSimpleQueue1(String msg) throws InterruptedException {
	System.out.println("消费者1接收到消息 -> " +msg);
	Thread.sleep(20);
}

@RabbitListener(queues = "simple.queue")
public void listenSimpleQueue2(String msg) throws InterruptedException {
	System.err.println("消费者2接收到消息 -> " +msg);
	Thread.sleep(200);
}
```
