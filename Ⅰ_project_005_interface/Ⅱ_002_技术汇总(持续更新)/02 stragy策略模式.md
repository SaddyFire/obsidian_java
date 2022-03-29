搭建顺序建议从下到上

##### 01 定义策略上下文
`package com.consmation.demo.stragy;`

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

/**
 * @author SaddyFire
 * @date 2022/3/24
 */
@Service
public class InvokeContext {

    @Autowired
    private final Map<String, InvokeStrategyService> invokeStrategyMap = new ConcurrentHashMap<>();

    public InvokeContext(Map<String, InvokeStrategyService> strategyMap){
        this.invokeStrategyMap.clear();
        strategyMap.forEach(this.invokeStrategyMap::put);
    }

    public InvokeStrategyService getResource(String type) {
        return invokeStrategyMap.get(type);
    }
}
```

##### 02 策略接口 InvokeStrategyService
`package com.consmation.demo.stragy;`

```java
import com.consmation.demo.model.vo.ResponseResult;

/**
 * @author SaddyFire
 * @date 2022/3/24
 * @TIME:18:38
 */
public interface InvokeStrategyService {
    /**
     * 分页查询
     * @param current
     * @param size
     * @return
     */
    ResponseResult invoke(Long current, Long size);
}
```


##### 03 策略实现类
`package com.consmation.demo.stragy.impl;`

此处@component()注解value要和 策略Context的值保持一致

```java
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.consmation.demo.anno.RSAEncrpyt;
import com.consmation.demo.mapper.UserMapper;
import com.consmation.demo.model.pojo.User;
import com.consmation.demo.model.vo.PageResult;
import com.consmation.demo.model.vo.ResponseResult;
import com.consmation.demo.stragy.InvokeStrategyService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * @author SaddyFire
 * @date 2022/3/24
 * @TIME:18:43
 */
@Component("type1")
@Slf4j
public class InvokeStrategyType1 implements InvokeStrategyService {

    @Autowired
    private UserMapper userMapper;

    @Override
    @RSAEncrpyt
    public ResponseResult invoke(Long current, Long size) {
        IPage<User> iPage = new Page<>(current, size);
        LambdaQueryWrapper<User> qw = new LambdaQueryWrapper<>();
        IPage<User> userIPage = userMapper.selectPage(iPage, qw);
        PageResult pageResult = PageResult.builder()
                //总记录数
                .total(userIPage.getTotal())
                //页大小
                .size(userIPage.getSize())
                //总页数
                .current(userIPage.getCurrent())
                //列表
                .records(userIPage.getRecords())
                .build();

        log.info("type1执行");
        return ResponseResult.okResult(pageResult);
    }
}
```

##### 04 调用层
(此处在controller中调用)
```java
    @Autowired
    private InvokeContext invokeContext;

    //根据传参不同策略至不同实现类
    @GetMapping("/invoke")
    public ResponseResult getData(@RequestParam(defaultValue = "1") Long current,
                     @RequestParam(defaultValue = "10") Long size,
                     String type){
        InvokeStrategyService invokeServiceImpl = null;
        try {
            //通过策略上下文拿到策略实现类
            invokeServiceImpl = invokeContext.getResource(type);
        } catch (Exception e) {
            throw new CustomException(AppHttpCodeEnum.PARAM_REQUIRE);
        }
        return invokeServiceImpl.invoke(current, size);
    }
```
