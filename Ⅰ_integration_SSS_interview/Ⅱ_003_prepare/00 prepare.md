##### 介绍下HashMap
![[Pasted image 20220213195738.png]]
	
	首先hashmap是一种key-value键值对形式的集合, 在1.7之前他的底层是数组加链表, 1.8之后他的底层是数组, 链表和红黑树. 
	在把数据存入hashmap中时, 会先通过该数据key的hash值进行哈希函数, 类似于对数字的取模取余, 如果得到的角标位置已经有数据了, 也就是hash冲突, 此时会以链表的形式挂下来.
	在1.7之前, 如果链表过长,就会导致每次查询要一个个撸下来, 而1.8之后采用红黑树方式的查询效率就会大大提高. 但是数据少的时候树的结点大小比链表要大, 所以在一个哈希桶数量大于8的时候才会变为红黑树, 小于6的时候会转回链表. 
	还有一点是1.7的hashmap是头插法,速度快,但是在多线程并发和数组扩容的时候会出现循环列表, 导致死锁问题, 所以在1.8之后的红黑树用尾插法, 能相对避免死锁. 在项目中的话, 如果有并发需求, 我们通常会采用concurrentHashmap, 他的底层是对hashmap的哈希桶上锁, 达到多线程安全.
	
##### 介绍下线程池


##### 介绍下JVM

	jvm是指Java virtual Machine, 在一个class文件加载到java虚拟机中, 要经历loading , linking, initializing, 
	其中首先虚拟机要将class文件load进classloader, 加载器会通过双亲委派模型, 从costumclassloader到appClassloader最终到bootstrap, 这样做最终要的原因是安全, 防止底层类库被篡改.同时如果经过了双亲委派模型最后也没找到该类, 则会报classnotfound异常. class文件load进内存之后会进行verification,preparation和resolution,进行字节码文件的解析, initializing是对静态变量的初始化赋值.
	java运行后会进入jvm运行时数据区, 

##### 谈谈对spring的理解
spring广义上来说是一个完整的生态, spring对项目的开发有超强的拓展性,spring家族让开发进入了春天时代. 侠义的spring框架核心主要是ioc和aop, ioc是指inversion of control 也就是控制反转, 本质是一种思想, 是一种程序之间的解耦思想,将对象的创建权力反转给spring. ioc也是工厂模式, 单例模式, 模板方法, 观察者设计模式的一种实现. 
aop是aspect 

##### spring的bean生命周期
[[06 spring#说一说Spring中bean的生命周期？|答案]]

##### 谈一下SpringMVC的加载过程
SpringMVC是指spring module view controller, 也叫模型视图控制器, 把web层进行解耦, 简化开发
首先springMVC的入口是DispatcherServlet(`package org.springframework.web.servlet`)前端控制器, 其中有核心方法`initStrategies()`初始化了映射处理器, `doDispatch() `处理请求方法, 并且获取到`HandlerAdapter`, 也就是处理器适配器, 最终调用了后端处理器`handle()`方法处理请求, 并返回`ModelAndView`模型视图

##### SpringBoot的工作原理/自动配置/SPI机制是怎么样子的？
首先SPI 是Service Provider Interface, 是一种服务发现机制, springboot的底层也使用到了spi机制
`@SpringBootApplication`  -> `@EnableAutoConfiguration` -> `@import` 中导入的类会被加载到spring IOC容器中 ->  最终会被spring反射调用classLoader的getResourcees 把`META-INFO`下的`spring.factories`文件里的类load进jvm




