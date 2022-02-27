## CAS
compare and swap
![[Pasted image 20220226165631.png]]
```java
AtomicInteger i = new AtomicInteger();
i.incrementAndGet();    //点进去源码
```


本质的实现
JVM是一种标准, 具体的实现:
- hotspot
- openSDK

最终实现: 
cmpxcha = cas 修改变量值
`lock cmpxchg` 指令  -> (背掉)

lock指令不允许别的cpu不允许打断


## 对象在内存中的布局

Object o = new Object();


![[Pasted image 20220227101021.png]]
