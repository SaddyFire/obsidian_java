# 一、springcloud简介

<br/>



</br>

# 二、Eureka注册中心

<br/>
## 1 搭建eureka-server

1.  创建模块
2.  依赖

```xml
<dependencies>
	<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
	</dependency>
</dependencies>
```
4. 配置文件
```xml
server:
  port: 10086
spring:
  application:
    name: eureka-server
eureka:
  client:
    service-url: 
      defaultZone: http://127.0.0.1:10086/eureka
```
5. 编写启动类
```java
package cn.itcast.eureka;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```
## 2 服务注册
1. 依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```
2. 配置文件(注意缩进)

```yml
spring:
  application:
    name: userservice
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
```
## 3 服务发现
1. 依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```
2. 配置文件
```yml
spring:
  application:
    name: orderservice
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
```
3. 负载平衡

在RestTemplateConfig类添加<font color=#ffa500>@LoadBalanced</font>注解
```java
@Configuration
public class RestTemplateConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }

}
```

</br>

# 三、Ribbon负载均衡

<br/>

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

</br>

# 四、Nacos注册中心

</br>

## 1 Nacos的安装及启动
```cmd
startup.cmd -m standalone
```
## 2 引入依赖
#### 2.1父工程依赖
```xml
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-alibaba-dependencies</artifactId>
	<version>2.2.6.RELEASE</version>
	<type>pom</type>
	<scope>import</scope>
</dependency>
```
#### 2.2子工程依赖
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```
## 3 配置nacos地址
```
spring:
  cloud:
    nacos:
      server-addr: 127.0.0.1:8848
```

</br>

# 五、Nacos与Eureka的区别

</br>

 **1.** Nacos支持服务端主动检测提供者状态：临时实例采用心跳模式，非临时实例采用主动检测模式

 **2.** 临时实例心跳不正常会被剔除，非临时实例则不会被剔除

 **3.**  Nacos支持服务列表变更的消息推送模式，服务列表更新更及时

 **4.** Nacos集群默认采用AP方式，当集群中存在非临时实例时，采用CP模式；Eureka采用AP方式





