## 01 依赖&配置类

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

- [ ] **注意缩进**

```yml
redis:
  host: localhost
  port: 6379
  password: 
  database: 0
cache:
  redis:
    time-to-live: 1800000   #设置缓存过期时间，可选
```

## 02 注入CacheManager

![[5d29a4a562f54006a48f517eeb89e466.png]]

## 03 引导类加@EnableCaching

![[5e766313c126486686dde179f8f018ef.png]]

## 04 注解详情

### 4.1 在save方法上加注解@CachePut

> @CachePut 说明：
> 
> 作用: 将方法返回值，放入缓存
> 
> value: 缓存的名称, 每个缓存名称下面可以有很多key
> 
> key: 缓存的key ----------> 支持Spring的表达式语言SPEL语法

![[b28c1c8d48cf4685bc939c45fcd8049b.png]]

> key的写法如下：
> 
> \#user.id : \#user指的是方法形参的名称, id指的是user的id属性 , 也就是使用user的id属性作为key ;
> 
> \#user.name: \#user指的是方法形参的名称, name指的是user的name属性 ,也就是使用user的name属性作为key ;
> 
> \#result.id : \#result代表方法返回值，该表达式 代表以返回对象的id属性作为key ；
> 
> \#result.name : \#result代表方法返回值，该表达式 代表以返回对象的name属性作为key ；

### 4.2  在 delete 方法上加注解@CacheEvict

> @CacheEvict 说明：
> 
> 作用: 清理指定缓存
> 
> value: 缓存的名称，每个缓存名称下面可以有多个key
> 
> key: 缓存的key ----------> 支持Spring的表达式语言SPEL语法

![[b0b31085794f47b8a44bc3234d9125c6.png]]

参数为列表**的批量更改或删除 :** <ins>采用全部删除</ins>

![[dafb15fb570443c285d3d0d48b581fad.png]]

### 4.3  在 update 方法上加注解@CacheEvict

![[9975101fc62e47ff921856513d60aec3.png]]

### 4.4  在getById上加注解@Cacheable

[[001 markdown常用语法#三级标题]]

> @Cacheable 说明:
> 
> 作用: 在方法执行前，spring先查看缓存中是否有数据，如果有数据，则直接返回缓存数据；若没有数据，调用方法并将方法返回值放到缓存中
> 
> value: 缓存的名称，每个缓存名称下面可以有多个key
> 
> key: 缓存的key ----------> 支持Spring的表达式语言SPEL语法

![[993fe7b7c66040a7a690e58926ef2e31.png]]