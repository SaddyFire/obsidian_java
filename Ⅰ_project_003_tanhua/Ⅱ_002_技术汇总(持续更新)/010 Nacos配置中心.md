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







