### 安装虚拟机

1. 创建新的虚拟机

2. "典型"配置

3. "稍后安装操作系统"

4. "CentOS7 64位" --->> 启动上述创建的新虚拟机

5. "Install CentOS7"

6. "64G磁盘空间"

7. "最小安装""NAT模式"

8. 编辑虚拟机设置

9. 改时区, 改网络, 分盘, 设置"root"用户密码

10. 修改网卡配置项

11. 改NAT网络

```shell
vim /etc/sysconfig/network-scripts/ifcfg-ens33	# 配置静态ip文件

1. 把UUID删了 (两次dd)
2. BOOTPROTO=dhcp -> 改成static
3. ONBOOT=no -> 改成yes
4. 最后一行添加: IPADDR=192.168.123.100
			NETMASK=255.255.255.0
			GATEWAY=192.168.123.2
			DNS1=114.114.114.114

切记不用按 ctrl + s
如果ctrl +s 后再按 ctrl + q

进入文件后执行如下操作: 
①. 按 i 键 		 进入编辑状态
②. 按↑↓键来移动光标, 删除no,输入yes 
③. 按 ESC 键
④. 输入 :wq
⑤. 按 ENTER	保存退出
```

12. 重启虚拟机

#### 设置静态ip

1. 设置静态IP

   `vim /etc/sysconfig/network-scripts/ifcfg-en33`

   ```shell
   TYPE=Ethernet
   PROXY_METHOD=none
   BROWSER_ONLY=no
   BOOTPROTO=static
   IPADDR="192.168.138.100"        # 设置的静态IP地址
   NETMASK="255.255.255.0"         # 子网掩码
   GATEWAY="192.168.138.2"         # 网关地址
   DNS1="192.168.138.2"            # DNS服务器
   DEFROUTE=yes
   IPV4_FAILURE_FATAL=no
   IPV6INIT=yes
   IPV6_AUTOCONF=yes
   IPV6_DEFROUTE=yes
   IPV6_FAILURE_FATAL=no
   IPV6_ADDR_GEN_MODE=stable-privacy
   NAME=ens33
   UUID=afd0baa3-8bf4-4e26-8d20-5bc426b75fd6
   DEVICE=ens33
   ONBOOT=yes
   ZONE=public
   
   # 上述我们所设置的网段为138，并不是随意指定的，需要和我们虚拟机中的虚拟网络编辑器中的NAT模式配置的网关保持一致
   systemctl restart network	# 重启网络服务
   ```

#### 软件安装限制/关闭计算机

```shell
vi /etc/selinux/config
SELINUX=enforcing -> 改成 disabled 

# 关闭计算机
poweroff
shutdown -h now
```

#### 拍摄虚拟机快照

```shell
右键虚拟机-快照-拍摄快照

# 描述:
修改网络
关闭防火墙
软件安装检查
```

#### 虚拟机克隆



### 1. jdk1.8安装

1. 上传文件

2. 解压

   ```shell
   tar -zxvf jdk-8u171-linux-x64.tar.gz -C /usr/local
   ```

3. 配置环境变量

   ```shell
   vim /etc/profile
   # 切换至文件最后
   G
   i
   # 插入环境变量
   JAVA_HOME=/usr/local/jdk1.8.0_171
   PATH=$JAVA_HOME/bin:$PATH
   :wq
   # 重新加载profile文件
   source /etc/profile
   # 测试是否成功
   java -version
   ```

### 2. MySQL安装

采用RPM安装方式，Red-Hat Package Manager

1. 检测是否安装过MySQL相关数据库

   ```shell
   rpm -qa		# 查询当前系统中安装的所有软件
   rpm -qa | grep mysql	# 查询但概念系统安装带mysql的软件
   rpm -qa | grep mariadb	# 查询当前系统中带mariadb的软件
   ```

