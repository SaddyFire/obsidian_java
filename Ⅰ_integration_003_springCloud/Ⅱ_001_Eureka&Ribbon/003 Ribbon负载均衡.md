## 1 负载均衡原理



SpringCloud底层其实是利用了一个名为Ribbon的组件，来实现负载均衡功能的。 

![[2021122001.png]]


#### 1.1 LoadBalancerIntercepor

![[2021122002.png]]


可以看到这里的intercept方法，拦截了用户的HttpRequest请求，然后做了几件事：

- request.getURI()：获取请求uri，本例中就是 http://user-service/user/8
- originalUri.getHost()：获取uri路径的主机名，其实就是服务id，user-service
- this.loadBalancer.execute()：处理服务id，和用户请求。

这里的this.loadBalancer是LoadBalancerClient类型，我们继续跟入。

#### 1.2 LoadBalancerClient

![[2021122003.png]]

- getLoadBalancer(serviceId)：根据服务id获取ILoadBalancer，而ILoadBalancer会拿着服务id去eureka中获取服务列表并保存起来。
- getServer(loadBalancer)：利用内置的负载均衡算法，从服务列表中选择一个。本例中，可以看到获取了8082端口的服务 !

![[2021122004 1.png]]

## 2 负载均衡策略IRule

![[2021122005.png]]

![[187807f501b340ef8498d7de16f911c5.png]]

## 3 自定义负载策略

#### 3.1 RuleConfig类

```java
@Configuration
public class RuleConfig {

    @Bean
    public IRule randomRule() {
        return new RandomRule();
    }
}
```

#### 3.2 启动项

```java
@MapperScan("cn.itcast.order.mapper")
@SpringBootApplication
@RibbonClient(name = "userservice", configuration = RuleConfig.class)
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }

}
```

#### 3.3 服务配置文件

```yml
userservice: # 给某个微服务配置负载均衡规则，这里是userservice服务
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule # 负载均衡规则 
```
