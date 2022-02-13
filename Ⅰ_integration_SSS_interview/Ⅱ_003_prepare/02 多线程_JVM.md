##### 四大线程池

newSingleThreadExecutor
newFixedThreadPool
newCachedThreadPool
newScheduledThreadPool

##### 线程池七大参数
- 核心线程数
- 最大线程数
- 临时线程最大存活时间
- 时间单位
- 等待队列
- 创建线程工厂
- 任务拒绝策略

##### JMM (Java Memory Module)


##### MESI缓存一致性


##### synchronized 和 volatile 的区别
synchronized 表示只有一个线程可以获取作用对象的锁，执行代码，阻塞其他线程。
volatile 表示变量在 CPU 的寄存器中是不确定的，必须从主存中读取。保证多线程环境下变量的可见性；禁止指令重排序。