2. 卸载mariadb

   ```shell
   rpm -e --nodeps mariadb-libs-5.5.60-1.el7_5.x86_64
   ```

3. 解压

   ```shell
   mkdir /usr/local/mysql
   tar -zxvf mysql-5.7.25-1.el7.x86_64.rpm-bundle.tar.gz -C /usr/local/mysql
   ```

4. 安装rpm安装包(按顺序)

   ```shell
   rpm -ivh mysql-community-common-5.7.25-1.el7.x86_64.rpm
   rpm -ivh mysql-community-libs-5.7.25-1.el7.x86_64.rpm
   rpm -ivh mysql-community-devel-5.7.25-1.el7.x86_64.rpm
   rpm -ivh mysql-community-libs-compat-5.7.25-1.el7.x86_64.rpm
   rpm -ivh mysql-community-client-5.7.25-1.el7.x86_64.rpm
   
   yum install net-tools
   rpm -ivh mysql-community-server-5.7.25-1.el7.x86_64.rpm
   ```

5. 启动MySQL

   ```shell
   systemctl status mysqld		# 查看mysql服务状态
   systemctl start mysqld		# 启动mysql服务
   systemctl stop mysqld		# 停止mysql服务
   systemctl enable mysqld		# 开机自启
   
   netstat -tunlp		# 查看已经启动的服务
   netstat -tunlp | grep mysql		# 查看mysql的服务信息
   
   ps -ef | grep mysql		# 查看mysql进程
   ```

6. MySQL登录

   ```shell
   cat /var/log/mysqld.log | grep password		# 查询password的行信息
   mysql -uroot -p		# 登录mysql
   
   set global validate_password_length=4	# 设置密码长度最低位数
   set global validate_password_policy=LOW	# 设置密码安全等级低
   set password = password('123456')		# 设置密码为123456
   
   # 开启访问权限
   grant all on *.* to 'root'@'%' identified by '123456';	# 第二个'root'表示密码
   flush privileges;
   exit;
   
   # 开启防火墙
   firewall-cmd --zone=public --add-port=3306/tcp --permanent
   firewall-cmd --reload
   ```

### 3. lrzsz安装

```shell
yum list lrzsz	# 搜索lrzsz安装包
yum install lrzsz.x86_64	# 在线安装
rz 		# 上传文件
yum repolist	# 检查当前的yum源

yum install vim	# a
```



### 4. Tomcat安装

```shell
rz 		# 上传文件
tar -zxvf apache-tomcat-7.0.57.tar.gz -C /usr/local
cd /usr/local/apache-tomcat-7.0.57/
cd bin
sh startup.sh或者./startup.sh
```

**Tomcat进程查看**

1. 查看启动日志

Tomcat的启动日志输出在Tomcat的安装目录下的logs目录中，Tomcat的启动及运行日志文件名为 catalina.out，所以我们查看Tomcat启动日志，主要可以通过两条指令，如下：

```shell
1). 分页查询Tomcat的日志信息
more /usr/local/apache-tomcat-7.0.57/logs/catalina.out

2). 查询日志文件尾部的50行记录
tail -50 /usr/local/apache-tomcat-7.0.57/logs/catalina.out
# 只要Tomcat在启动的过程中，日志输出没有报错，基本可以判定Tomcat启动成功了。
```

2. 查询系统进程_关闭防火墙&放开端口

   ```shell
   ps -ef|grep tomcat
   # 方法一(不推荐)
   systemctl stop firewalld	# 关闭防火墙
   # 方法二
   systemctl start firewalld	# 先开启系统防火墙
   firewall-cmd --zone=public --add-port=8080/tcp --permanent	# 再开放8080端口号
   firewall-cmd --reload	# 重新加载防火墙
   
   # 停止Tomcat
   	# 方法一: 在Tomcat目录下bin目录下
   sh shutdown.sh
   ./shutdown.sh
   	# 方法二(不推荐)
   ps -ef|grep tomcat
   kill -9 *id*
   ```

