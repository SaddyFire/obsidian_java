## 1 Nacos的安装及启动
```cmd
startup.cmd -m standalone
```
## 2 引入依赖
#### 2.1父工程依赖
```xml
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-alibaba-dependencies</artifactId>
	<version>2.2.6.RELEASE</version>
	<type>pom</type>
	<scope>import</scope>
</dependency>
```
#### 2.2子工程依赖
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```
## 3 配置nacos地址
```
spring:
  cloud:
    nacos:
      server-addr: 127.0.0.1:8848
```
