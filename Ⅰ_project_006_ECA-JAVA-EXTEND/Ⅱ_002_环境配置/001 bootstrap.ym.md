#### 配置文件关联nacos
**注意: **

host配置映射
nacos端口在conf文件夹
```yml
spring:
  profiles:
    active: dev
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
  application:
    name: j-dataexchangeserver
  cloud:
    nacos:
      discovery:
        server-addr: ${NACOS_HOST:eca-register}:${NACOS_PORT:8868}
        namespace: ${NAMESPACE:eca}
      config:
        server-addr: ${spring.cloud.nacos.discovery.server-addr}
        file-extension: ${FILE_EXTENSION:yml}
        group: ${GROUP:DEFAULT_GROUP}
        prefix: ${spring.application.name}
        namespace: ${NAMESPACE:eca}
        shared-configs:
          - data-id: common-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
            refresh: true
          - data-id: ecasqls-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
            refresh: true
server:
  port: 28820
```


