## Maven 学习

- 什么是Maven

![[cce42f80069d44af944449f1860da120.png]]

- Maven的必要性：

由于 Java 的生态非常丰富，无论你想实现什么功能，都能找到对应的工具类，这些工具类都是以 jar 包的形式出现的，例如 Spring，SpringMVC、MyBatis、数据库驱动，等等，都是以 jar 包的形式出现的，jar 包之间会有关联，在使用一个依赖之前，还需要确定这个依赖所依赖的其他依赖，所以，当项目比较大的时候，依赖管理会变得非常麻烦臃肿，这是 Maven 解决的第一个问题。

Maven 还可以处理多模块项目。简单的项目，单模块分包处理即可，如果项目比较复杂，要做成多模块项目，例如一个电商项目有订单模块、会员模块、商品模块、支付模块…，一般来说，多模块项目，每一个模块无法独立运行，要多个模块合在一起，项目才可以运行，这个时候，借助 Maven 工具，可以实现项目的一键打包。

- Maven的两大核心：
- 依赖管理：对 jar 的统一管理(Maven 提供了一个 Maven 的中央仓库，[https://mvnrepository.com/，当我们在项目中添加完依赖之后，Maven](https://mvnrepository.com/%EF%BC%8C%E5%BD%93%E6%88%91%E4%BB%AC%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E6%B7%BB%E5%8A%A0%E5%AE%8C%E4%BE%9D%E8%B5%96%E4%B9%8B%E5%90%8E%EF%BC%8CMaven) 会自动去中央仓库下载相关的依赖，并且解决依赖的依赖问题)
- 项目构建：对项目进行编译、测试、打包、部署、上传到私服等
- Maven坐标：

俗称 gav：使用下面三个向量子仓库中唯一定位一个 Maven 工程

在项目中的 pom.xml 文件中，我们可以看到下面gav的定义：

1、groupid:公司或组织域名倒序

&lt;groupid&gt;com.ys.maven&lt;/groupid&gt;

2、artifactid:模块名，也是实际项目的名称

&lt;artifactid&gt;Maven_05&lt;/artifactid&gt;

3、version:当前项目的版本

&lt;version&gt;0.0.1-SNAPSHOT&lt;/version&gt;

groupId,artifactId和version：依赖的基本坐标，对于任何一个依赖来说，基本坐标是最重要的，Maven根据坐标才能找到需要的依赖。

- Maven的group-Id：

Group-id分为三部分，每个部分以“.”相隔，第一部分是项目用途，比如用于商业的就是“com”，用于非盈利性质就是“org”，第二部分是公司名，比如“alibaba”，“jingdong”，第三部分是项目名。

Maven 的 文件目录结构：

bin binary二进制文件的简称，里面存放的一般是可执行的[二进制文件](https://baike.baidu.com/item/%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%96%87%E4%BB%B6)

boot 里面存放启动目录的核心文件

conf 里面存放配置文件，包含核心全局配置文件settings.xml

lib 里面存放类库或者资源文件

- Maven 仓库：

![[bd4a053e669a49019fefd1736942f19e.png]]

仓库选址优先级：

本机仓库>setting.xml配置的镜像仓库>默认中央仓库

- Maven 依赖：

Maven依赖具有传递性

![[0c922aaf1c5e478ba4f91b20a452c311.png]]

依赖范围

![[cebcbddb2cc64d28b106360a36acea46.png]]

![[8900cb19b7144d599e0435f3020873f3.png]]

![[36cee84081f842ff9437d1975fc07c57.png]]
父工程可以通过pom.xml配置import属性结合dependencyManagement，强制规范子工程项目版本举例：

父类定义如下：

![](:/f933d270728d4f3fb83217b8457e478f)

![[385a06cdd67a4ed984cb9277f4be12d9.png]]

![[8eac6a77803346f78cf290f3c90369ba.png]]

子类定义如下：

![[9e2f81f047484fd3b3e240d7ed7cdfc3.png]]

模块聚合：

![[01c39a6bc3be4e6f9359b6ef62cbe5f8.png]]

总项目一般为pom项目（在 pom.xml 文件中设置）

![[731063b4222e4ec5ba6df54c8c44a160.png]]

在modules中添加模块

模块中定义如下：

![[61cb170289294e67b8ccd7973e99cb50.png]]

