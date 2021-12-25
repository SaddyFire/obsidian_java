- <ins>clean命令</ins>

清除由项目编译创建的**target**

- <ins>validate命令</ins>

验证项目是否正确并且所有必要的信息均可用

- <ins>compile命令</ins>

编译项目的源代码

- <ins>test命令</ins>

使用合适的单元测试框架来测试编译的源代码。 这些测试不应要求将代码打包或部署

- <ins>verify命令</ins>

对集成测试的结果进行任何检查，以确保符合质量标准

- <ins>package命令</ins>

完成了项目编译、单元测试、打包功能，但没有把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库和远程maven私服仓库

- <ins>install命令</ins>

完成了项目编译、单元测试、打包功能，同时把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库，但没有布署到远程maven私服仓库

- <ins>deploy命令</ins>

完成了**项目编译、单元测试、打包功能**，同时把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库和远程maven私服仓库

- mvn clean install

依次执行了clean、resources、compile、testResources、testCompile、test、jar(打包)、install等8个阶段。

- <ins>site命令</ins>

用于为Maven项目生成站点（用以生成HTML页面的模块等文档）