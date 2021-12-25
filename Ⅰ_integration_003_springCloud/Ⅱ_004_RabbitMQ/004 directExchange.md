## 1.publisher
```java
/**  
 * directExchange */@Test  
public void testDirectExchange() {  
	// 交换机名称  
	String exchangeName = "itcast.direct";  
	// 消息  
	String message = "hello, red!";  
	String message2 = "hello, blue!";  
	String message3 = "hello, yellow!";  

	// 发送消息  
	for (int i = 1; i <= 3; i++) {  
		rabbitTemplate.convertAndSend(exchangeName, "red", message);  
		rabbitTemplate.convertAndSend(exchangeName, "blue", message2);  
		rabbitTemplate.convertAndSend(exchangeName, "yellow", message3);  
	}  
}
```

<br/>

## 2.consumer
*<u>注意注解</u>*
```java
/**  
 * directExchange * @param msg  
 */  
@RabbitListener(bindings = @QueueBinding(  
 value = @Queue(name = "direct.queue1"), //定义队列名  
 exchange = @Exchange(name = "itcast.direct"), //定义交换机名字/类型(可省略)  
 key = {"blue","yellow"} //定义derict  
))  
public void listenDirectQueue1(String msg){  
	 System.out.println("direct.queue1收到消息__" +msg);  
}  
  
@RabbitListener(bindings = @QueueBinding(  
 value = @Queue(name = "direct.queue2"), //定义队列名  
 exchange = @Exchange(name = "itcast.direct"), //定义交换机名字/类型(可省略)  
 key = {"red","yellow"} //定义derict  
))  
public void listenDirectQueue2(String msg){  
	 System.err.println("direct.queue2收到消息__" +msg);  
}
```
