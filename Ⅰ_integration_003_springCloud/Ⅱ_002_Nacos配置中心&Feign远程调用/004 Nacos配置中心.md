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
        namespace: ${NAMESPACE:extend}  # 命名空间要和nacos对应
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


#### 05 多环境的共享
微服务启动时会从nacos读取多个配置文件
- \[spring.application.name\]-\[spring.profiles.active\].yml    例如: userservice-dev.yml
- \[spring.application.name\].yml    例如: userservice.yml

这样, 无论profile如何变化, \[spring.application.name\].yml 这个文件一定会加载, 因此多环境共享配置可以写入这个文件

#### 06 多种配置的优先级

- 服务名-profile.yml > 服务名.yml > 本地配置