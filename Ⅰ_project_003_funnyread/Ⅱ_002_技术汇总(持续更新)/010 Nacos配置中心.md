### 01 依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

### 02 bootstrap.yml配置
`bootstrap.yml`
```yml
server:
  port: 8888
spring:
  profiles:
    active: prod
  application:
    name: tanhua-gateway
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.136.160:8848
      config:
        server-addr: 192.168.136.160:8848
        file-extension: yml
```

![[Pasted image 20220103143618.png]]

### 03 nacos添加配置

##### 在nacos中添加网关配置

##### 添加配置内容	

##### 配置如下内容

```yml
server:
  port: 8888
spring:
  cloud:
    gateway:
      globalcors:
        add-to-simple-url-handler-mapping: true
        corsConfigurations:
          '[/**]':
            allowedHeaders: "*"
            allowedOrigins: "*"
            allowedMethods:
              - GET
              - POST
              - DELETE
              - PUT
              - OPTION
      routes:
        # 用户微服务
        - id: tanhua-app-server
          uri: lb://tanhua-app-server
          predicates:
            - Path=/app/**
          filters:
            - StripPrefix= 1
        # 文章微服务
        - id: tanhua-admin
          uri: lb://tanhua-admin
          predicates:
            - Path=/admin/**
          filters:
            - StripPrefix= 1
gateway:
  excludedUrls: /user/login,/user/loginVerification,/system/users/verification,/system/users/login
```






