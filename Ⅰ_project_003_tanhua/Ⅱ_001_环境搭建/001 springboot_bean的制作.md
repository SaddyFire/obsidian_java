## 01 根据官方文档编辑测试类


整理出**需要传参**&&**需要定义在配置文件**的变量

<br/>

## 02 ×××Properties 配置类的定义

**@ConfigurationProperties(prefix = "tanhua.sms")** 注解:

声明配置属性

## 03 ×××Template 模板类的定义

1. 引入配置类: 如下

```java
public class SmsTemplate {  
  
	 private SmsProperties properties;  

	 public SmsTemplate(SmsProperties properties) {  
		 this.properties = properties;  
	 }  
  
}

```

2. 根据官方文档测试类进行封装

		注意: 确定参数与返回值类型
		

## 04 **项目名+**AutoConfiguration 自动装配类

注意:

**@EnableConfigurationProperties** 要装配配置类的class文件

**@Bean** 将模板类装配到spring容器

如果有多个封装对象则装配多个**bean**

```java
@EnableConfigurationProperties({
        SmsProperties.class,
        OssProperties.class,
        AipFaceProperties.class
})
public class TanhuaAutoConfiguration {

    @Bean
    public SmsTemplate smsTemplate(SmsProperties properties) {
        return new SmsTemplate(properties);
    }

    @Bean
    public OssTemplate ossTemplate(OssProperties properties) {
        return new OssTemplate(properties);
    }

    @Bean
    public AipFaceTemplate aipFaceTemplate(AipFaceProperties aipFaceProperties){
        return new AipFaceTemplate(aipFaceProperties);
    }
}

```

## 05 自动装配配置(只需一次)

根据自动装配原则，在**tanhua-autoconfig**模块创建 **/resources/META-INF/spring.factories** 文件

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\  
com.tanhua.autoconfig.TanhuaAutoConfiguration
```


## 06 消费者的 ->  application.yml文件
格式: 以tanhua为例
- 注意层级关系
- 将**配置类定义的属性** 与 官方test文件中**自己的信息**填入
- 注意: 此yml文件 谁调用, 谁提供

```yml
tanhua:  
  sms:  
    appCode: adc4473074cb49c1a024e80be86328f5  
    url: https://smssend.shumaidata.com/sms/send  
    templateId: MD8B7116BC
```


## 07 工程编写单元测试类
```java
import com.tanhua.autoconfig.template.SmsTemplate;
import com.tanhua.server.AppServerApplication;
import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.io.IOException;

@RunWith(SpringRunner.class)
@SpringBootTest(classes = AppServerApplication.class)
public class SmsTemplateTest {

    //注入
    @Autowired
    private SmsTemplate smsTemplate;

    //测试
    @Test
    public void testSendSms() throws IOException {
        smsTemplate.sendSms("15555054404","1234");
    }
}
```