#### 4~防火墙操作

| 操作                         | 指令                                                         | 备注                 |
| ---------------------------- | ------------------------------------------------------------ | -------------------- |
| 查看防火墙状态               | systemctl status firewalld / firewall-cmd --state            |                      |
| 暂时关闭防火墙               | systemctl stop firewalld                                     | 本次服务内关闭       |
| 永久关闭防火墙(禁用开机自启) | systemctl disable firewalld                                  | ==下次启动,才生效==  |
| 暂时开启防火墙               | systemctl start firewalld                                    |                      |
| 永久开启防火墙(启用开机自启) | systemctl enable firewalld                                   | ==下次启动,才生效==  |
| 开放指定端口                 | firewall-cmd --zone=public --add-port=8080/tcp --permanent   | ==需要重新加载生效== |
| 关闭指定端口                 | firewall-cmd --zone=public --remove-port=8080/tcp --permanent | ==需要重新加载生效== |
| 立即生效(重新加载)           | firewall-cmd --reload                                        |                      |
| 查看开放端口                 | firewall-cmd --zone=public --list-ports                      |                      |

> 注意：
> 	A. systemctl是管理Linux中服务的命令，可以对服务进行启动、停止、重启、查看状态等操作
> 	B. firewall-cmd是Linux中专门用于控制防火墙的命令
>	C. 为了保证系统安全，服务器的防火墙不建议关闭

### 5. git安装

```shell
yum list git	# 列出git安装包
yum install git	# 在线安装
git --version 
```

### 6. Maven安装

```shell
rz 		# 上传文件
tar -zxvf apache-maven-3.5.4-bin.tar.gz -C /usr/local

vim /etc/profile
# 修改配置文件，进入到命令模式，按G切换到最后一行，按a/i/o进入插入模式，然后在最后加入如下内容 :
export MAVEN_HOME=/usr/local/apache-maven-3.5.4
export PATH=$JAVA_HOME/bin:$MAVEN_HOME/bin:$PATH

source /etc/profile
```

**修改maven的settings.xml配置文件,配置本地仓库地址**

```shell
cd /usr/local/apache-maven-3.5.4/conf
vim settings.xml
# 在其中增加如下配置,配置本地仓库地址:
<localRepository>/usr/local/repo</localRepository>


# 配置阿里云镜像(可选,暂时不配置)
<mirror> 
    <id>alimaven</id> 
    <mirrorOf>central</mirrorOf> 
    <name>aliyun maven</name> 
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
</mirror> 
```



### 7. Redis安装

```shell
rz
tar -zxvf redisXXX -C /usr/local
yum install gcc-c++		# 安装依赖环境
cd /usr/local/redisXXX		# 进入目录
make		# 进行编译
cd src		# 进入src目录
make install	# 安装
redis-server/ctrl + C	#启动服务/停止服务
redis-cli		#连接本地Redis服务

vim redis.conf		# 打开配置文件
/daemonize	# 将no改为yes (允许后台运行)
# requirepass foobared	# 设置redis服务密码, 默认为注释状态, foobared为密码
bind 1270.0.01		# 设置允许客户端远程连接redis服务, 将此行注释掉, 默认不允许远程连接

# 进入Redis安装目录
cd /usr/local/redis-4.0.0
# 启动Redis服务，指定使用的配置文件
./src/redis-server ./redis.conf

echo "/usr/local/bin/redis-server /etc/redis/redis.conf &" >> /etc/rc.local  	# 开机启动
```

> 文件说明
>
> /usr/local/redis-4.0.0/src/redis-server：Redis服务启动脚本
>
> /usr/local/redis-4.0.0/src/redis-cli：Redis客户端脚本
>
> /usr/local/redis-4.0.0/redis.conf：Redis配置文件

==集群搭建见/redis==

