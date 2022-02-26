##  00 FastDFS简介
FastDFS是分布式文件系统。使用 FastDFS很容易搭建一套高性能的文件服务器集群提供文件上传. 下载等服务。

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

- 依赖
```xml
<dependency>
    <groupId>com.github.tobato</groupId>
    <artifactId>fastdfs-client</artifactId>
    <version>1.26.7</version>
    <exclusions>
        <exclusion>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

- application.yml

注意缩进
```yml
# 分布式文件系统FDFS配置
fdfs:
  so-timeout: 1500
  connect-timeout: 600
  #缩略图生成参数
  thumb-image:
    width: 150
    height: 150
  #TrackerList参数,支持多个
  tracker-list: 192.168.136.160:22122
  web-server-url: http://192.168.136.160:8888/

#配置请求文件和请求体 , 设置文件大小
spring:
  servlet:
    multipart:
      max-file-size: 30MB
      max-request-size: 30MB
```

## 02 测试代码
- 核心代码
```java
...
	//1. 指定文件
	File file = new File("D:\\1.jpg");
	//2. 文件上传
	StorePath path = client.uploadFile(new FileInputStream(file),
			file.length(), "jpg", null);
	//3. 拼接访问路径
	String url = webServer.getWebServerUrl() + path.getFullPath();
...
```

- tset代码

```java
import com.github.tobato.fastdfs.domain.conn.FdfsWebServer;
import com.github.tobato.fastdfs.domain.fdfs.StorePath;
import com.github.tobato.fastdfs.service.FastFileStorageClient;
import com.tanhua.server.AppServerApplication;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;

@RunWith(SpringRunner.class)
@SpringBootTest(classes = AppServerApplication.class)
public class TestFastDFS {

    //从调度服务器获取，一个目标存储服务器，上传
    @Autowired
    private FastFileStorageClient client;

    @Autowired
    private FdfsWebServer webServer;// 获取存储服务器的请求URL

    @Test
    public void testFileUpdate() throws FileNotFoundException {
        //1. 指定文件
        File file = new File("D:\\1.jpg");
        //2. 文件上传
        StorePath path = client.uploadFile(new FileInputStream(file),
                file.length(), "jpg", null);
        //3. 拼接访问路径
        String url = webServer.getWebServerUrl() + path.getFullPath();
    }
}
```


## 03 tanhua业务代码

service层
```java
import com.github.tobato.fastdfs.domain.conn.FdfsWebServer;
import com.github.tobato.fastdfs.domain.fdfs.StorePath;
import com.github.tobato.fastdfs.service.FastFileStorageClient;
import com.tanhua.autoconfig.template.OssTemplate;
import com.tanhua.dubbo.api.VideoApi;
import com.tanhua.model.mongo.Video;
import com.tanhua.server.interceptor.UserHolder;
import com.tanhua.server.service.SmallVideosService;
import org.apache.dubbo.config.annotation.DubboReference;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.FileInputStream;
import java.io.IOException;

/**
 * @author SaddyFire
 * @date 2022/1/2
 * @TIME:19:10
 */
@Service
public class SmallVideosServiceImpl implements SmallVideosService {

    @Autowired
    private OssTemplate ossTemplate;

	//上传文件
    @Autowired
    private FastFileStorageClient fastFileStorageClient;

	//获取url
    @Autowired
    private FdfsWebServer webServer;

    @DubboReference
    private VideoApi videoApi;

    @Override
    public void publishSmallVideos(MultipartFile videoThumbnail, MultipartFile videoFile) throws IOException {
        Long currentUserId = UserHolder.getUserId();
        //非空判断
        if(videoThumbnail.isEmpty() || videoFile.isEmpty()){
            throw new RuntimeException("参数非法");
        }
        //上传图片
        String pictureUrl = ossTemplate.upload(videoThumbnail.getOriginalFilename(), videoThumbnail.getInputStream());
        /**
		 * 上传视频
		*/ 
        String filename = videoFile.getOriginalFilename();
		//提取扩展名
        filename = filename.substring(filename.lastIndexOf(".")+1);
		//上传文件
        StorePath storePath = fastFileStorageClient.uploadFile(videoFile.getInputStream(), videoFile.getSize(), filename, null);
		//拼接url
        String videoUrl = webServer.getWebServerUrl() + storePath.getFullPath();

        Video video = new Video();
        video.setUserId(currentUserId);
        video.setVideoUrl(videoUrl);
        video.setPicUrl(pictureUrl);
        Boolean result = videoApi.saveVideos(video);

        if(!result){
            throw new RuntimeException("上传失败");
        }
    }
}

```

