# 一、过滤工厂
<br/>

## 1 gateway更改配置文件,如下:
```yml
server:
  port: 10010
logging:
  level:
    cn.itcast: debug
  pattern:
    dateformat: MM-dd HH:mm:ss:SSS
spring:
  application:
    name: gateway
  cloud:
    nacos:
      server-addr: 127.0.0.1:8848 # nacos地址
    gateway:
      routes:
        - id: user-service # 路由标示，必须唯一
          uri: lb://userservice # 路由的目标地址
          predicates: # 路由断言，判断请求是否符合规则
            - Path=/user/** # 路径断言，判断路径是否是以/user开头，如果是则符合
          filters: # 过滤器
            - AddRequestHeader=Truth, is freaking awesome! # 添加请求头
            - AddRequestParameter=username,canglaoshi # 添加请求参数
        - id: order-service
          uri: lb://orderservice
          predicates:
            - Path=/order/**
```

## 2 userservice的controller类添加方法
```java
    @GetMapping("/one/{id}")
    public User queryById2(@PathVariable("id") Long id, String username, @RequestHeader("Truth") String truth) {
        User user = userService.queryById(id);
        user.setUsername(username + ": " + truth);
        return user;
    }
```

## 3 默认过滤器
```yml
spring:
  cloud:
    gateway:
      routes:
      - id: user-service 
        uri: lb://userservice 
        predicates: 
        - Path=/user/**
      - id: user-service 
        uri: lb://userservice 
        predicates: 
        - Path=/user/**
      default-filters: # 默认过滤项
      - AddRequestHeader=Truth, Itcast is freaking awesome! 
```
## 4 过滤器总结

① 对路由的请求或响应做加工处理，比如添加请求头

② 配置在路由下的过滤器只对当前路由的请求生效

defaultFilters的作用是什么？

- ① 对所有路由都生效的过滤器

<br/>

# 二、gateway入门
<br/>

## 1 创建gateway服务 引入依赖

```xml
<!--网关-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!--nacos服务发现依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

## 2 启动类
```java
package cn.itcast.gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class GatewayApplication {

	public static void main(String[] args) {
		SpringApplication.run(GatewayApplication.class, args);
	}
}
```
## 3 application.yml配置文件
```yml
server:
  port: 10010 # 网关端口
spring:
  application:
    name: gateway # 服务名称
  cloud:
    nacos:
      server-addr: localhost:8848 # nacos地址
    gateway:
      routes: # 网关路由配置
        - id: user-service # 路由id，自定义，只要唯一即可
          # uri: http://127.0.0.1:8081 # 路由的目标地址 http就是固定地址
          uri: lb://userservice # 路由的目标地址 lb就是负载均衡，后面跟服务名称
          predicates: # 路由断言，也就是判断请求是否符合路由规则的条件
            - Path=/user/** # 这个是按照路径匹配，只要以/user/开头就符合要求
```
 **注意缩进**

## 04 重启测试
打开网关,访问http://localhost:10010/user/1

若查询到结果即转发成功

## 05 网关搭建步骤总结
1. 创建项目，引入nacos服务发现和gateway依赖
2. 配置application.yml，包括服务基本信息、nacos地址、路由

路由配置包括：

1. **路由id**：路由的唯一标示
2. **路由目标（uri）**：路由的目标地址，http代表固定地址，lb代表根据服务名负载均衡
3. **路由断言（predicates）**：判断路由的规则，
4. **路由过滤器（filters）**：对请求或响应做处理

</br>

# 三、GlobalFilter
<br/>

## 01 在gateway中定义一个过滤器
- <font color=#ff4500 size=4>1. 参数中是否有authorization</font>

- <font color=#ff4500 size=4>2. authorization参数值是否为admin</font>

```java
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.annotation.Order;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.util.MultiValueMap;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Order(-1)
@Component
public class AuthorizeFilter implements GlobalFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 1.获取请求参数
        MultiValueMap<String, String> params = exchange.getRequest().getQueryParams();
        // 2.获取authorization参数
        String auth = params.getFirst("authorization");
        System.out.println(auth);
        // 3.校验
        if ("admin".equals(auth)) {
            // 放行
            return chain.filter(exchange);
        }
        // 4.拦截
        // 4.1.禁止访问，设置状态码
        exchange.getResponse().setStatusCode(HttpStatus.FORBIDDEN);
        // 4.2.结束处理
        return exchange.getResponse().setComplete();
    }
}
```

## 02 访问网址
http://localhost:10010/user/one/1?**authorization=admin**

**(请求参数应加访问身份)**























