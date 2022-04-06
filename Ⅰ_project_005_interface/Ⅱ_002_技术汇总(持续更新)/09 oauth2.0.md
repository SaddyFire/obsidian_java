## 1. 第一部分
#### 01 四种认证方式
1. 授权码模式
![[image-20220403135812493.png]]
2. 简化模式
![[image-20220403140124326.png]]
3. 密码模式
![[image-20220403140221252.png]]
4. 客户端模式
	类似微服务调用场景
![[image-20220403140320283.png]]

在梳理OAuth2.0协议流程的过程中, 其中有一个主线, 就是三方参与者之间的信任度

普通令牌只是一个随机的字符串, 没有特殊的意义. 这就意味着, 当客户带上令牌去访问应用的接口时, 应用本身无法判断这个令牌是否正确, 需要到授权服务器上判断令牌是否有效

高并发情况下授权服务器会成为性能瓶颈

改良的方式就是JWT令牌, 将令牌对应的相关信息全部冗余到令牌本身, 这样资源服务器就不再需要发送请求给授权服务器去检查令牌了, 他自己就可以读取到令牌的授权信息. 
JWT本质是一个加密的字符串

#### 02 JWT令牌

## 2. 第二部分
#### 01 各种授权平台


#### 02 手写开放授权平台
#### 03 通过开放授权平台理解OAuth2.0协议
#### 04 SpringSecurity提供的OHuth2.0实现
#### 联合登录和单点登录


## 3. SpringSecurity搭建
### 01 依赖
```xml
<!--springboot 和 security 的整合-->
<!--此处省略父依赖-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 02 启动类注解
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;

/**
 * @author SaddyFire
 * @date 2022/4/4
 * @TIME:23:25
 */
@SpringBootApplication
@EnableWebSecurity  //开启security
public class SpringBootSecurityApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootSecurityApplication.class, args);
    }
}
```
启动后, 密码在控制台
localhost:xxxx/login 为默认登录地址
localhost:xxxx/logout 为默认登出地址


### 03 拓展点
1. 主体数据来源
2. 密码解析器
3. 自定义授权及安全拦截策略
4. 拦截策略