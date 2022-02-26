## CAS
compare and swap
![[Pasted image 20220226165631.png]]
```java
AtomicInteger i = new AtomicInteger();
i.incrementAndGet();    //点进去源码
```


本质的实现
JVM的标准
- hotspot
- openSDK
最终实现: 
cmpxcha = cas 修改变量值
`lock cmpxchg` 指令  -> (背掉)

