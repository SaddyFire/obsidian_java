## 1 微服务技术对比

ps: 微服务是一种经过良好架构设计的分布式架构方案

![2021121901.png](/_resources/2021121901.png)

![2021121902.png](/_resources/2021121902.png)

## 2 springcloud简介

官网: https://spring.io/projects/spring-cloud

- springcloud与spring boot版本对应关系
- ![2021121903.png](/_resources/2021121903.png)

## 3 初步实现远程调用

实现步骤:


- 注册一个RestTemplate的实例到Spring容器
    
- 修改order-service服务中的OrderService类中的queryOrderById方法，根据Order对象中的userId查询User
    
- 将查询的User填充到Order对象，一起返回



### 3.1 注册容器

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

### 3.2 远程调用

```java
@Service
public class OrderService {

    @Autowired
    private OrderMapper orderMapper;

    @Autowired
    private RestTemplate restTemplate;
    public Order queryOrderById(Long orderId) {
        // 1.查询订单
        Order order = orderMapper.findById(orderId);

        String url = "http://userservice/user/" + order.getUserId();
        User user = restTemplate.getForObject(url, User.class);
        //设置用户
        order.setUser(user);
        // 4.返回
        return order;
    }
}
```
