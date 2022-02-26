## 01 随机生成id (idea的方法)

### mybatisPlus的方法:

IdWorker.getId();

![[8638b0a2c2a5408b8124074090675426.png]]

## 02 LocalDateTime格式的接收

```java
@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") LocalDateTime beginTime
```

![[de82f39f74e6485b98396353158f4d9a.png]]