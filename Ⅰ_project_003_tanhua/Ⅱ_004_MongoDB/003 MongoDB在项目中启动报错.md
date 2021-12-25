当我们在项目中引入了MongoDB依赖之后启动**tanhua-dubbo-db项目和tanhua-app-server项目**

![[Pasted image 20211225200944.png]]

>**原因 :** 实体类模块中引入了MongoDB的依赖，根据自动装配的原理 `tanhua-dubbo-db`和`tanhua-app-server`中会自动查找默认MongoDB的地址（localhost:27017）,而本地有没有开启Mongo所以连接失败。

**解决方案** : `tanhua-dubbo-db`和`tanhua-app-server`中排除掉MongoDB 自动配置类即可