1. RDB持久化配置: 修改redis.conf文件

   ```properties
   # 900秒内，如果至少有1个key被修改，则执行bgsave ， 如果是save "" 则表示禁用RDB
   save 900 1  
   save 300 10  
   save 60 10000 
   
   # 其他配置
   # 是否压缩 ,建议不开启，压缩也会消耗cpu，磁盘的话不值钱
   rdbcompression no
   
   # RDB文件名称
   dbfilename dump.rdb  
   
   # 文件保存的路径目录
   dir ./ 
   ```

2. AOF持久化: 修改redis.conf文件

   ```properties
   # 是否开启AOF功能，默认是no
   appendonly yes
   # AOF文件的名称
   appendfilename "appendonly.aof"
   
   # 更改AOF命令记录频率
   # 表示每执行一次写命令，立即记录到AOF文件
   appendfsync always 
   # 写命令执行完先放入AOF缓冲区，然后表示每隔1秒将缓冲区数据写到AOF文件，是默认方案, 牺牲一定安全性, 最多丢失1秒z
   appendfsync everysec 
   # 写命令执行完先放入AOF缓冲区，由操作系统决定何时将缓冲区内容写回磁盘
   appendfsync no
   ```

   

### 8. docker安装

1. 先卸载

   ```shell
   yum remove docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-selinux \
                     docker-engine-selinux \
                     docker-engine \
                     docker-ce
   ```

2. 安装docker

   安装yum工具

   ```shell
   yum install -y yum-utils \
              device-mapper-persistent-data \
              lvm2 --skip-broken
   ```

   更新本地镜像源：

   ```shell
   # 设置docker镜像源
   yum-config-manager \
       --add-repo \
       https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
       
   sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo
   
   yum makecache fast
   ```

   docker-ce为社区免费版本。稍等片刻，docker即可安装成功:

   ```shell
   yum install -y docker-ce
   ```

   关闭防火墙:

   ```shell
   # 关闭
   systemctl stop firewalld
   # 禁止开机启动防火墙
   systemctl disable firewalld
   ```

   启动docker：

   ```shell
   systemctl start docker  # 启动docker服务
   
   systemctl stop docker  # 停止docker服务
   
   systemctl restart docker  # 重启docker服务
   
   systemctl enable docker	# 开机自启
   ```

   查看docker版本：
   
   ```shell
   docker -v
   ```

   3. 安装DockerCompose
   
   ```shell
   # 安装
   curl -L https://github.com/docker/compose/releases/download/1.23.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
   # 如果下载速度较慢，或者下载失败，可以使用docker-compose文件：
   上传到`/usr/local/bin/`目录即可
   ```

   修改文件权限
   
   ```shell
   # 修改权限
   chmod +x /usr/local/bin/docker-compose
   ```

   Base自动补全命令：
   
   ```shell
   # 补全命令
   curl -L https://raw.githubusercontent.com/docker/compose/1.29.1/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose
   # 如果这里出现错误，需要修改自己的hosts文件：
   echo "199.232.68.133 raw.githubusercontent.com" >> /etc/hosts
   ```

   4. Docker镜像仓库

      Docker官方的Docker Registry是一个基础版本的Docker镜像仓库，具备仓库管理的完整功能，但是没有图形化界面。

      搭建方式比较简单，命令如下：
   
      ```shell
      docker run -d \
          --restart=always \
          --name registry	\
          -p 5000:5000 \
          -v registry-data:/var/lib/registry \
          registry
      ```

      命令中挂载了一个数据卷registry-data到容器内的/var/lib/registry 目录，这是私有镜像库存放数据的目录。
   
      访问http://YourIp:5000/v2/_catalog 可以查看当前私有镜像服务中包含的镜像

### 9. docker安装MongoDB

