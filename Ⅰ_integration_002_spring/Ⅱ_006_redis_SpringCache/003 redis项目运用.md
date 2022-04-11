### 01 依赖
```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.4.5</version>
            </plugin>
        </plugins>
    </build>
```

### 02 RedisConfig
`package com.consmation.demo.config;`
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericToStringSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

/**
 * @author SaddyFire
 * @date 2022/4/10
 * @TIME:22:33
 */
@Configuration
public class RedisConfig {

    /**
     * 设置 redisTemplate 的序列化设置
     * @param redisConnectionFactory
     * @return
     */
    @Bean
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        // 1.创建 redisTemplate 模版
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        // 2.关联 redisConnectionFactory
        template.setConnectionFactory(redisConnectionFactory);
        // 3.创建 序列化类
        GenericToStringSerializer genericToStringSerializer = new GenericToStringSerializer(Object.class);
        // 6.序列化类，对象映射设置
        // 7.设置 value 的转化格式和 key 的转化格式
        template.setValueSerializer(genericToStringSerializer);
        template.setKeySerializer(new StringRedisSerializer());
        template.afterPropertiesSet();
        return template;
    }
}
```


### 03 自增demo
注意此处不要泛型
```java
    @Autowired
    private RedisTemplate redisTemplate;

    @GetMapping("/redis")
    public ResponseResult redisTest(String key) {
        Object testObj = redisTemplate.opsForValue().get(key);
        Integer testInt;
        if (ObjectUtils.isEmpty(testObj)) {
            redisTemplate.opsForValue().set(key, 1);
        } else {
            //若存在则强转
            testInt = Integer.valueOf((String) testObj);
            System.out.println(testInt);
            redisTemplate.opsForValue().increment(key, 1L);
        }
        Object test = redisTemplate.opsForValue().get(key);
        System.out.println((String) test);
        return ResponseResult.okResult("递增之后的数据为: " + test);
    }
```