此处注意使用了hutool包对密钥进行md5加密
##### 01 META-INF包指定自动装配类路径
resources -> META-INF包 -> spring.factories

例: 
```factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.consmation.demo.autoconfig.InterfaceAutoConfiguration
```

##### 02 新增自动装配类
`package com.consmation.demo.autoconfig;`
将yml变量写入注解中, 并且将OssTemplate写入bean
```java
import com.consmation.demo.autoconfig.properties.InterfaceProperties;
import com.consmation.demo.autoconfig.properties.OssProperties;
import com.consmation.demo.autoconfig.template.OssTemplate;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;

/**
 * @author SaddyFire
 * @date 2022/3/28
 * @TIME:13:39
 */
@EnableConfigurationProperties({
        OssProperties.class,
        InterfaceProperties.class
})
public class InterfaceAutoConfiguration {

    @Bean
    public OssTemplate ossTemplate(OssProperties properties) {
        return new OssTemplate(properties);
    }

}
```

##### 03 yml变量类OssProperties
`package com.consmation.demo.autoconfig.properties;`
此处要定义prefix
```java
import lombok.Data;  
import org.springframework.boot.context.properties.ConfigurationProperties;  
  
/**  
 * @author SaddyFire  
 * @date 2022/3/28  
 * @TIME:13:32  
 */  
@Data  
@ConfigurationProperties(prefix="interface.oss")  
public class OssProperties {  
  
 private String accessKey;  
 private String secret;  
 private String bucketName;  
 private String url; //域名  
 private String endpoint;  
  
}
```

##### 04 模板类OssTemplate
`package com.consmation.demo.autoconfig.template;`

此处将yml的 账号密码进行md5 加密后对比, 真正的账号密码在Constant类中
```java
import cn.hutool.crypto.SecureUtil;
import com.aliyun.oss.ClientException;
import com.aliyun.oss.OSS;
import com.aliyun.oss.OSSClientBuilder;
import com.aliyun.oss.OSSException;
import com.consmation.demo.autoconfig.properties.OssProperties;
import com.consmation.demo.exception.CustomException;
import com.consmation.demo.model.enums.AppHttpCodeEnum;
import com.consmation.demo.utils.Constants;

import java.io.InputStream;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.UUID;

/**
 * @author SaddyFire
 * @date 2022/3/28
 * @TIME:13:45
 */
public class OssTemplate {

    private OssProperties ossProperties;

    public OssTemplate(OssProperties ossProperties) {
        this.ossProperties = ossProperties;
    }

    /**
     *
     * @param fileName 文件名
     * @param is 输入流
     * @return
     */
    public String upload(String fileName, InputStream is){
        //属性提取
        String endpoint = null;
        String accessKeyId = null;
        String secreAccessKey = null;
        String bucketName = null;
        //参数判断
        try {
            endpoint = ossProperties.getEndpoint();
            accessKeyId = ossProperties.getAccessKey();
            secreAccessKey = ossProperties.getSecret();
            bucketName = ossProperties.getBucketName();
        } catch (Exception e) {
            throw new CustomException(AppHttpCodeEnum.PARAM_REQUIRE);
        }

        //密码校验(小写32位)
        if (!SecureUtil.md5(Constants.aliAccessKeyId).equals(accessKeyId) || !SecureUtil.md5(Constants.aliSecret).equals(secreAccessKey)) {
            throw new CustomException(AppHttpCodeEnum.PARAM_INVALID);
        }

        //封装oos路径
        String storePath = new SimpleDateFormat("yyyy/MM/dd").format(new Date()) + "/" + UUID.randomUUID() + fileName.substring(fileName.lastIndexOf("."));
        System.out.println(storePath);

        // 创建OSSClient实例。
        OSS ossClient = new OSSClientBuilder().build(endpoint, Constants.aliAccessKeyId, Constants.aliSecret);
        try {
            ossClient.putObject(bucketName, storePath, is);
        } catch (OSSException oe) {
            System.out.println("Caught an OSSException, which means your request made it to OSS, "
                    + "but was rejected with an error response for some reason.");
            System.out.println("Error Message:" + oe.getErrorMessage());
            System.out.println("Error Code:" + oe.getErrorCode());
            System.out.println("Request ID:" + oe.getRequestId());
            System.out.println("Host ID:" + oe.getHostId());
            throw new CustomException(AppHttpCodeEnum.SERVER_ERROR);
        } catch (ClientException ce) {
            System.out.println("Caught an ClientException, which means the client encountered "
                    + "a serious internal problem while trying to communicate with OSS, "
                    + "such as not being able to access the network.");
            System.out.println("Error Message:" + ce.getMessage());
            throw new CustomException(AppHttpCodeEnum.SERVER_ERROR);
        } finally {
            String url = ossProperties.getUrl() + storePath;
            if (ossClient != null) {
                ossClient.shutdown();
            }
            return url;
        }
    }
}
```

##### 05 application.yml
此处accessKey, secret 均是md5密文
```yml
interface:
  oss:
    accessKey: b9f0431fac535ec7117e5ff5113ad3df
    secret: 1257303cee61bb377fc5f490d733f399
    endpoint: oss-cn-hangzhou.aliyuncs.com
    bucketName: invoke-test
    url: https://saddyfire-test1.oss-cn-hangzhou.aliyuncs.com/
```

