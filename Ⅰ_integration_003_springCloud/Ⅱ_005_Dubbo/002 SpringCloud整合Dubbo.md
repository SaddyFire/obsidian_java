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
