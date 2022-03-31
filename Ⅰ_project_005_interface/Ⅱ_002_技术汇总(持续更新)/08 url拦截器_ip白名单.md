此处核心是WebConfig拦截机配置类

##### 01 application.yml配置
```yml
interface:  
  oss:  
    accessKey: b9f0431fac535ec7117e5ff5113ad3df  
    secret: 1257303cee61bb377fc5f490d733f399  
    endpoint: oss-cn-hangzhou.aliyuncs.com  
    bucketName: invoke-test  
    url: https://saddyfire-test1.oss-cn-hangzhou.aliyuncs.com/  
  whitelist:  
    - 127.0.0.1  
    - localhost
```

##### 02 InterfaceProperties属性类
`package com.consmation.demo.autoconfig.properties;`
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
@ConfigurationProperties(prefix="interface")  
public class InterfaceProperties {  
	private List<String> whiteList;  
}
```

##### 03 AuthorizationInterceptor属性注入
`package com.consmation.demo.interceptor;`
获取访客url, 以及校验
```java
import com.consmation.demo.autoconfig.properties.InterfaceProperties;
import com.consmation.demo.exception.CustomException;
import com.consmation.demo.model.enums.AppHttpCodeEnum;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.util.ObjectUtils;
import org.springframework.util.StringUtils;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.net.URL;
import java.util.List;

/**
 * @author SaddyFire
 * @date 2022/3/28
 * @TIME:19:14
 */
public class AuthorizationInterceptor implements HandlerInterceptor {

    @Autowired
    private InterfaceProperties interfaceProperties;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //白名单校验
        if (ObjectUtils.isEmpty(interfaceProperties) || ObjectUtils.isEmpty(interfaceProperties.getWhiteList())) {
            throw new CustomException(AppHttpCodeEnum.SERVER_ERROR);
        }
        //获取url
        StringBuffer requestURL = request.getRequestURL();  //  http://localhost:8888/demo/limit
        //校验url
        if (StringUtils.isEmpty(requestURL)) {
            throw new CustomException(AppHttpCodeEnum.PARAM_REQUIRE);
        }
        URL url = new URL(requestURL.toString());
        String host = url.getHost();
        //ip校验
        List<String> whiteList = interfaceProperties.getWhiteList();
        if (!whiteList.contains(host)) {
            throw new CustomException(AppHttpCodeEnum.NO_OPERATOR_AUTH);
        }
        return preHandle(request, response, handler);
    }
}
```

##### 04 WebConfig拦截机配置类(核心)
`package com.consmation.demo.config;`

**说明: 此处一定要注意先注入AuthorizationInterceptor() Bean, 然后将bean添加至拦截器**
```java
import com.consmation.demo.interceptor.AuthorizationInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * 拦截器配置类  
 */
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public HandlerInterceptor getDataInterceptor(){
        return new AuthorizationInterceptor();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //添加拦截器
        registry.addInterceptor(getDataInterceptor())
                .addPathPatterns("/**")
                .excludePathPatterns(
                        "/user/login",
                        "/user/loginVerification"
                );
    }
}
```

**错误写法**
```java
/**
 * 拦截器配置类  
 */
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //添加拦截器
        registry.addInterceptor(new AuthorizationInterceptor())
                .addPathPatterns("/**")
                .excludePathPatterns(
                        "/user/login",
                        "/user/loginVerification"
                );
    }
}
```