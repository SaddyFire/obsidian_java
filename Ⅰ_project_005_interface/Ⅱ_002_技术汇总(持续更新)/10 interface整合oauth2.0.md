## 1. uaa模块
### 01 依赖版本
```xml
<!--父版本-->
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.3.2.RELEASE</version>
	<relativePath/>
</parent>
<spring-cloud.version>Hoxton.SR10</spring-cloud.version>
<spring-cloud-alibaba.version>2.2.5.RELEASE</spring-cloud-alibaba.version>

<!--uaa模块依赖-->
<dependencies>
	<!--核心依赖: -->
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-security</artifactId>
	</dependency>

	<!-- spring cloud alibaba 依赖 -->
	<dependency>
		<groupId>com.alibaba.cloud</groupId>
		<artifactId>spring-cloud-alibaba-dependencies</artifactId>
		<version>${spring-cloud-alibaba.version}</version>
		<type>pom</type>
		<scope>import</scope>
	</dependency>
	<!--nacos-discovery-->
	<dependency>
		<groupId>com.alibaba.cloud</groupId>
		<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
		<version>2.2.5.RELEASE</version>
	</dependency>
	<!--nacos配置中心依赖支持-->
	<dependency>
		<groupId>com.alibaba.cloud</groupId>
		<artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
		<version>2.2.5.RELEASE</version>
	</dependency>


	<!--核心依赖: oauth2支持-->
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-oauth2</artifactId>
		<version>2.1.0.RELEASE</version>
	</dependency>

	<!--spring security-->
	<dependency>
		<groupId>org.springframework.security.oauth.boot</groupId>
		<artifactId>spring-security-oauth2-autoconfigure</artifactId>
		<version>2.1.2.RELEASE</version>
		<exclusions>
			<exclusion>
				<groupId>org.springframework.security</groupId>
				<artifactId>spring-security-jwt</artifactId>
			</exclusion>
		</exclusions>
	</dependency>
	<!--核心依赖: jwt令牌-->
	<dependency>
		<groupId>org.springframework.security</groupId>
		<artifactId>spring-security-jwt</artifactId>
	</dependency>
	<dependency>
		<groupId>javax.interceptor</groupId>
		<artifactId>javax.interceptor-api</artifactId>
		<version>1.2</version>
	</dependency>
	<dependency>
		<groupId>com.alibaba</groupId>
		<artifactId>fastjson</artifactId>
	</dependency>
</dependencies>

<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<configuration>
				<mainClass>cn.saddyfire.security.distributed.uaa.UaaServerApplication</mainClass>
			</configuration>
			<executions>
				<execution>
					<goals>
						<goal>repackage</goal>
					</goals>
				</execution>
			</executions>
		</plugin>
		<plugin>
			<artifactId>maven-compiler-plugin</artifactId>
			<configuration>
				<source>1.8</source>
				<target>1.8</target>
			</configuration>
		</plugin>
	</plugins>
</build>

```

### 02 启动类
```java
import com.consmation.uaa.autoconfig.properties.UaaProperties;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;

/**
 * @author SaddyFire
 * @date 2022/4/7
 * @TIME:15:39
 */
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})   //此处要将数据源自动装配排除(依赖含有springboot-stater-jdbc)
@EnableAuthorizationServer  //打开OAuth认证服务的基础注解
@EnableConfigurationProperties({UaaProperties.class})
public class UaaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(UaaServerApplication.class,args);
        System.out.println("启动成功");
    }
}

```


### 03 bootstrap.yml
```yml
server:
  port: 8899
#  servlet:
#    context-path: /uaa   # 拦截路径
spring:
  profiles:
    active: dev
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
  application:
    name: uaa
  cloud:
    nacos:
      discovery:
        server-addr: ${NACOS_HOST:eca-register}:${NACOS_PORT:8848}
        namespace: ${NAMESPACE:interface}  # 命名空间
      config:
        server-addr: ${spring.cloud.nacos.discovery.server-addr}
        file-extension: ${FILE_EXTENSION:yml}
        group: ${GROUP:DEFAULT_GROUP}
        prefix: ${spring.application.name}
        namespace: ${NAMESPACE:interface}  # 命名空间
uaa:
  whiteList:
    - 127.0.0.1
    - localhost
```

### 04 config配置类
#### 4.1 MyAuthorizationConfig_基础认证授权功能的配置
`package com.consmation.uaa.config;`
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.ClientDetailsService;
import org.springframework.security.oauth2.provider.code.AuthorizationCodeServices;
import org.springframework.security.oauth2.provider.code.InMemoryAuthorizationCodeServices;
import org.springframework.security.oauth2.provider.token.AuthorizationServerTokenServices;
import org.springframework.security.oauth2.provider.token.DefaultTokenServices;
import org.springframework.security.oauth2.provider.token.TokenStore;
import org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter;

/**
 * @author SaddyFire
 * @date 2022/4/7
 * @TIME:17:14 基础认证授权功能的配置
 */