```shell
docker search mongo
docker pull mongo	# 拉取镜像
docker images		# 查看镜像
mkdir -p /dockerdata/mongodb/datadb	# 创建文件夹
chmod 777 /dockerdata/mongodb/datadb	# 修改文件夹权限为可读可写可执行

#####################
docker run -d --name mongodb -v /dockerdata/mongodb/datadb:/data/db -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=2323 --privileged=true fefd78e9381a
# 参数说明
# -d 后台运行容器
# -name mongodb 运行容器名
# -v /docker-software/mongodb/datadb:/data/db 挂载目录
# -p 27017:27017 映射外部端口
# -e 用户名和密码
# --privileged=true 使得容器内的root拥有真正root权限,否则container内的root只是外部的一个普通用户权限
# fefd78e9381a 也可以使用镜像名
###################
```

```shell
docker images	# 查询镜像
docker ps -a	# 查询所有容器
docker rm b9dadcb212f2	# 删除容器
docker rmi 15a9bc7c6340	# 删除镜像
```

### 10. docker安装xxl-job

1. 创建mysql容器, 初始化xxl-job的SQL脚本

   ```shell
   docker run -p 3306:3306 --name mysql57 \
   -v /opt/mysql/conf:/etc/mysql \
   -v /opt/mysql/logs:/var/log/mysql \
   -v /opt/mysql/data:/var/lib/mysql \
   -e MYSQL_ROOT_PASSWORD=root \
   -d mysql:5.7
   ```

2. 查询镜像

   `docker search xxl-job-admin`

3. 拉取镜像

   ```shell
   docker pull xuxueli/xxl-job-admin:2.3.0
   ```

4. 创建容器

   ```shell
   docker run -e PARAMS="--spring.datasource.url=jdbc:mysql://192.168.200.130:3306/xxl_job?Unicode=true&characterEncoding=UTF-8 \  
   --spring.datasource.username=root \  
   --spring.datasource.password=root" \  
   -p 8888:8080 -v /tmp:/data/applogs \  
   --name xxl-job-admin --restart=always -d xuxueli/xxl-job-admin:2.3.0
   ```

5. 启动调度中心

   www.ip:8888/xxl-job-admin

   账号密码: "admin/123456"

### 11. docker安装Nacos

```SHELL
docker search nacos	# 查询nacos
docker pull nacos/nacos-server	# 拉取镜像
docker images		# 查看镜像

# 挂载目录
mkdir -p /dockerdata/nacos/logs/                      #新建logs目录
mkdir -p /dockerdata/nacos/init.d/   
chmod 777 /dockerdata/nacos/logs	# 修改权限
chmod 777 /dockerdata/nacos/init.d
vim /dockerdata/nacos/init.d/custom.properties        #修改配置文件如下

#### mysql新建nacos的数据库，并执行脚本 sql脚本地址如下：
#### https://github.com/alibaba/nacos/blob/master/config/src/main/resources/META-INF/nacos-db.sql
```

- **custom.properties文件如下**(更改数据库账号密码)

```properties
server.contextPath=/nacos
server.servlet.contextPath=/nacos
server.port=8848
 
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://192.168.123.100:3306/nacos_devtest_prod?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=2323
 
nacos.cmdb.dumpTaskInterval=3600
nacos.cmdb.eventTaskInterval=10
nacos.cmdb.labelTaskInterval=300
nacos.cmdb.loadDataAtStart=false
management.metrics.export.elastic.enabled=false
management.metrics.export.influx.enabled=false
server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D %{User-Agent}i
nacos.security.ignore.urls=/,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/v1/auth/login,/v1/console/health/**,/v1/cs/**,/v1/ns/**,/v1/cmdb/**,/actuator/**,/v1/console/server/**
nacos.naming.distro.taskDispatchThreadCount=1
nacos.naming.distro.taskDispatchPeriod=200
nacos.naming.distro.batchSyncKeyCount=1000
nacos.naming.distro.initDataRatio=0.9
nacos.naming.distro.syncRetryDelay=5000
nacos.naming.data.warmup=true
nacos.naming.expireInstance=true
```

