## 解决session存储用户id的弊端, 采用ThreadLocal类

*com.itheima.reggie.common.BaseContext*
    
- **说明:** 此处是采用ThreadLocal可以存储数据的功能, 将登录用户id存储至ThreadLocal对象, 在拦截器放行的时候存储id. 在自动装配时调用
    

```java
package com.itheima.common;

/**
 * @author SaddyFire
 * @date 2021/11/24
 * @TIME:10:38
 */
public class BaseContext {
    private static ThreadLocal<Long> threadLocal = new ThreadLocal<>();

    public static void setLoginId(Long id){
        threadLocal.set(id);
    }

    public static Long getLoginId(){
        return threadLocal.get();
    }
}
```

*com.itheima.filter.LoginCheckFilter*
    

```java
//优化:3、如果不需要处理，则直接放行
if(check){
    log.info("本次请求{}不需要处理",uri);
    filterChain.doFilter(request,response);
    return;
}
HttpSession session = request.getSession();
Long employeeId = (Long) session.getAttribute("employeeId");
if(employeeId != null){
    BaseContext.setLoginId(employeeId);		//取得放行资格后, 存储当前id
    filterChain.doFilter(request,response);
    return;
}
//相应结果给前端
response.getWriter().write(JSON.toJSONString(R.error("NOTLOGIN")));
return;
```
<br/>

(见 007 工具类 )*com.itheima.common.MyMetaObjecthandler*

![[007 创建时间-创建人等自动填充功能#2 工具类–自定义元数据对象处理器]]



    
