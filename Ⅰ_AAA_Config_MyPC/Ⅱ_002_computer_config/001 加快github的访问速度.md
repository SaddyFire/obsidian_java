#  一、加快github的访问速度



## 加快github的访问速度

**第一步。** 打开域名解析网址：https://www.ipaddress.com/ 打开之后，分别搜索以下3个域名：

第一个：github.com

第二个： assets-cdn.github.com

第三个：github.global.ssl.fastly.net

**第二步、配置静态域名映射。**

windows操作系统为例，我们在C:\\Windows\\System32\\drivers\\etc\\hosts文件里加上上边我们查到的域名映射： 注意将ip改成自己查到的。

```java
192.30.253.112 github.com
151.101.72.133 assets-cdn.github.com
151.101.193.194 github.global.ssl.fastly.net` 

```

配置好之后保存退出。然后刷新一下DNS缓存，在命令行中输入以下命令刷新域名：

```yml
ipconfig /flushdns
```
