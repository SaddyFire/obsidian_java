#### 01 网页端:
Data ID: 服务名-dev.yml

- 配置内容:

```yml
# 按需写

```


#### 02 本地端
三要素: 服务名, 环境名, 后缀名
```yml
spring:
	application:
		name: userservice # 服务名
	profiles:
		active: dev # 环境名, 这里是dev
  cloud:
    nacos:
      discovery:  # 服务发现
        server-addr: ${NACOS_HOST:eca-register}:${NACOS_PORT:8848}
        namespace: ${NAMESPACE:extend}
      config:  # 服务注册
        server-addr: ${spring.cloud.nacos.discovery.server-addr}
        file-extension: ${FILE_EXTENSION:yml}
        group: ${GROUP:DEFAULT_GROUP}
        prefix: ${spring.application.name}
        namespace: ${NAMESPACE:extend}  # 后缀名
        shared-configs:
          - data-id: common-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
            refresh: true
          - data-id: ecasqls-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
            refresh: true
```


#### 03 读取顺序

项目启动 -> bootstrap.yml -> 读取nacos中配置文件 -> 读取本地配置文件application.yml
-> 创建spring容器 -> 加载bean


#### 04 证明是否拉到配置中心配置

1. 在配置中心定义数据
```yml
pattern:
  dateformat: yyyy-MM-dd HH:mm:ss
```

2. 在本地Controller层尝试读取数据
```java
@slf4j
@RestController
@RequestMapping("/test")
@RefreshScope    //属性热刷新(方式一)
@ConfigurationProperties(prefix = "pattern")  //属性热更新(方式二), 此处仅供举例
public class TestController{
	@Value("${pattern.dateformat}")
	private String dateformat;

	@GetMapping("now")
	public String now(){
		return LocalDateTime.now().format(DateTimeFormatter.ofPattern(dateformat));
	}
}

```




