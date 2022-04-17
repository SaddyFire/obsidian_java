### 01 所需文件
1. bat可执行脚本文件(可省略)
2. winsw.exe
3. winsw.xml

### 02 bat文件
#### 2.1 一键安装.bat
```shell
@echo off

echo -----------------------
echo saddyfire 666 一键安装程序
echo -----------------------

%~d0

cd %~dp0

echo saddyfire安装服务中
winsw install
echo saddyfire安装完成

pause

```

#### 2.2 一键启动
```shell
@echo off

echo -----------------------
echo saddyfire 666 一键启动程序
echo ----------------------- 

net start Sentinel

pause

```
#### 2.3 一键关闭
```shell
@echo off

echo -----------------------
echo saddyfire 666 一键关闭
echo -----------------------

:: 此处是服务名
net stop Sentinel

pause

```
#### 2.4 一键卸载
```shell
@echo off

echo -----------------------
echo saddyfire 666 一键卸载程序
echo -----------------------

%~d0

cd %~dp0

echo Nacos卸载服务中
winsw uninstall
echo Nacos卸载完成

pause

```


### 03 winsw.xml
**注意端口设置**
```xml
<service>
  <!-- 指定在Windows系统内部使用的识别服务的ID。在系统中安装的所有服务中，这必须是唯一的，它应该完全由字母数字字符组成 -->
  <id>Sentinel</id>
   <!-- 服务的简短名称，它可以包含空格和其他字符。尽量简短，就像“id”一样，在系统的所有服务名称中，也要保持唯一 -->
  <name>Sentinel</name>
  <!-- 该服务可读描述。当选中该服务时，它将显示在Windows服务管理器中 -->
  <description>Sentinel</description>
  <!-- 环境变量设置 -->
  <env name="JAVA_HOME" value="%JAVA_HOME%"/>
  <!-- 启动方式: 开机自启 -->
  <startmode>Automatic</startmode>
  <!-- 可执行文件传递的参数 -->
  <arguments>-Xrs -Xmx256m -jar "%BASE%\sentinel-dashboard-1.8.1.jar"  --server.port=9999</arguments>
  <!-- 该元素指定要启动的可执行文件 -->
  <executable>java</executable>
  <!-- 日志输出位置 -->
  <logmode>rotate</logmode>
</service>
```