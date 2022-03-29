```yml
server:
  port: 8888
spring:
  application:
    #应用的名称
    name: interface
  datasource:
    # 达梦
#    name: dm8
#    url: jdbc:dm://localhost:5236/INTERFACE1
#    username: INTERFACE1
#    password: PW[xmtij2TpTa3FQpvvFR6xkJyyJgTfWzUd]
#    driver-class-name: dm.jdbc.driver.DmDriver

    # mysql
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/interface0?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull&useSSL=false&allowPublicKeyRetrieval=true
      username: root
      password: PW[w8IoQjdyZt5sfw/LHEresw==]
  servlet:
    multipart:
      max-file-size: 5MB
      max-request-size: 5MB


mybatis-plus:
  configuration:
    #在映射实体或者属性时，将数据库中表名和字段名中的下划线去掉，按照驼峰命名法映射
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
#      id-type: auto
      table-prefix: app_   # mysql表名前缀
interface:
  oss:
    accessKey: b9f0431fac535ec7117e5ff5113ad3df
    secret: 1257303cee61bb377fc5f490d733f399
    endpoint: oss-cn-hangzhou.aliyuncs.com
    bucketName: invoke-test
    url: https://saddyfire-test1.oss-cn-hangzhou.aliyuncs.com/
  whitelist:
    - 测试数据
    - 127.0.0.1
    - localhost

jasypt:
  encryptor:
#    password:
    property:
      prefix: "PW["
      suffix: "]"

```