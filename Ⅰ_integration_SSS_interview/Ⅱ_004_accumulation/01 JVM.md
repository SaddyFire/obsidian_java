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

![[Pasted image 20220227101146.png]]

![[Pasted image 20220227102333.png]]

PC: **程序计数器**存指令的地方
Registers: **寄存器**存数据的地方
ALU: **数学逻辑单元**计算的地方

运算单元访问cpu的时间为1, 访问内存的时间为100, 中间加缓存
![[Pasted image 20220227102614.png]]
![[Pasted image 20220227103230.png]]
cache line : 缓存行 64 bytes(记住)
缓存一致性协议: 当一个cpu更新数据, 要通知别的cpu   mesi(inter的缓存一致性协议)