- **启动容器**

  ```shell
  docker  run \
  --name nacos -d \
  -p 8848:8848 \
  --privileged=true \
  --restart=always \
  -e JVM_XMS=256m \
  -e JVM_XMX=256m \
  -e MODE=standalone \
  -e PREFER_HOST_MODE=hostname \
  -v /mydata/nacos/logs:/home/nacos/logs \
  -v /mydata/nacos/init.d/custom.properties:/home/nacos/init.d/custom.properties \
  nacos/nacos-server
  ```

### docker安装Redis



### @ 命令整理

#### 1. ls

```shell
ls -al	# 查看当前目录的所有文件及目录详细信息
ls -al /etc	# 查看/etc目录下所有文件及目录详细信息
ll	# 查看当前目录文件及目录的详细信息
```

#### 2. cat/more/tail

```shell
cat /etc/profile	#查看文档全部内容
more /etc/profile	#分页查看文档；回车：下一行；空格：下一屏；b上一屏  ；q/ctrl+C 推出
tail /etc/profile	显示末尾10行内容
tail -20 /etc/profile	显示末尾20行内容
tail -f /xxx/my.log		动态读取末尾内容并显示
```

#### 3. mkdir/touch/rm/cp/mv

```shell
mkdir -p xxx/test	# 在xxx目录下建一个子目录, 若xxx不存在, 则新建一个
touch  2.txt 3.txt 4.txt  #  一次性创建文件2.txt,3.txt,3.txt

rm -f xxx/		# 直接删除xxx下的所有文件
rm -rf xxx/		# r是递归删除, f是强制删除

cp test.txt ./demo.txt	# 将test.txt文件复制到当前目录, 并改名
cp -r xxx/ ./sss/	# 将xxx目录和目录下所有文件复制到sss目录下
cp -r xxx/* ./sss/	# 将xxx目录下所有文件复制到sss目录下

mv test.txt demo.txt	# 改名
mv test.txt xxx/	# 移动
mv test.txt xxx/demo.txt	# 移动并改名
mv xxx/ sss/	# 移动文件夹, 如果不存在就新建
```

#### 4. ==tar==

```shell
# 说明
-z: gzip ---> 压缩或解压
-c: create ---> 创建新的包文件
-x: extract ---> 从包文件中还原文件
-v: verbose ---> 显示命令的执行过程
-f: file ---> 指定包文件名
# 打包
tar -cvf hello.tar ./*	# 将当前目录下所有文件打包, 打包后文件名为hello.tar
tar -zcvf hello.tar.gz ./*	# 打包并压缩, 打包后文件名为hello.tar.gz
# 解压
tar -zxvf hello.tar.gz	# 解压并放在当前目录
tar -zxvf hello.tar.gz -C /usr/local	# 解压并放在/usr/local 目录
```

#### 5. ==vi==

```shell
yum install vim		# 安装
vim fileName	# 打开或新建文本
# 命令模式(Command mode)
gg/G		# 定位到文本第一行/定位到最后一行
dd/ndd		# 删除光标所在的数据/删除光标所在及之后的n行数据
u/ctrl + r	# 撤销操作/撤销撤销
kjhl	  # 上下左右/可以和数字结合使用
/text		# 查找text, 按n建查找下一个, 按N键查找前一个
yy/nyy		# 拷贝当前行/拷贝n行
p		# 在光标后粘贴

# 插入模式(Insert mode)
# 底行模式(Last line mode)
:wq/:q!		# 保存并退出/不保存退出
:set nu/ :set nonu	   # 显示行号/取消行号显示
:set ignorecase/ :set noignorecase	# 忽略大小写查找/不忽略大小写
:set hlsearch/ : set nohlserach		# 高亮搜索/关闭高亮
:n		# 定位到n行
```

#### 6. find/grep

