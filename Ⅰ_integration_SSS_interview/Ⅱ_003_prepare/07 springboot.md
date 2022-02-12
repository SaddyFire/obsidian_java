##### SringBoot的核心注解是哪个？它主要由哪几个注解组成？
``@SpringBootApplication`
该注解包含 
`@SpringBootConfiguration`
`@EnableAutoConfiguration`
`@ComponentScan`

##### SpringBoot的工作原理/自动配置/SPI机制是怎么样子的？
`@SpringBootApplication`  -> `@EnableAutoConfiguration` -> `@import` 中导入的类会被加载到spring IOC容器中 ->  `SpringFactoriesLoader` 反射出`META-INFO`下的`spring.factories`文件