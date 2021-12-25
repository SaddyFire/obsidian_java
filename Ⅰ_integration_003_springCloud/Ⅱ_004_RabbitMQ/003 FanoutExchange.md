## 1.publisher
```java
/**  
 * fanoutExchange */@Test  
public void testFanoutExchange() {  
	// 队列名称  
	String exchangeName = "itcast.fanout";  
	// 消息  
	String message = "hello, everyone!";  
	rabbitTemplate.convertAndSend(exchangeName, "", message);  
}
```

## 2.FanoutConfig类
```java
import org.springframework.amqp.core.Binding;  
import org.springframework.amqp.core.BindingBuilder;  
import org.springframework.amqp.core.FanoutExchange;  
import org.springframework.amqp.core.Queue;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
  
/**  
 * @author SaddyFire  
 * @date 2021/12/18  
 * @TIME:14:47  
 */  
@Configuration  
public class FanoutConfig {  
 /**  
 * 声明交换机  
 * @return Fanout类型交换机  
 */  
 @Bean  
 public FanoutExchange fanoutExchange(){  
	 return new FanoutExchange("itcast.fanout");  
 }  
  
 /**  
 * 第1个队列  
 */  
 @Bean  
 public Queue fanoutQueue1(){  
	 return new Queue("fanout.queue1");  
 }  
  
 /**  
 * 绑定队列和交换机  
 */  
 @Bean  
 public Binding bindingQueue1(Queue fanoutQueue1, FanoutExchange fanoutExchange){  
	 return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);  
 }  
  
 /**  
 * 第2个队列  
 */  
 @Bean  
 public Queue fanoutQueue2(){  
	 return new Queue("fanout.queue2");  
 }  
  
/**  
* 绑定队列和交换机  
*/  
@Bean  
public Binding bindingQueue2(Queue fanoutQueue2, FanoutExchange fanoutExchange){  
		return BindingBuilder
		.bind(fanoutQueue2)
		.to(fanoutExchange);  
}  
  

```

## 3.consumer

```java
@RabbitListener(queues = "fanout.queue1")  
public void listenFanoutQueue1(String msg) {  
	 System.out.println("消费者1接收到Fanout消息：【" + msg + "】");  
}  
  
@RabbitListener(queues = "fanout.queue2")  
public void listenFanoutQueue2(String msg) {  
	 System.out.println("消费者2接收到Fanout消息：【" + msg + "】");  
}
```
