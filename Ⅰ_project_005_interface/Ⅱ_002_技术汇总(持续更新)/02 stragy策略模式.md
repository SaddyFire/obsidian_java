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

