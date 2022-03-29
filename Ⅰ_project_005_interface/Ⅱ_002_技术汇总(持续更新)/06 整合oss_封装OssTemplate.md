此处注意使用了hutool包对密钥进行md5加密
##### 01 META-INF包新增自动装配类
resources -> META-INF包 -> spring.factories

例: 
```factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.consmation.demo.autoconfig.InterfaceAutoConfiguration
```




