### 01 依赖

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>

	<dependency>
		<groupId>com.github.xiaoymin</groupId>
		<artifactId>knife4j-spring-boot-starter</artifactId>
		<version>2.0.2</version>
	</dependency>
	<!--原生页面-->
	<dependency>
		<groupId>io.swagger</groupId>
		<artifactId>swagger-annotations</artifactId>
		<version>1.5.20</version>
	</dependency>
	<dependency>
		<groupId>io.springfox</groupId>
		<artifactId>springfox-swagger2</artifactId>
		<version>2.9.2</version>
	</dependency>
	<dependency>
		<groupId>io.springfox</groupId>
		<artifactId>springfox-swagger-ui</artifactId>
		<version>2.9.2</version>
	</dependency>
</dependencies>
```

### 02 yml
```yml
server:
  port: 4522
spring:
  application:
    name: swagger
```


### 03 config
`package com.consmation.common.swagger.config;`
```java
package com.consmation.common.swagger.config;

import com.github.xiaoymin.knife4j.spring.annotations.EnableKnife4j;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * @author SaddyFire
 * @date 2021/12/8
 */
@EnableKnife4j
@EnableSwagger2 //原生
@Configuration
public class SwaggerConfig {
    @Bean
    public Docket createRestApi() {
        // 文档类型
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //标题名
                .title("浙江监管服务平台")
                //版本号
                .version("1.0")
                //api描述
                .description("浙江监管服务平台接口")
                .build();
    }
}
```

### 04 controller_api
`package com.consmation.common.swagger.controller_api;`
```java
import com.consmation.common.swagger.model.PrjDetailInfoDto;
import com.consmation.common.swagger.model.PrjDetailInfoPageResult;
import com.consmation.common.swagger.model.ResponseResult;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestParam;

/**
 * @author SaddyFire
 * @date 2022/4/15
 * @TIME:9:48
 */
@Api(description = "施工信息")
public interface ProjectControllerApi {

    /**
     * 施工许可详情获取
     *
     * @param prjDetailInfoDto
     * @param current
     * @param size
     * @return
     */
    @ApiOperation("获取施工许可详情")
    @PostMapping("/project/detail")
    ResponseResult<PrjDetailInfoPageResult> getPrjDetailInfo(@RequestBody PrjDetailInfoDto prjDetailInfoDto,
                                                             @RequestParam(defaultValue = "1") Long current,
                                                             @RequestParam(defaultValue = "10") Long size);

}

```

### 05 controller
`package com.consmation.common.swagger.controller;`
```java
package com.consmation.common.swagger.controller;

import com.consmation.common.swagger.controller_api.ProjectControllerApi;
import com.consmation.common.swagger.model.PrjDetailInfoDto;
import com.consmation.common.swagger.model.PrjDetailInfoPageResult;
import com.consmation.common.swagger.model.ResponseResult;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author SaddyFire
 * @date 2022/4/15
 * @TIME:11:43
 */
@RestController
@RequestMapping("/project")
public class ProjectController implements ProjectControllerApi {


    @Override
    public ResponseResult<PrjDetailInfoPageResult> getPrjDetailInfo(PrjDetailInfoDto prjDetailInfoDto, Long current, Long size) {
        return null;
    }
}
```

### 06 实体类
`package com.consmation.common.swagger.model;`
```java
package com.consmation.common.swagger.model;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

import java.io.Serializable;

/**
 * @author SaddyFire
 * @date 2022/4/3
 */
@ApiModel("响应实体")
@Data
public class ResponseResult<T> implements Serializable {

    @ApiModelProperty("响应码")
    private Integer code;
    @ApiModelProperty("信息")
    private String message;
    @ApiModelProperty("数据")
    private T data;

}
```

`package com.consmation.common.swagger.model;`
```java
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Builder;
import lombok.Data;

import java.io.Serializable;
import java.util.Collections;
import java.util.List;

/**
 * @author SaddyFire
 * @date 2022/4/14
 * @TIME:16:48
 */
@Data
@Builder
@ApiModel("施工许可详情分页结果集")
public class PrjDetailInfoPageResult implements Serializable {

    @ApiModelProperty("总记录数")
    private Long total = 0L;
    @ApiModelProperty("页大小")
    private Long size = 10L;
    @ApiModelProperty("当前页")
    private Long current = 0L;
    @ApiModelProperty("数据")
    private List<PrjDetailInfoVo> records = Collections.emptyList();
    @ApiModelProperty("总面积")
    private String sumArea;
    @ApiModelProperty("总金额")
    private String sumMoney;
}
```