## 01 服务搭建

```shell
#进入目录
cd /root/docker-file/fastdfs/
#启动
docker-compose up -d
#查看容器
docker ps -a
#FastDFS占用虚拟机资源较多，如果启动时发现Tracker或者Storage没有正常启动，
#使用如下命令 重启即可
#docker-compose restart
```

- 设置开机自启

```shell
docker update --restart=always storage

docker update --restart=always tracker
```

>FastDFS**调度服务器**地址：192.168.136.160:22122 FastDFS**存储服务器**地址：[http://192.168.136.160:8888/](http://192.168.136.160:8888/)