```shell
find . -name "*.java"		# 在当前目录及其子目录查找.java结尾的文件
find /xxx -name "*.java"	# 在/xxx目录及其子目录下查找.java结尾的文件

grep Hello test.java		# 查找Hello字符串位置
grep hello *.java			# 查找当前目录所有.java结尾的文件中包含hello字符串的位置
```

#### 7. 后台运行程序 ==nohup==

```shell
# 后台运行 java -jar 命令，并将日志输出到hello.log文件
nohup java -jar helloworld-1.0-SNAPSHOT.jar &> hello.log &
ps -ef|grep java	# 查找进程
kill -9 45199	# 杀进程
```

#### 8.Shell脚本 ==bootStart.sh== /项目部署

1. **脚本详情见 /linux软件**

在/usr/local/目录下创建一个目录 sh(mkdir sh)，并将shell脚本上传到该目录下。或者直接在sh目录下创建一个脚本bootStart.sh，然后将bootStart.sh文件打开,内容拷贝过来即可。

2. **Linux权限**

Shell脚本要想正常的执行，还需要给Shell脚本分配执行权限。 由于linux系统是一个多用户的操作系统，并且针对每一个用户，Linux会严格的控制操作权限。

> 1). ==chmod==（英文全拼：change mode）命令是控制用户对文件的权限的命令
>
> 2). Linux中的权限分为三种 ：读(r)、写(w)、执行(x)
>
> 3). Linux文件权限分为三级 : 文件所有者（Owner）、用户组（Group）、其它用户（Other Users）
>
> 4). 只有文件的所有者和超级用户可以修改文件或目录的权限
>
> 5). 要执行Shell脚本需要有对此脚本文件的执行权限(x)，如果没有则不能执行

chmod命令可以使用八进制数来指定权限(0 - 代表无 , 1 - 执行x , 2 - 写w , 4 - 读r):

| 值   | 权限           | rwx  |
| ---- | -------------- | ---- |
| 7    | 读 + 写 + 执行 | rwx  |
| 6    | 读 + 写        | rw-  |
| 5    | 读 + 执行      | r-x  |
| 4    | 只读           | r--  |
| 3    | 写 + 执行      | -wx  |
| 2    | 只写           | -w-  |
| 1    | 只执行         | --x  |
| 0    | 无             | ---  |

```shell
chmod 777 bootStart.sh   为所有用户授予读、写、执行权限
chmod 755 bootStart.sh   为文件拥有者授予读、写、执行权限，同组用户和其他用户授予读、执行权限
chmod 210 bootStart.sh   为文件拥有者授予写权限，同组用户授予执行权限，其他用户没有任何权限
# 第1位表示文件拥有者的权限
# 第2位表示同组用户的权限
# 第3位表示其他用户的权限
```



#### 9. 关机

```shell
poweroff
shutdown -h now
```



#### & 查询进程/netstat/==ps==

```shell
A. netstat命令用来打印Linux中网络系统的状态信息，可让你得知整个Linux系统的网络情况。
netstat -p|grep 3306	#查询进程
参数说明: 

-l或--listening：显示监控中的服务器的Socket；
-n或--numeric：直接使用ip地址，而不通过域名服务器；
-p或--programs：显示正在使用Socket的程序识别码和程序名称；
-t或--tcp：显示TCP传输协议的连线状况；
-u或--udp：显示UDP传输协议的连线状况；

B. ps命令用于查看Linux中的进程数据。
# "|" 在Linux中称为管道符，可以将前一个命令的结果输出给后一个命令作为输入
ps -ef|grep tomcat	# 查看

kill -9 进程id	# 

systemctl status firewalld		#查看防火墙信息
systemctl start firewalld		#开启防火墙
systemctl stop firewalld	      #关闭防火墙
firewall-cmd --zone=public --add-port=6379/tcp --permanent	#开放端口
systemctl reload firewalld		#重启防火墙
```



