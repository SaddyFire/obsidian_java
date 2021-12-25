# 1. 事务 - 主入口类添加注解

- 添加@EnableTransactionManagement

```java
@SpringBootApplication
@EnableTransactionManagement
public class Mytest20211115SpringbootApplication {

    public static void main(String[] args) {
        SpringApplication.run(Mytest20211115SpringbootApplication.class, args);
    }
}
```

- 在dao接口添加注解@Transactional, 参照上个代码块

# 2. AOP - 通知类

- 追加aop依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

- MyAdvice类(列举添加和删除)
    - 定义组件@Component
    - 声明切面@Aspect
    - 牢记统配符书写
    - @Around中try{中抓取非法输入}—>catch{将自定义异常抛出}

```java
@Component
@Aspect
public class MyAdvice {
    @Around("execution(* com.itheima.service.*.add(..))")
    public Object checkBookParam(ProceedingJoinPoint pjp){
        Book book = (Book) pjp.getArgs()[0];
        //非空判断
        if(book.getName().equals("") && book.getType().equals("") && book.getDescription().equals("")){
            throw new BusinessException("您输入的内容不合法");
        }
        Boolean proceed = null;
        try {

            System.out.println(book);
            proceed = (Boolean) pjp.proceed();
        } catch (Throwable throwable) {
            throw new Exception("系统繁忙,请稍后重试", throwable.getCause());
        }
        return proceed;
    }

    @Around("execution(* com.itheima.service.*.delete(..))")
    public Object checkIdsParam(ProceedingJoinPoint pjp){
        Object proceed = null;
        Integer[] ids = (Integer[]) pjp.getArgs()[0];
        if(ids.length == 0){
            throw new BusinessException("您输入的内容不合法");
        }

        try {
            proceed = pjp.proceed();
        } catch (Throwable throwable) {
            throw new Exception("系统繁忙,请稍后重试",throwable.getCause());
        }
        return proceed;
    }
}
```