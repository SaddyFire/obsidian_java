**核心功能**
1. 路由转发
2. 负载均衡
3. 统一鉴权


## 01 搭建网关

##### 依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>

    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>

    <!-- 监控检查-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <!-- nacos配置中心依赖支持
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>
    -->
    <dependency>
        <groupId>com.itheima</groupId>
        <artifactId>tanhua-commons</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

##### 引导类
`tanhua-gateway`
```java
@SpringBootApplication
public class GatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

##### 跨域问题配置类
`com.itheima.gateway.config`
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.reactive.CorsWebFilter;
import org.springframework.web.cors.reactive.UrlBasedCorsConfigurationSource;
import org.springframework.web.util.pattern.PathPatternParser;

/**
 * 跨域支持
 */
@Configuration
public class CorsConfig {

    @Bean
    public CorsWebFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
        config.addAllowedMethod("*");
        config.addAllowedOrigin("*");
        config.addAllowedHeader("*");
        UrlBasedCorsConfigurationSource source =
                new UrlBasedCorsConfigurationSource(new PathPatternParser());
        source.registerCorsConfiguration("/**", config);
        return new CorsWebFilter(source);
    }
}
```

##### 配置文件

```yml
server:
  port: 8888
spring:
  application:
    name: tanhua-gateway
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.136.160:8848
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
      #配置路由
      #探花系统 tanhua-app-server
      #后台系统 tanhua-admin
      routes:
        # 手机端访问 , 随机即可, 通常未服务名
        - id: tanhua-app-server
          #负载均衡, 服务名称
          uri: lb://tanhua-app-server
          #路由转发:
          # http://127.0.0.1:8888/app/login/in --> http://tanhua-app-server(服务名称)/login/in
          predicates:
            - Path=/app/**
          filters:
            #表示只去一个斜杠,会将请求中的/app删掉
            - StripPrefix= 1
        # 管理后台
        - id: tanhua-admin
          uri: lb://tanhua-admin
          predicates:
            - Path=/admin/**
          filters:
            - StripPrefix= 1
#自定义配置，定义不需要校验token的连接
gateway:
  excludedUrls: /user/login,/user/loginVerification,/system/users/verification,/system/users/login
```





