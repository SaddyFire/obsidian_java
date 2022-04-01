### RocketMQ 

#### 编译

```shell
# 在rocketmq-all-4.9.2/ 目录下cmd 并进行编译
mvn -Prelease-all -DskipTests clean install -U
# 编译后文档在此处
cd distribution/target/rocketmq-4.9.2/rocketmq-4.9.2
```

**编译时报错:** 
No compiler is provided in this environment. Perhaps you are running on a JRE rather than a JDK?

**原因分析:** 
maven加载的JDK路径不对, 
在MINGW64 中执行mvn -v 命令 -> Java home: C:\Program Files\Java\jre

在cmd中执行mvn -v命令 -> Java home: C:\Program Files\Java\jdk1.8.0_161\jre

**结论:**
在执行mvn命令的时候传入JDK路径不正确

**解决方案:**
编辑注册表中相关的配置项，来解决这个问题。配置如下所示：
注册表路径:
`HKEY_LOCAL_MACHINE\SOFTWARE\JavaSoft\Java Runtime Environment`
JavaHome 改为 `C:\Program Files\Java\jre`
RuntimeLib 改为 `C:\Program Files\Java\jre\bin\server\jvm.dll`



**java报错**
Error: could not open C:\Program Files\Java\jre1.8.0_121\lib\amd64\jvm.cfg

**解决: **新增java环境变量, 并将`%JAVA_HOME%\jre\bin`上移至顶端, 同时将用户的环境变量删除

#### linux安装

```shell
tar -zxvf rocketmq-4.9.2.tar.gz -C /usr/local

# 配置环境变量, 添加至最低行
vim /etc/profile
export ROCKETMQ=/usr/local/rocketmq-4.9.2
export PATH=$PATH:$ROCKETMQ/bin
source /etc/profile		# 刷新配置
chmod +x mqnamesrv mqbroker mqshutdown	# 添加权限
```

#### 配置

```shell
mkdir -p /root/log/rocketmqlogs
touch namesrv.log broker.log

# 进入/bin目录并找到runserver.sh和runbroker.sh文件，根据自己的内存环境配置
cd /usr/local/rocketmq-4.9.2/bin/
JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m"
# 保存后返回，打开conf目录下的broker.conf文件，在最后面添加外网IP配置：
brokerIP1 = 192.168.159.129		# 不配置默认是内网IP
```

#### 启动

```shell
mqnamesrv &		# 启动namesrv, 按crtl+c退出命令
mqbroker -n 192.168.159.129:9876 -c /usr/local/rocketmq-4.9.2/conf/broker.conf &  # 启动broker
jps		# 查看启动状态
mqshutdown broker	# 关闭broker
mqshutdown name		#关闭namesrv
```

### rocketmq-dashboard

官网地址：https://github.com/apache/rocketmq-dashboard

#### 编译后windows打开

更改application.yml配置文件

```yml
server:
	prot: 8080	# 此处改访问端口
rocketmq:	
	config:
		namesrvAddrs:
			- 192.168.159.129:9876		# 此处改成RocketMQ的ip及端口
```

```shell
mvn clean
mvn install
# 此处可能无法连接github
# hosts文件更改添加dns
20.205.243.166 www.github.com
185.199.108.153	assets-cdn.github.com
31.13.88.169 github.global.ssl.fastly.net

# 完成jar包位于rocketmq-dashboard/target/ 
# 以及 maven仓库respository/org/apache/rocketmq/rocketmq-dashboard/1.0.1-SNAPSHOT/下

java -jar rocketmq-dashboard-1.0.1-SNAPSHOT.jar		# windows开启
```

#### linux服务化可视化界面

```shell
mkdir -p /usr/local/rocketmq-dashboard
cd /usr/local/rocketmq-dashboard	# 上传jar包

# 前台启动
java -jar /usr/local/rocketmq-dashboard/rocketmq-dashboard-1.0.1-SNAPSHOT.jar
# 后台启动
nohup java -jar /usr/local/rocketmq-dashboard/rocketmq-dashboard-1.0.1-SNAPSHOT.jar &

# 查看进程
jps|grep jar
ps -ef|grep rocketmq-dashboard-1.0.1-SNAPSHOT.jar

# 杀进程
kill -9 pid
```







