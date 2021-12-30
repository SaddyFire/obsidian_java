## 01 依赖

*在model模块引入依赖*

```xml
<!--bean copy的工具类-->

<dependency>
	<groupId>com.github.dozermapper</groupId>
	<artifactId>dozer-core</artifactId>
	<version>6.5.0</version>
</dependency>

```
<br/>


## 02 DozerConfig 配置类

*com.tanhua.server.config*

```java
import com.github.dozermapper.core.DozerBeanMapper;
import com.github.dozermapper.core.DozerBeanMapperBuilder;
import com.github.dozermapper.core.Mapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DozerConfig {

    @Bean
    public Mapper mapper() {
        DozerBeanMapper mapper = (DozerBeanMapper) DozerBeanMapperBuilder.buildDefault();
        return mapper;
    }
}
```
<br/>

## 03 深克隆的使用
*任意包*

1. 自动装配 Mapper **(import com.github.dozermapper.core.Mapper)**

```java
public class *** {

	@Autowired  
	private Mapper mapper;

	@Override  
	public UserInfoVo getUserInfo(Long id) {  
		UserInfo userinfo = userInfoApi.getUserInfo(id);  
		//深克隆  
		UserInfoVo userInfoVo = mapper.map(userinfo, UserInfoVo.class);  

		return userInfoVo;  
	}
} 
```


