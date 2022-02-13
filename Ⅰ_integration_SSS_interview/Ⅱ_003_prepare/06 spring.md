##### 说一说Spring中bean的生命周期？
1、首先，我们知道bean对象的创建不是由我们自己创建的，而是由Spring来给我们创建的，而我们需要做的是告诉Spring我们需要创建哪些Bean对象。我们可以通过xml、注解等方式来提供创建Bean对象所以需要的信息，而Spring要创建bean对象，就首先得有一个BeanFactory工厂来创建Bean对象，如果这个Bean对象已经存在，则销毁清空Bean工厂的内容，如果不存在，就会通过DefaultListableBeanFactory方法先创建一个Bean对象工厂（即BeanFactory）。

2、BeanFactory要创建Bean对象，就需要Bean对象的相关信息，这些信息就是通过xml或注解方式获得，所以第二步是需要读取这些bean的相关信息，通过LoadBeanDefinition来加载配置文件，并将相关信息加载成一个个BeanDefinition对象（Bean与BeanDefinition的关系就像类与字节码文件之间的关系）。

3、BeanDefinitions生成后，会通过invokeBeanFactoryPostProcessors（ /ˈprəʊsesər/）方法对所有的BeanDefinitions以及BeanFactory进行后置处理执行：
(1)拿到当前应用上下文 beanFactoryPostProcessors 变量中的值，默认情况返回为空。
(2)实例化并调用所有已注册的 BeanFactoryPostProcessor信息，也就是Bena的相关定义。
BeanFactory工厂的相关增强处理结束后，BanFactory就会正式开始实Bean对象的实例化，由InstantiateSingletons （/ɪn'stænʃɪeɪt/）方法通过反射创建Bean对象，其中会判断该对象是否是单例或者懒加载：
如果不是单例或者不是懒加载，就采用FactoryBean单独构创建对象；
如果是单例对象或懒加载，就由BeanFactory工厂直接反射创建对象。
BeanFactory将对象实例化之后，Spring会通过BeanPostProcessor（bean对象后置处理器）对Bean对象的初始化的前后进行后置处理，填充Bean对象的属性，完成初始化。
通过Map数据类型放这些Bean对象存放在IOC容器中。
bean销毁：注册销毁的回调方法，当对象销毁时，会执行destory-method方法。

##### spring 用到哪些设计模式
           
- 工厂模式：BeanFactory就是简单工厂模式的体现，用来创建对象的实例；
- 单例模式：Bean默认为单例模式。
- 代理模式：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术；
- 模板方法：用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplate。
- 观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新，如Spring中listener的实现–ApplicationListener。

##### BeanFactory和FactoryBean的区别是什么？

BeanFactory：由Bean工厂统一生成对象，相当于一个模子克隆出来；

FactoryBean：单独构造复杂对象，在Spring中BeanFactory进行实例化时，判断该对象不是单例或者不是懒加载形式，就改由FactoryBean来单独创建对象。

##### spring里面的 bean为什么要注册，作用是什么？
注册就是把信息存起来，内部是基于集合实现的，注册方便之后对于信息的调取。


##### 常用注解
`@Component`
`@Controller`
`@Service`
`@Bean`
`@Autowired`

