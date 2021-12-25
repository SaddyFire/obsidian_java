# 一、基础架构
</br>

1. **Provider** -> 暴露服务的服务提供方。
2. **Consumer** -> 调用远程服务的服务消费方。
3. **Registry** -> 服务注册与发现的注册中心。
4. **Monitor** -> 统计服务的调用次数和调用时间的监控中心。

<u>*架构图*</u>

![[Pasted image 20211220161026.png]]

<u>*Dubbo调用原理*</u>

![[Pasted image 20211220162515.png]]

## 1 user-provider
### 1.1 依赖
```xml
<!--dubbo的起步依赖-->  
<dependency>  
	 <groupId>org.apache.dubbo</groupId>  
	 <artifactId>dubbo-spring-boot-starter</artifactId>  
	 <version>2.7.8</version>  
</dependency>  
  
<dependency>  
	 <groupId>org.apache.dubbo</groupId>  
	 <artifactId>dubbo-registry-nacos</artifactId>  
	 <version>2.7.8</version>  
</dependency>  
  <!--user-api-->  
<dependency>  
	 <groupId>cn.itcast</groupId>  
	 <artifactId>user-api</artifactId>  
	 <version>1.0-SNAPSHOT</version>  
</dependency>
```

### 1.2 application.yml

```yml
server:  
  port: 18081  
spring:  
  datasource:  
    url: jdbc:mysql://localhost:3306/dubbo-demo?useSSL=false  
    username: root  
    password: 2323  
    driver-class-name: com.mysql.jdbc.Driver  
 application:  
    name: user-provider  
logging:  
  level:  
    cn.itcast: debug  
 pattern:  
    dateformat: HH:mm:ss:SSS  
#配置dubbo提供者  
#dubbo协议和访问端口  
dubbo:  
  protocol:  
    name: dubbo  
    port: 20881  
 #注册中心的地址  
 registry:  
    address: nacos://127.0.0.1:8848  
  #dubbo注解的包扫描  
 scan:  
    base-packages: cn.itcast.user.service
```

### 1.3 UserServiceImpl	
**@DubboService注解**

```java
import cn.itcast.user.domain.User;
import cn.itcast.user.mapper.UserMapper;
import cn.itcast.user.service.UserService;
import org.apache.dubbo.config.annotation.DubboService;
import org.springframework.beans.factory.annotation.Autowired;
/**
 * @author SaddyFire
 * @date 2021/12/20
 * @TIME:11:01
 */


@DubboService
public class UserServiceImpl implements UserService {

    @Autowired
    private UserMapper userMapper;

    //根据id查询用户名称
    public String queryUsername(Long id) {
        return userMapper.findById(id).getUsername();
    }

    @Override
    public User queryById(Long id) {
        return userMapper.findById(id);
    }
}
```

</br>

## 2 user-consumer

### 2.1 依赖同上
### 2.2 application.yml

```yml
server:
  port: 18080
spring:
  application:
    name: user-consumer
logging:
  level:
    cn.itcast: debug
  pattern:
    dateformat: HH:mm:ss:SSS
#配置dubbo服务消费者
dubbo:
  registry:
    address: nacos://127.0.0.1:8848
```

### 2.3 UserController
**@DubboReference**

```java
import cn.itcast.user.domain.User;
import cn.itcast.user.service.UserService;
import lombok.extern.slf4j.Slf4j;
import org.apache.dubbo.config.annotation.DubboReference;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Slf4j
@RestController
@RequestMapping("/user")
public class UserController {

    //引用远程服务
    @DubboReference
    private UserService userService;

//    @GetMapping("/username/1")
//    public String findUserName(@PathVariable("id") Long id) {
//        return userService.queryUsername(id);
//    }

    @GetMapping("/username/{id}")
    public User queryById(@PathVariable("id") Long id) {
        return userService.queryById(id);
    }
}
```

## 3 user-api
将domain和service接口抽取出来

</bd>

# 二、SpringCloud整合Dubbo

<bd/>
</bd>

## 1 依赖
**父工程**

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.2.5.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

**consumer & provider**

```xml
<!--nacos注册中心的依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

<!--springcloud alibaba dubbo依赖   -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-dubbo</artifactId>
</dependency>

<dependency>
    <groupId>cn.itcast</groupId>
    <artifactId>dubbo-api</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

## 2 application.yml

**provider** -> user-service

```yml
server:  
  port: 8081  
spring:  
  datasource:  
    url: jdbc:mysql://localhost:3306/dubbo-demo?useSSL=false  
    username: root  
    password: 2323  
    driver-class-name: com.mysql.jdbc.Driver  
 application:  
    name: user-service  
  cloud:  
    nacos:  
      discovery:  
        server-addr: localhost:8848  
#配置dubbo，注册中心，暴露的端口和协议，dubbo注解的包扫描  
dubbo:  
  protocol:  
    name: dubbo  
    port: 20881  
 registry:  
    address: spring-cloud://localhost   #使用SpringCloud中的注册中心  
 scan:  
    base-packages: cn.itcast.user.service  #dubbo中包扫描
```

**consumer** -> order-service

```yml
server:  
  port: 8088  
spring:  
  datasource:  
    url: jdbc:mysql://localhost:3306/dubbo-demo?useSSL=false  
    username: root  
    password: 2323  
    driver-class-name: com.mysql.jdbc.Driver  
 application:  
    name: order-service  
  cloud:  
    nacos:  
      discovery:  
        server-addr: localhost:8848  
#dubbo配置  
dubbo:  
  registry:  
    address: spring-cloud://localhost  #使用cloud的注册中心  
 consumer:  
    check: false #dubbo默认有启动检查  
 retries: 0 #dubbo内置的重试机制
 
```


## 3 业务代码略







