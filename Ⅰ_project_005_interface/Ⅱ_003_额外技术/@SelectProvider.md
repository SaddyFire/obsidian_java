### 01 controller层
```java
{
	//根据name,age模糊查询, 采用selectProvider
	@GetMapping("/find")
	public ResponseResult findByCondition(@RequestParam(defaultValue = "1") Long current,
										  @RequestParam(defaultValue = "10") Long size,
										  @RequestParam(value = "name", required = false)String name,
										  @RequestParam(value = "age", required = false)Integer age){
		return userService.findByCondition(current,size,name,age);
	}
}
```


### 02 service层
```java
{
	@Override
	public ResponseResult findByCondition(Long current, Long size, String name, Integer age) {
		List<User> userList = userMapper.selectByCondition(size * (current-1),size,name,age);
		return ResponseResult.okResult(userList);
	}
}
```

### 03 mapper层
`@SelectProvider(type = SelectUserProvider.class, method = "selectByCondition")`
**type**  -->  对应provider对应类, **method**  -->  类中具体方法
```java
{
	//type --> 对应provider类
	//method --> provider中的方法
	@SelectProvider(type = SelectUserProvider.class, method = "selectByCondition")
	List<User> selectByCondition(@Param("index") Long index,
								 @Param("limit") Long limit,
								 @Param("name") String name,
								 @Param("age") Integer age);
}
```


### 04 provider层
**注意:**
1. 达梦数据库中, **string类型字段要拼接引号  '**
2. 如果参数是 Integer, Long类型要注意, 无法直接强转(String), 也不能用toString()方法(可能会空指针), **需要用ObjectUtils.isNotEmpty()进行非空判断**
`package com.consmation.demo.mapper.provider;`
```java
import org.apache.commons.lang3.ObjectUtils;
import org.apache.commons.lang3.StringUtils;
import org.apache.ibatis.jdbc.SQL;

import java.util.Map;

/**
 * @author SaddyFire
 * @date 2022/4/11
 * @TIME:12:09
 */
public class SelectUserProvider {

    /**
     * 入参使用@param, 则要对应使用Map接收
     * @param para
     * @return
     */
    public String selectByCondition(Map<String, Object> para){
        return new SQL(){{
            SELECT("*");
            FROM("INTERFACE1.app_user");
            if(StringUtils.isNotBlank((String) para.get("name"))){
                WHERE("name='" + para.get("name") + "'");   //此处前后要拼接 ' ,string类型
            }
            if(ObjectUtils.isNotEmpty(para.get("age"))){     //此处age为Integer, 无法强转(String) , 同时: 为了防止age为空, 此处只能用ObjectUtils
                OR().WHERE("age=" + para.get("age"));
            }
            LIMIT(para.get("index") + "," +para.get("limit"));
        }}.toString();
    }

}
```