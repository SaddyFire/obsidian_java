1. 拉取镜像(以4.4版本为例)

docker pull mongo:4.4

2. 创建容器

docker run --name mongo -p 27017:27017 -v ~/data/mongodata:/data -d mongo:4.4