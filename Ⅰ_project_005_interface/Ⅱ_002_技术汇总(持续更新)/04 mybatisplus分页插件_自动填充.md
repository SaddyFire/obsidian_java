- mybatisplus如果报错可能是id增长策略错误, 要在yml中配置
```yml
mybatis-plus:
  configuration:
    #在映射实体或者属性时，将数据库中表名和字段名中的下划线去掉，按照驼峰命名法映射
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
#      id-type: auto
      table-prefix: app_   # mysql表名前缀
```
## 1. mybatisplus分页插件
##### 01 MyBatisPlusConfig
`package com.consmation.demo.config;`
```java
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
/**
 * @author SaddyFire
 * @date 2022/3/24
 * @TIME:18:26
 * 处理器配置类
 */

@Configuration
public class MyBatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor getMPI(){
        //创建拦截器容器
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        //向容器增加分页拦截器
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        return interceptor;
    }
}
```

## 2. mybatisplus自动填充
##### 01 自动填充处理器
`package com.consmation.demo.handler;`
```java
import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import org.apache.ibatis.reflection.MetaObject;
import org.springframework.stereotype.Component;
import java.util.Date;
/**
 * @author SaddyFire
 * @date 2022/3/27
 */
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        Object created = getFieldValByName("created", metaObject);
        if (null == created) {
            //字段为空，可以进行填充
            setFieldValByName("created", new Date(), metaObject);
        }

        Object updated = getFieldValByName("updated", metaObject);
        if (null == updated) {
            //字段为空，可以进行填充
            setFieldValByName("updated", new Date(), metaObject);
        }
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        //更新数据时，直接更新字段
        setFieldValByName("updated", new Date(), metaObject);
    }
}
```

##### 02 定义BasePojo, 需要自动填充的类继承此类即可
`package com.consmation.demo.model.pojo;`
```java
import com.baomidou.mybatisplus.annotation.FieldFill;
import com.baomidou.mybatisplus.annotation.TableField;
import lombok.Data;

import java.io.Serializable;
import java.util.Date;

/**
 * @author SaddyFire
 * @date 2022/3/27
 */
@Data
public class BasePojo implements Serializable {
    @TableField(fill = FieldFill.INSERT) //自动填充
    private Date created;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updated;
}
```


