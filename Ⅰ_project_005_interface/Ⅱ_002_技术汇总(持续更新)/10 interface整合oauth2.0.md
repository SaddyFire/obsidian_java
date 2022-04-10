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

### 05 properties动态配置类
#### 5.1 UaaProperties_定义uaa动态配置类
`package com.consmation.uaa.autoconfig.properties;`
```java
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;

import java.util.List;

/**
 * @author SaddyFire
 * @date 2022/3/28
 * @TIME:17:58
 */
@Data
@ConfigurationProperties(prefix="uaa")
public class UaaProperties {
    //白名单列表
    private List<String> whiteList;

}

```

### 06 filter过滤器类
#### 6.1 MyAuthValidationFilter_白名单过滤器
`package com.consmation.uaa.filter;`
```java
import com.consmation.uaa.autoconfig.properties.UaaProperties;
import com.consmation.uaa.utils.Constant;
import com.consmation.uaa.utils.ParameterRequestWrapper;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.*;
import java.net.URL;
import java.util.HashMap;
import java.util.List;


/**
 * @author SaddyFire
 * @date 2022/4/9
 * @TIME:23:02
 * AbstractAuthenticationProcessingFilter
 * ClientCredentialsTokenEndpointFilter client_credentials模式下的过滤器  attemptAuthentication
 * UsernamePasswordAuthenticationFilter password模式下的过滤器
 * TokenEndpoint 请求token路径
 */
@Component
public class MyAuthValidationFilter implements Filter {

    @Autowired
    private UaaProperties uaaProperties;


    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        HttpServletResponse httpServletResponse = (HttpServletResponse) response;

        StringBuffer requestURL = httpServletRequest.getRequestURL();

        URL url = new URL(requestURL.toString());
        String host = url.getHost();
        List<String> whiteList = uaaProperties.getWhiteList();

        if (!whiteList.contains(host)) {
            filterChain.doFilter(request, response);
            return;
        }
        //获取所有参数集合
        HashMap<String,String[]> parameterMap = new HashMap<>();

        String clientId = request.getParameter("client_id");
        String clientSecret = request.getParameter("client_secret");
        String grantType = request.getParameter("grant_type");

        //若核心参数未缺失, 则放行
        if (StringUtils.isBlank(clientId)) {
            parameterMap.put("client_id", new String[]{Constant.CLIENT_ID});
        }
        if (StringUtils.isBlank(clientSecret)) {
            parameterMap.put("client_secret", new String[]{Constant.CLIENT_SECRET});
        }
        if (!Constant.GRANT_TYPE_CREDENTIALS.equals(grantType)) {
            parameterMap.put("grant_type", new String[]{Constant.GRANT_TYPE_CREDENTIALS});
        }

        ParameterRequestWrapper parameterRequestWrapper = new ParameterRequestWrapper(httpServletRequest, parameterMap);
        parameterRequestWrapper.setMethod("POST");

        filterChain.doFilter(parameterRequestWrapper,httpServletResponse);

    }
    
}
```

### 07 utils工具包
#### 7.1 Constant_常量
`package com.consmation.uaa.utils;`
```java
/**
 * @author SaddyFire
 * @date 2022/4/10
 * @TIME:0:43
 */
public class Constant {

    public static final String CLIENT_ID = "c1";

    public static final String CLIENT_SECRET = "secret";

    public static final String GRANT_TYPE_CREDENTIALS = "client_credentials";

}
```

#### 7.2 ParameterRequestWrapper_参数装饰者类
`package com.consmation.uaa.utils;`
```java
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.util.*;

/**
 * 继承HttpServletRequestWrapper，
 * 装饰类，修改HttpServletRequest参数
 */
public class ParameterRequestWrapper extends HttpServletRequestWrapper {


    private final Map<String, String[]> modifiableParameters;
    private Map<String, String[]> allParameters = null;
    //请求方法
    private String method;

    /**
     * Create a new request wrapper that will merge additional parameters into
     * the request object without prematurely reading parameters from the
     * original request.
     *
     * @param request
     * @param additionalParams
     */
    public ParameterRequestWrapper (final HttpServletRequest request,
                                final Map<String, String[]> additionalParams) {
        super(request);
        modifiableParameters = new TreeMap<>();
        modifiableParameters.putAll(additionalParams);
    }

    @Override
    public String getParameter(final String name) {
        String[] strings = getParameterMap().get(name);
        if (strings != null) {
            return strings[0];
        }
        return super.getParameter(name);
    }

    @Override
    public Map<String, String[]> getParameterMap() {
        if (allParameters == null) {
            allParameters = new TreeMap<>();
            allParameters.putAll(super.getParameterMap());
            allParameters.putAll(modifiableParameters);
        }
        //Return an unmodifiable collection because we need to uphold the interface contract.
        return Collections.unmodifiableMap(allParameters);
    }

    @Override
    public Enumeration<String> getParameterNames() {
        return Collections.enumeration(getParameterMap().keySet());
    }

    @Override
    public String[] getParameterValues(final String name) {
        return getParameterMap().get(name);
    }

    public void setMethod(String method){
        this.method = method;
    }


    @Override
    public String getMethod() {
        return this.method;
    }
}
```

## 2. interface模块
```xml
<!--核心依赖: -->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-security</artifactId>
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


<build>
        <plugins>
            <!-- 资源文件拷贝插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <configuration>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <!-- java编译插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <!--跳过单测-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.2</version>
                <configuration>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>
        </plugins>
    </build>
```