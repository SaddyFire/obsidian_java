### 01 相关网址
1. 博客园
https://www.cnblogs.com/lixingwu/articles/14889716.html

2. Spring Cloud 官网找到对应的说明
https://docs.spring.io/spring-cloud/docs

3. Spring Cloud Alibaba + Spring Cloud
https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E

Spring Cloud Alibaba 文档托管在github ，可以在下面的网址看到版本说明：

4. 第三方依赖的版本
假设我们现在需要使用 `mybatis-spring-boot-starter` 这个依赖，这个依赖的版本在对应的官网有说明。

http://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/


### 02 alibaba组件版本关系
具体见 01标题 3# 网址

![[Pasted image 20220410231618.png]]

### 03 毕业版本依赖关系(推荐使用)

![[Pasted image 20220410231755.png]]


### 04 依赖管理

Spring Cloud Alibaba BOM 包含了它所使用的所有依赖的版本。

RELEASE 版本

1. Spring Cloud 2021

如果需要使用 Spring Cloud 2021 版本，请在 dependencyManagement 中添加如下内容
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2021.0.1.0</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

2. Spring Cloud 2020

如果需要使用 Spring Cloud 2020 版本，请在 dependencyManagement 中添加如下内容
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2021.1</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```
3. Spring Cloud Hoxton

如果需要使用 Spring Cloud Hoxton 版本，请在 dependencyManagement 中添加如下内容

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.2.7.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

4. Spring Cloud Greenwich

如果需要使用 Spring Cloud Greenwich 版本，请在 dependencyManagement 中添加如下内容

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.1.4.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

5. Spring Cloud Finchley

如果需要使用 Spring Cloud Finchley 版本，请在 dependencyManagement 中添加如下内容

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.0.4.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

6. Spring Cloud Edgware

如果需要使用 Spring Cloud Edgware 版本，请在 dependencyManagement 中添加如下内容
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>1.5.1.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

### 05 孵化器版本依赖关系(不推荐使用)

![[Pasted image 20220410232352.png]]

