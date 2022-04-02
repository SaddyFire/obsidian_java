#### 01 依赖
```xml
<spring-cloud.version>2020.0.1</spring-cloud.version>  
<spring-cloud-alibaba.version>2.2.5.RELEASE</spring-cloud-alibaba.version>
<!--springcloud-->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-dependencies</artifactId>
	<version>${spring-cloud.version}</version>
	<type>pom</type>
	<scope>import</scope>
</dependency>
<!-- spring cloud alibaba 依赖 -->
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-alibaba-dependencies</artifactId>
	<version>${spring-cloud-alibaba.version}</version>
	<type>pom</type>
	<scope>import</scope>
</dependency>

<!--nacos-discovery 此处版本要和alibaba.cloud统一-->
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
	<version>2.2.5.RELEASE</version>
</dependency>
<!--nacos配置中心依赖支持 此处版本要和alibaba.cloud统一-->
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
	<version>2.2.5.RELEASE</version>
</dependency>

```

#### 02 bootstrap.yml配置, 配置nacos
**注意点**
- **NAMESPACE**(2处)要和配置中心统一
- 其余需要改的地方: server.port(**项目端口**),  spring.profiles.active(**环境**), spring.application.name(**项目名**)
- nacos的**ip**, 端口号 ->本地nacos的端口在配置文件中配置
- 其余不变暴露或不需要动态更新的数据也配置在此文件中
```yml
server:
  port: 8888
spring:
  profiles:
    active: dev
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
  application:
    name: interface
  cloud:
    nacos:
      discovery:
        server-addr: ${NACOS_HOST:eca-register}:${NACOS_PORT:8848}
        namespace: ${NAMESPACE:interface}  # 命名空间
      config:
        server-addr: ${spring.cloud.nacos.discovery.server-addr}
        file-extension: ${FILE_EXTENSION:yml}
        group: ${GROUP:DEFAULT_GROUP}
        prefix: ${spring.application.name}
        namespace: ${NAMESPACE:interface}  # 命名空间
        shared-configs:
          - data-id: common-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
            refresh: true
          - data-id: ecasqls-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
            refresh: true
jasypt:
  encryptor:
#    password:
    property:
      prefix: "PW["
      suffix: "]"

# log4j2
#logging:
#  config: classpath:log4j2.yml
```

#### 03 nacos管理端设置

##### 3.1 新增命名空间 -- 此处是interface

##### 3.2 配置列表里新增 -- common-dev.yml
	![[Pasted image 20220402111457.png]]

```yml
spring:
  datasource:
    #  达梦
    name: dm8
    url: jdbc:dm://localhost:5236/INTERFACE1
    username: INTERFACE1
    password: PW[xmtij2TpTa3FQpvvFR6xkJyyJgTfWzUd]
    driver-class-name: dm.jdbc.driver.DmDriver

    #  mysql
  #   druid:
  #     driver-class-name: com.mysql.cj.jdbc.Driver
  #     url: jdbc:mysql://localhost:3306/interface0?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull&useSSL=false&allowPublicKeyRetrieval=true
  #     username: root
  #     password: PW[w8IoQjdyZt5sfw/LHEresw==]
  # servlet:
  #   multipart:
  #     max-file-size: 5MB
  #     max-request-size: 5MB


mybatis-plus:
  configuration:
    #在映射实体或者属性时，将数据库中表名和字段名中的下划线去掉，按照驼峰命名法映射
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      id-type: auto # 按mysql内置策略
      table-prefix: app_   # mysql表名前缀
```

##### 3.4 配置列表里新增 -- 此处是 interface-dev.yml

