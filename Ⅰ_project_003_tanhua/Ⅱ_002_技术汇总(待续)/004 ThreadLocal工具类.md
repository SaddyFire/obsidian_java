![[#^f78103]]

## 01 ThreadLocal工具类 & Token拦截器

*com.tanhua.server.interceptor.UserHolder*

定义ThreadLocal工具类，仅需要调用set方法即可将数据存入ThreadLocal中

```java
import com.tanhua.model.domain.User;

/**
 * 令牌拦截器,在每次访问前会进行令牌检测
 * @author Administrator
 */
public class UserHolder {

    private static ThreadLocal<User> tl = new ThreadLocal<User>();

    /**
     * 保存数据到线程
     */
    public static void set(User user) {
        tl.set(user);
    }

    /**
     * 获取线程中的用户信息
     */
    public static User get() {
        return tl.get();
    }


    /**
     * 从当前线程，获取用户对象的id
     */
    public static Long getUserId() {
        if (tl.get() == null) {
            return null;
        }
        return tl.get().getId();
    }

    /**
     * 从当前线程，获取用户对象的手机号码
     */
    public static String getMobile() {
        if (tl.get() == null) {
            return null;
        }
        return tl.get().getMobile();
    }

     /**
     * 移除线程中数据
     */
    public static void remove() {
        tl.remove();
    }
}
```
<br/>

## 02定义拦截器
*com.tanhua.server.interceptor.TokenInterceptor*

定义拦截器，在前置拦截方法preHandle中解析token并验证有效性，如果失效返回状态码401。如果有效，解析User对象，存入ThreadLocal中

```java
import com.tanhua.model.domain.User;
import com.tanhua.utils.JwtUtils;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwt;
import org.apache.commons.lang3.StringUtils;
import org.springframework.http.HttpStatus;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 通过ThreadLocal 将令牌中的用户信息存储至当前线程
 * @author Administrator
 */
public class TokenInterceptor extends HandlerInterceptorAdapter {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        //1、获取请求头
        String token = request.getHeader("Authorization");

        //2、使用工具类，判断token是否有效
        boolean verifyToken = JwtUtils.verifyToken(token);
        //3、如果token失效，返回状态码401，拦截
        if(!verifyToken) {
            response.setStatus(401);
            return false;
        }
        //4、如果token正常可用，放行
        //解析token，获取id和手机号码，
        Claims claims = JwtUtils.getClaims(token);
        String mobile = (String) claims.get("mobile");
        Integer id = (Integer) claims.get("id");
        
        //构造User对象，存入Threadlocal
        User user = new User();
        user.setId(Long.valueOf(id));
        user.setMobile(mobile);

        UserHolder.set(user);

        return true;
    }
    
    //清空
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        UserHolder.remove();
    }
}
```
<br/>

## 03注册拦截器
*com.tanhua.server.config.WebConfig*
拦截器需要注册到MVC容器中

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**  
 * 拦截器配置类  
 */
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new TokenInterceptor())
                .addPathPatterns("/**")
                .excludePathPatterns(
                        "/user/login", 
                        "/user/loginVerification"
                );
    }
}
```

<br/>

## 04 修改控制器方法
修改控制器方法, 所有需要用到**userId**都可以直接从线程中获取
Long userId = UserHolder.getUserId();


```java
@PostMapping(path = "/loginReginfo")
public ResponseEntity loginReginfo(@RequestBody UserInfo userInfo) {
    //1. 校验token
    Long userId = UserHolder.getUserId();
    //2. 保存用户信息
    userInfo.setId(userId);
    userInfoService.save(userInfo);
    //3. 返回数据
    return ResponseEntity.ok(null);
}
```


<br/>

## 思路整理

^f78103

*统一用户鉴权*

需求：使用拦截器完成用户鉴权

步骤分析：

1. 编写拦截器TokenInterceptor，在preHandler方法中进行业务处理 

2. 获取请求header中的Authorization(即token)

3. 根据token解析获取用户对象

4. 判断是否存在，如不存在返回状态码401

5. 如果用户存在，放行。

6. 注册TokenInterceptor