@Configuration
public class MyAuthorizationConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private UserDetailsService userDetailsService;

    @Autowired
    private TokenStore tokenStore;
    //解析JWT
    @Autowired
    private JwtAccessTokenConverter accessTokenConvert;

    //会通过ClientDetailsServiceConfigurer注入到Spring容器中
    @Autowired
    private ClientDetailsService clientDetailsService;

    //此处容器名不同, 是否注入?
    @Autowired
    private AuthenticationManager authenticationManager;


    @Autowired
    private AuthorizationCodeServices authorizationCodeServices;

    /**
     * 令牌端点的安全约束
     * @param security
     * @throws Exception
     */
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security
                .tokenKeyAccess("permitAll()") // oauth/token_key公开
                .checkTokenAccess("permitAll()") // oauth/check_token公开
                .allowFormAuthenticationForClients(); // 表单认证，申请令牌
    }

    /**
     * 配置客户端详细信息
     *
     * @param clients
     * @throws Exception
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients
                .inMemory()  //内存方式&jdbc/自定义客户端
                .withClient("c1")   //客户端标识
                .secret(new BCryptPasswordEncoder().encode("secret"))   //客户端安全码
                .resourceIds("interface")   //匹配资源id
                .authorizedGrantTypes(
//                        "authorization_code",   //授权码模式, 微信所采用的方式
                        "password", //密码模式, 也用于内部系统
                        "client_credentials", //客户端模式, 最不安全, 用于内部系统
//                        "implicit", //简化模式, 一般用于没有服务器的第三方
                        "refresh_token" //刷新令牌
                )    //该client允许的授权类型
                .scopes("all")  //限制客户端访问范围
                .autoApprove(false); //自动跳转
    }

    /**
     * 令牌访问端点配置
     * @param endpoints
     * @throws Exception
     */
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints
                //配置授权断点的URL(Endpoint URLS)
                .pathMapping("/oauth/confirm_access", "/customer/confirm_access")
                .authenticationManager(authenticationManager)//认证管理器
                //配置授权类型(Grant Types)
                .userDetailsService(userDetailsService)//密码模式的用户信息管理
                .authorizationCodeServices(authorizationCodeServices)//授权码服务
                .tokenServices(tokenService())//令牌管理服务
                .allowedTokenEndpointRequestMethods(HttpMethod.POST);
    }

    //注入AuthorizationServerTokenService  令牌管理服务服务策略
    public AuthorizationServerTokenServices tokenService() {
        DefaultTokenServices service = new DefaultTokenServices();
        service.setClientDetailsService(clientDetailsService); //客户端详情服务
        service.setSupportRefreshToken(true); //允许令牌自动刷新
        service.setTokenStore(tokenStore); //令牌存储策略-内存
        //使用JWT令牌
        service.setTokenEnhancer(accessTokenConvert);
        service.setAccessTokenValiditySeconds(7200); // 令牌默认有效期2小时
        service.setRefreshTokenValiditySeconds(259200); // 刷新令牌默认有效期3天
        return service;
    }

    //设置授权码模式的授权码如何存取。
    @Bean
    public AuthorizationCodeServices authorizationCodeServices() {
        return new InMemoryAuthorizationCodeServices();
        //JdbcAuthorizationCodeServices
    }

}

```


#### 4.2 MyWebSecurityConfig_安全策略配置
`package com.consmation.uaa.config;`
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;

/**
 * @author SaddyFire
 * @date 2022/4/7
 * @TIME:17:26
 * 安全策略配置
 */
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true)
public class MyWebSecurityConfig extends WebSecurityConfigurerAdapter {



    /**
     *用户安全拦截策略
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {

        http.csrf().disable()//关闭csrf跨域检查
                .authorizeRequests()
//                .antMatchers(GET, "/oauth/check_token")
//                .access("isFullyAuthenticated() and (hasIpAddress('IP_OR_NETWORK_OF_RESOURCE_SERVER') or hasIpAddress('127.0.0.1/32'))")
                .antMatchers("/**")  //配置动态路径
                .hasIpAddress("192.168.141.1")
                .anyRequest()
                .authenticated() //其他请求需要登录
                .and() //并行条件
                .formLogin(); //允许表单登录

    }


    @Bean
    public PasswordEncoder passwordEncoder() {
        // return NoOpPasswordEncoder.getInstance();
        return new BCryptPasswordEncoder();
    }

    //从父类加载认证管理器
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Bean
    public UserDetailsService userDetailsService(){
        InMemoryUserDetailsManager userDetailsManager = new
                InMemoryUserDetailsManager(User.withUsername("admin").password(passwordEncoder().encode("admin")).authorities("interface").build()
                ,User.withUsername("admin1").password(passwordEncoder().encode("admin1")).authorities("interface").build()
                ,User.withUsername("bbb").password(passwordEncoder().encode("bbb")).authorities("bbb").build());
        return userDetailsManager;
    }


}

```

#### 4.3 TokenConfig_令牌配置
`package com.consmation.uaa.config;`
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.provider.token.TokenStore;
import org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter;
import org.springframework.security.oauth2.provider.token.store.JwtTokenStore;

/**
 * @author SaddyFire
 * @date 2022/4/7
 * @TIME:17:37
 * 令牌配置
 */
@Configuration
public class TokenConfig {

    private static final String SIGN_KEY="uaa";

    // 使用JWT令牌。
    @Bean
    public TokenStore tokenStore(){
        return new JwtTokenStore(accessTokenConvert());
    }
    @Bean
    public JwtAccessTokenConverter accessTokenConvert(){
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey(SIGN_KEY);
        return converter;
    }

}

```
#### 4.4 WhiteListConfig_白名单配置类
`package com.consmation.uaa.config;`
```java
import com.consmation.uaa.filter.MyAuthValidationFilter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;

import javax.servlet.DispatcherType;

/**
 * @author SaddyFire
 * @date 2022/4/9
 * @TIME:22:59
 */
@Configuration
public class WhiteListConfig {

    @Autowired
    private MyAuthValidationFilter myAuthValidationFilter;

    @Bean
    public FilterRegistrationBean registFilter() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(myAuthValidationFilter);
        registration.addUrlPatterns("/*");
        registration.setName("WhiteListConfig");
        registration.setOrder(Ordered.HIGHEST_PRECEDENCE);
        registration.setDispatcherTypes(DispatcherType.REQUEST);
        return registration;
    }
}

```

### 05 
