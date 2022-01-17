## 1. 软件安装
```shell
docker pull rabbitmq:3.8-management
```
## 2.创建mq容器
```shell
docker run \
 -e RABBITMQ_DEFAULT_USER=root \
 -e RABBITMQ_DEFAULT_PASS=root \
 -v mq-plugins:/plugins \
 --name mq \
 --hostname mq \
 -p 15672:15672 \
 -p 5672:5672 \
 -d \
 rabbitmq:3.8-management
```
访问控制台: [http://139.224.230.186:15672/#/](http://139.224.230.186:15672/#/ "http://139.224.230.186:15672/#/")