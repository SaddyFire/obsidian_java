
![[Pasted image 20220105192257.png]]

**实现步骤:**
发布者/publisher/server:
- 定义 exchange\_交换机, routingKey\_路由, message_消息


订阅者/监听者/消费者/cumuser/listener
- 注解 CV
- 注入参数 例如: (String message)


## 1.publisher

```java
/**  
 * topicExchange 
 */
@Test  
public void testTopicExchange() {  
	 // 交换机名称  
	 String exchangeName = "itcast.topic";  
	 // 消息  
	 String message = "中国yyds!";  
	 String message2 = "小日本👇!";  
	 String message3 = "明天是个好天气!";  

	 // 发送消息  
	 for (int i = 1; i <= 3; i++) {  
		 rabbitTemplate.convertAndSend(exchangeName, "china.new", message);  
		 rabbitTemplate.convertAndSend(exchangeName, "japan.new", message2);  
		 rabbitTemplate.convertAndSend(exchangeName, "china.day", message3);  
	 }  
}
```

## 2.consumer
```java
/**  
 * topicExchange * @param msg  
 */  
@RabbitListener(bindings = @QueueBinding(  
 value = @Queue(name = "topic.queue1"), //定义队列名  
 exchange = @Exchange(name = "itcast.topic",type = ExchangeTypes.TOPIC), //定义交换机名字/类型(可省略)  
 key = "china.#" //定义derict  
))  
public void listenTopicQueue1(String msg){  
 System.err.println("topic.queue1收到消息__" +msg);  
}  
  
@RabbitListener(bindings = @QueueBinding(  
 value = @Queue(name = "topic.queue2"), //定义队列名  
 exchange = @Exchange(name = "itcast.topic",type = ExchangeTypes.TOPIC), //定义交换机名字/类型(可省略)  
 key = "#.new" //定义derict  
))  
public void listenTopicQueue2(String msg){  
 System.err.println("topic.queue2收到消息__" +msg);  
}
```