##### 06 OosTemplate文件上传
- controller层
```java
    //oos上传文件
    @PostMapping("/uploadoss")
    public ResponseResult uploadOss(MultipartFile multipartFile){
        return ossService.upload(multipartFile);
    }
```

- service层
```java
import com.consmation.demo.autoconfig.template.OssTemplate;
import com.consmation.demo.exception.CustomException;
import com.consmation.demo.model.enums.AppHttpCodeEnum;
import com.consmation.demo.model.vo.ResponseResult;
import com.consmation.demo.service.OssService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;
import org.springframework.web.multipart.MaxUploadSizeExceededException;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.io.InputStream;

/**
 * @author SaddyFire
 * @date 2022/3/28
 * @TIME:14:52
 */
@Service
public class OssServiceImpl implements OssService {

    @Autowired
    private OssTemplate ossTemplate;

    @Override
    public ResponseResult upload(MultipartFile multipartFile) {
        String originalFilename;
        InputStream inputStream;
        try {
            //参数校验
            originalFilename = multipartFile.getOriginalFilename();
            inputStream = multipartFile.getInputStream();
        }catch (MaxUploadSizeExceededException maxUploadSizeExceededException){
            throw new CustomException(AppHttpCodeEnum.PICTURE_MAX);
        } catch (IOException e) {
            throw new CustomException(AppHttpCodeEnum.PARAM_INVALID);
        }
        String url = ossTemplate.upload(originalFilename, inputStream);
        //url校验
        if (StringUtils.isEmpty(url)) {
            throw new CustomException(AppHttpCodeEnum.SERVER_ERROR);
        }

        return ResponseResult.okResult(url);
    }
}
```

##### 07 测试类
```java
import com.aliyun.oss.ClientException;
import com.aliyun.oss.OSS;
import com.aliyun.oss.OSSClientBuilder;
import com.aliyun.oss.OSSException;
import com.consmation.demo.InterfaceApplication;
import com.consmation.demo.autoconfig.template.OssTemplate;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.io.ByteArrayInputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;

/**
 * @author SaddyFire
 * @date 2022/3/28
 * @TIME:13:47
 */
@SpringBootTest(classes = InterfaceApplication.class)
@RunWith(SpringRunner.class)
public class OssTest {


    // Endpoint以华东1（杭州）为例，其它Region请按实际情况填写。
    String endpoint = "oss-cn-hangzhou.aliyuncs.com";
    // 阿里云账号AccessKey拥有所有API的访问权限，风险很高。强烈建议您创建并使用RAM用户进行API访问或日常运维，请登录RAM控制台创建RAM用户。
    String accessKeyId = "LTAI5tBzXbi8X2TRPx8xfoEW";
    String accessKeySecret = "A8tjhHJyQCookP4MJNdMIUgbDpn91S";
    // 填写Bucket名称，例如examplebucket。
    String bucketName = "invoke-test";

    //创建存储空间
    @Test
    public void test1(){

        // 创建OSSClient实例。
        OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);

        try {
            // 创建存储空间。
            ossClient.createBucket(bucketName);

        } catch (OSSException oe) {
            System.out.println("Caught an OSSException, which means your request made it to OSS, "
                    + "but was rejected with an error response for some reason.");
            System.out.println("Error Message:" + oe.getErrorMessage());
            System.out.println("Error Code:" + oe.getErrorCode());
            System.out.println("Request ID:" + oe.getRequestId());
            System.out.println("Host ID:" + oe.getHostId());
        } catch (ClientException ce) {
            System.out.println("Caught an ClientException, which means the client encountered "
                    + "a serious internal problem while trying to communicate with OSS, "
                    + "such as not being able to access the network.");
            System.out.println("Error Message:" + ce.getMessage());
        } finally {
            if (ossClient != null) {
                ossClient.shutdown();
            }
        }
    }

    //上传文件
    @Test
    public void test(){
        // 填写Object完整路径，例如exampledir/exampleobject.txt。Object完整路径中不能包含Bucket名称。
        String objectName = "exampledir/exampleobject.txt";

        // 创建OSSClient实例。
        OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);

        try {
            String content = "Hello OSS";
            ossClient.putObject(bucketName, objectName, new ByteArrayInputStream(content.getBytes()));
        } catch (OSSException oe) {
            System.out.println("Caught an OSSException, which means your request made it to OSS, "
                    + "but was rejected with an error response for some reason.");
            System.out.println("Error Message:" + oe.getErrorMessage());
            System.out.println("Error Code:" + oe.getErrorCode());
            System.out.println("Request ID:" + oe.getRequestId());
            System.out.println("Host ID:" + oe.getHostId());
        } catch (ClientException ce) {
            System.out.println("Caught an ClientException, which means the client encountered "
                    + "a serious internal problem while trying to communicate with OSS, "
                    + "such as not being able to access the network.");
            System.out.println("Error Message:" + ce.getMessage());
        } finally {
            if (ossClient != null) {
                ossClient.shutdown();
            }
        }
    }

    @Autowired
    private OssTemplate ossTemplate;

    @Test
    public void test02() throws FileNotFoundException {
        File file = new File("E:\\【SaddyFire】\\【SaddyFire】picture\\必应壁纸\\GoldenGinkgo_ZH-CN8507013452_1920x1080.jpg");
        ossTemplate.upload(file.getName(), new FileInputStream(file));
    }
    
}
```

