**核心功能** ^891ba3
1. 路由转发
2. 负载均衡
3. 统一鉴权
4. 网关映射图 [[009 GateWay网关#03 网关映射图|网关映射图]]


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

## 02 配置鉴权管理器
##### 鉴权过滤器

`tanhua-gateway.com.tanhua.gateway.filter.AuthFilter`

```java
@Component
public class AuthFilter implements GlobalFilter, Ordered {

    @Value("${gateway.excludedUrls}")
    private List<String> excludedUrls; //配置不校验的连接

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        //获取当前请求连接
        String url = exchange.getRequest().getURI().getPath();
        System.out.println( "url:"+ url);
        //放行不需要校验的接口
        if(excludedUrls.contains(url)){
            return chain.filter(exchange);
        }
        //获取请求头中的token
        String token = exchange.getRequest().getHeaders().getFirst("Authorization");
        //后台系统页面发送的token以"Bearer "开头，需要处理
        if(!StringUtils.isEmpty(token)){
            token = token.replace("Bearer ", "");
        }
        ServerHttpResponse response = exchange.getResponse();

        //使用工具类，判断token是否有效
        boolean verifyToken = JwtUtils.verifyToken(token);
        //如果token失效，返回状态码401，拦截
        if(!verifyToken) {
            Map<String, Object> responseData = new HashMap<>();
            responseData.put("errCode", 401);
            responseData.put("errMessage", "用户未登录");
            return responseError(response,responseData);
        }
        return chain.filter(exchange);
    }

    //响应错误数据
    private Mono<Void> responseError(ServerHttpResponse response,Map<String, Object> responseData){
        // 将信息转换为 JSON
        ObjectMapper objectMapper = new ObjectMapper();
        byte[] data = new byte[0];
        try {
            data = objectMapper.writeValueAsBytes(responseData);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
        // 输出错误信息到页面
        DataBuffer buffer = response.bufferFactory().wrap(data);
        response.setStatusCode(HttpStatus.UNAUTHORIZED);
        response.getHeaders().add("Content-Type", "application/json;charset=UTF-8");
        return response.writeWith(Mono.just(buffer));
    }

    /**
     * 设置过滤器的执行顺序
     */
    @Override
    public int getOrder() {
        return Ordered.LOWEST_PRECEDENCE;
    }
}
```


## 03 网关映射图
![[Pasted image 20220105100852.png]]

![[Pasted image 20220121211341.png]]

[[#^891ba3|返回开头]]




