



---



## day12 字符流

* windows默认使用码表为: GBK, 一个字符两个字节

* idea和以后工作默认使用Unicode的UTF-8编码格式, 一个中文三个字节

* 想要进行拷贝,一律使用字节流或者字节缓冲流

* 想要把文本文件中的数据读到内存中, 使用字符输出流.

  想要把内存中的数据写到文本文件中, 使用字符输出流

- GBK码表一个中文两个字节, UTF-8编码格式一个中文3个字节

#### 1.1 `FileWriter`: 字符写入流

#### 1.2 `FileReader`: 字符读取流

```java
public class Before01 {
    public static void main(String[] args) throws IOException {
        //创建对象
        FileWriter fw = new FileWriter("E:\\000AAB_class_heima\\Java-se-advance\\day12_FileWriter\\before\\a.txt");
        fw.write("黑马程序员");  //写出数据
        fw.flush(); //刷新流
        fw.close(); //关闭流

        FileReader fr = new FileReader("E:\\000AAB_class_heima\\Java-se-advance\\day12_FileWriter\\before\\a.txt");
        int len;    //表示读取返回数组的长度
        char[] chars = new char[1024];
        while ((len = fr.read(chars)) != -1){   //read(chars) 返回的是读取到的数组长度
            System.out.println(new String(chars,0,len));    //new String(chars,offset,len)  (该方法可以截取数组创建字符串对象)
        }
        fr.close();

    }
}
```



#### 2.1 `BufferedReader`: 字符读取缓冲流

#### 2.2 `BufferedWriter`: 字符写入缓冲流

bw.newLine(): 换行(特殊方法)

bf.readLine(): 读一整行数据

```java
public class Before03 {
    public static void main(String[] args) throws IOException, FileNotFoundException {
        /*char [] chars = new char[1024];
        int len;
        while ((len = br.read(chars)) != -1){
            System.out.println(new String(chars,0,len));
        }*/

        BufferedReader br = new BufferedReader(new FileReader("E:\\000AAB_class_heima\\Java-se-advance\\day12_FileWriter\\before\\Before02.txt"));
        BufferedWriter bw = new BufferedWriter(new FileWriter("E:\\000AAB_class_heima\\Java-se-advance\\day12_FileWriter\\before\\Before02拷贝.txt"));
        //采用BufferedWriter 和 BufferedReader特殊方法读取和写入
        String line;
        while ((line = br.readLine()) != null){
            System.out.println(line);
            bw.write(line);
            bw.newLine();
        }
        bw.flush();

        br.close();
        bw.close();
    }
}
```



### 3. 转换流

#### 3.1 `InputStreamWriter`

#### 3.2 `OutputStreamWriter`

![转换流](E:\000AAB_class_heima\Java-se-advance\my-note\image\转换流.png)

```
    private static void method2() throws IOException {
        //如何解决乱码现象
        //文件是什么码表,那么咱们就必须使用什么码表去读取.
        //我们就要指定使用GBK码表去读取文件.
        InputStreamReader isr = new InputStreamReader(new FileInputStream("C:\\Users\\apple\\Desktop\\a.txt"),"gbk");
        int ch;
        while((ch = isr.read())!=-1){
            System.out.println((char) ch);
        }
        isr.close();


        OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream("C:\\Users\\apple\\Desktop\\b.txt"),"UTF-8");
        osw.write("我爱学习,谁也别打扰我");
        osw.close();
    }
```



### 4.对象操作流

- 可以把对象以字节的形式写道本地文件, 直接打开文件, 是读不懂的, 需要再次用对象操作流读到内存中

#### 4.1 `ObjectOutputStream`

- 将对象保存到磁盘中, 或者在网络中传输对象

#### 4.2 `ObjectInputStream`

- 把写到本地文件中的对象读到内存中, 或者接收网络中传输的对象

![对象操作流](E:\000AAB_class_heima\Java-se-advance\my-note\image\对象操作流.png)

```java
public class User implements Serializable {
    //如果想要这个类的对象能被序列化,那么这个类必须要实现一个接口Serializable:称之为是一个标记性接口,里面没有任何的抽象方法;只要一个类实现了这个Serializable接口,那么就表示这个类的对象可以被序列化.
    private String username;
    private String password;
    //InvalidClassException异常的解决办法
    private static final long serialVersionUID = 1L;
    
    //如果某个变量不想被序列化, 添加transient修饰符


    public User() {
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "User{" +
                "username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
-----------------------------------------------------
public class Test04 {	//对象操作输出流(序列化流)
    public static void main(String[] args) throws IOException {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("a.txt"));
        User user = new User("zhangsan", "qwer");
        oos.writeObject(user);
        oos.close();
    }
}
-----------------------------------------------------
public class Test05 {	//对象操作输入流(反序列化流)
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("java-se-advanced\\a.txt"));
        User user = (User) ois.readObject();
        System.out.println(user);
        ois.close();
    }
}
```

- 读多个对象,注意事项: 读对象的时候无法判断是否读到末尾, 因此要捕获异常

```java
public class Test {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Student stu1 = new Student("deedee", 18);
        Student stu2 = new Student("xianxian", 19);
        Student stu3 = new Student("weiwei", 20);
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("D:\\测试目录4\\Student.txt"));
        oos.writeObject(stu1);
        oos.writeObject(stu2);
        oos.writeObject(stu3);

        oos.close();

        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("D:\\测试目录4\\Student.txt"));
        while (true) {
            try {
                Object o = ois.readObject();
                System.out.println(o);
            } catch (EOFException e) {
                break;
            }
        }

    }
}
```

- 读多个对象,第二种写法(`采用集合进行存储与读取`)

```java
public class Test07 {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Student stu1 = new Student("deedee", 18);
        Student stu2 = new Student("xianxian", 19);
        Student stu3 = new Student("weiwei", 20);
        ArrayList<Student> stus = new ArrayList<>();
        stus.add(stu1);
        stus.add(stu2);
        stus.add(stu3);

        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("D:\\测试目录4\\Student2.txt"));
        oos.writeObject(stus);
        oos.close();

        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("D:\\测试目录4\\Student2.txt"));
        ArrayList<Student> readStus = (ArrayList<Student>) ois.readObject();
        System.out.println(readStus);
        ois.close();


    }
}
```

### 5.Properties

与map类似

- 特有方法

  setProperty() = put()

  getProperty()  = get()

  stringPropertyNames() = keySet()

- 与IO相结合的方法

  void load(Reader reader)		将本地文件中的键值对数据读取到集合中

  void store(Writer writer, String comments)		将集合中的数据以键值对形式保存在本地
  
  ```java
  public class Test08 {
  //Properties读取
      public static void main(String[] args) throws IOException {
          Properties prop = new Properties();
          FileReader fr = new FileReader("java-se-advanced\\src\\day12_filewriter\\before08\\测试.properties");
          prop.load(fr);
          fr.close();
  
          System.out.println(prop);
      }
  }

  
  ```
  

## day13 多线程

- 并行: 在同一时刻, 有多个指令在多个CPU上`同时`执行.
- 并发: 在同一时刻, 有多个指令在单个CPU上`交替`执行.
- 进程: 正在进行软件(就是操作系统正在运行的一个应用程序)
  - 独立性: 进程是一个能独立运行的基本单位, 同时也是系统分配资源和调度的独立单位
  - 动态性:  进程的实质是程序的一次执行过程, 进程是动态产生, 动态消亡的
  - 并发性: 任何进程都可以同其他进程一起并发执行
- 线程: 是进程中的单个顺序控制流, 是一条执行路径(比如:360软件中的杀毒,扫描木马,清理垃圾)
  - 单线程: 一个进程如果只有一条执行路径, 则成为单线程程序
  - 多线程:一个进程如果有多条执行路径, 则称为多线程程序

### 1.多线程的实现方案

- 方式1: run()  start()

  见`获取设置线程名称`

  

- 方式2: 实现Runnuable接口

  见`获得当前线程对象`

- 方式3: Callable和Future

```java
public class MyCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        for (int i = 0; i < 100; i++) {
            System.out.println("跟女孩表白" + i);
        }
        //返回值就表示线程运行完毕之后的结果
        return "答应";
    }
}
-----------------------------------------------------
public class Test03 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //线程开启之后需要执行里面的call方法
        MyCallable mc = new MyCallable();

        //可以获取线程执行完毕之后的结果, 也可以作为参数传递给Thread对象
        FutureTask<String> ft = new FutureTask<>(mc);

        //创建线程对象
        Thread t1 = new Thread(ft);

        t1.start();
        //get方法一定要在线程启动之后才能获得结果
        String s = ft.get();

        System.out.println(s);
    }
}
```

### 2.线程成员方法

#### 2. 1 获取设置线程名称	setName()	getName()

```java
public class MyThread extends Thread{
    public MyThread() {
    }

    public MyThread(String name) {
        super(name);
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(getName() + "@@@" + i);
        }
    }
}
----------------------------------------------------
public class Test {
    public static void main(String[] args) {
        MyThread t1 = new MyThread("小菜");
        MyThread t2 = new MyThread("小强");

//        t1.setName("小菜");
//        t2.setName("小强");
        t1.start();
        t2.start();
    }
}
```

#### 2. 2 获得当前线程对象	Thread.currentThread()

```java
public class MyRunnable implements Runnable{
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName()+"方式实现多线程"+i);
        }
    }
}
-----------------------------------------------------
public class Test02 {
    public static void main(String[] args) {
        //创建了一个参数的对象
        MyRunnable mr = new MyRunnable();
        //创建了一个线程对象, 并把参数传递给这个线程;
        //在线程启动之后,执行的就是参数里面的run方法
        Thread t1 = new Thread(mr);
        t1.start();

        MyRunnable mr2 = new MyRunnable();
        Thread t2 = new Thread(mr2);
        t2.start();
        System.out.println(Thread.currentThread().getName());
    }
}
```

#### 2.3 线程休眠	Thread.sleep(100)

```java
public class MyRunnable implements Runnable{


    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"---"+i);
        }
    }
}
-----------------------------------------------------
public class Test06 {
    public static void main(String[] args) throws InterruptedException {
        MyRunnable mr = new MyRunnable();

        Thread t1 = new Thread(mr);
        Thread t2 = new Thread(mr);

        t1.start();
        t2.start();
    }
}
```

#### 2.4 线程优先级	

```java
public class MyCallable implements Callable<String> {
    public MyCallable() {
    }

    @Override
    public String call() throws Exception {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName()+"---"+ i);
        }
        return "线程执行完毕";
    }
}
----------------------------------------------------
public class Test07 {
    public static void main(String[] args) {
        //线程开启之后需要执行里面的call方法
        MyCallable  mc = new MyCallable();

        FutureTask<String> ft = new FutureTask<>(mc);

        Thread t1 = new Thread(ft);
        t1.setName("飞机");
        t1.start();

        MyCallable  mc2 = new MyCallable();
        FutureTask<String> ft2 = new FutureTask<>(mc2);
        Thread t2 = new Thread(ft2);
        t2.setPriority(8);
        t2.setName("坦克");
        t2.start();


    }
}
```

#### 2.5 后台线程/守护线程

```java
public class MyThread1 extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(getName() + "---" +i);
        }
    }
}
-----------------------------------------------------
public class MyThread2 extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(getName() + "---" +i);
        }
    }
}
-----------------------------------------------------
public class Test01 {
    public static void main(String[] args) {
        MyThread1 t1 = new MyThread1();
        MyThread2 t2 = new MyThread2();
        t1.setName("女神");
        t2.setName("备胎");

        //把第二个线程设置为守护线程
        //当普通线程执行完之后, 守护线程也没有执行下去的必要
        t2.setDaemon(true);

        t1.start();
        t2.start();
    }

}
```

#### 2.6 同步代码块

```java
public class Ticket implements Runnable{
    //票的数量
    private int ticket = 100;
    private Object obj = new Object();
    @Override
    public void run() {
        while (true) {
            synchronized (obj) {
                if(ticket <= 0){
                    break;
                }
                //模拟出票时间
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+"在卖票,还剩下"+ --ticket+"张票");
            }
        }
    }
}
-----------------------------------------------------
public class Test {
    //需求:某电影院目前正在上映国产大片, 共有100张票,
    // 而它有3个窗口卖票,请设计一个程序模拟该电影院卖票
    public static void main(String[] args) {
        Ticket ticket = new Ticket();

        Thread t1 = new Thread(ticket);
        Thread t2 = new Thread(ticket);
        Thread t3 = new Thread(ticket);

        t1.setName("窗口一");
        t2.setName("窗口二");
        t3.setName("窗口三");

        t1.start();
        t2.start();
        t3.start();
    }
}
```

- Lock锁

```java
public class Ticket implements Runnable{
    //票的数量
    private int ticket = 100;
    Lock ReentrantLock = new ReentrantLock();

    @Override
    public void run() {
        while (true) {
            //synchronized (obj) {
            try {
                ReentrantLock.lock();
                if(ticket <= 0){
                    break;
                }
                //模拟出票时间
                Thread.sleep(10);

                System.out.println(Thread.currentThread().getName()+"在卖票,还剩下"+ --ticket+"张票");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                ReentrantLock.unlock();
            }
            //}
        }
    }
}
--------------------------------------------------
public class Test10 {
    public static void main(String[] args) {
        Ticket ticket = new Ticket();

        Thread t1 = new Thread(ticket);
        Thread t2 = new Thread(ticket);

        t1.setName("窗口一");
        t2.setName("窗口二");

        t1.start();
        t2.start();
    }
}
```

### 3. 生产者消费者模式

​	阻塞队列

```java
//模板:
	//1.while(true)死循环
	//2.sychronized 锁,锁对象要唯一
	//3.判断,共享数据是否结束,结束
	//4.判断,共享数据是否结束,没有结束
```



## ==day16==

### 1. 线程状态

- 新建状态(new)	=>    创建线程

- 就绪状态(runnable)    =>    start方法
- 阻塞状态(blocked)    =>    无法获得锁对象
- 等待状态(waiting)    =>    wait方法
- 计时等待(timed_waiting)    =>    sleep方法
- 结束等待(terminated)    =>     全部代码运行完毕

```java
public static void main(String[] args) {
        //创建一个默认的线程池对象
        ExecutorService executorService = Executors.newCachedThreadPool();

        executorService.submit(() -> System.out.println(Thread.currentThread().getName() + "在执行了"));
        executorService.submit(() -> System.out.println(Thread.currentThread().getName() + "在执行了"));
        executorService.shutdown();
    }
```

```
public class Test02 {
    public static void main(String[] args) {
        //参数不是初始值而是最大值
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        executorService.submit(() -> System.out.println(Thread.currentThread().getName() + "在执行了"));
        executorService.submit(() -> System.out.println(Thread.currentThread().getName() + "在执行了"));
        executorService.shutdown();
    }
}
```



### 2. 线程池

#### 2.1四大线程池

```java
public class Test03 {
    public static void main(String[] args) {
        //缓冲(无界)线程池 ->  恶意请求
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.submit(()-> System.out.println(Thread.currentThread().getName()+"执行了"));
        executorService.shutdown();
        //固定线程池 ->  不灵活
        ExecutorService executorService1 = Executors.newFixedThreadPool(10);    //nThreads - 线程池最大数量
        executorService1.submit(() -> System.out.println(Thread.currentThread().getName() + "执行了") );
        executorService1.shutdown();
        //单例线程池 -> 鸡肋
        ExecutorService executorService2 = Executors.newSingleThreadExecutor();
        executorService2.submit(() -> System.out.println(Thread.currentThread().getName() + "执行了"));

        //延迟线程池 可以定时执行  -> 技术替代
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(10);
        scheduledExecutorService.schedule(() ->
            System.out.println(Thread.currentThread().getName() + "执行了")
            ,1, TimeUnit.SECONDS
        );
    }
}
```



#### 2.2 创建线程池七大参数

```java
package com.itheima.mythreadpool;


import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class MyThreadPoolDemo3 {
//     参数一：核心线程数量---正式员工                      --不能小于0--
//     参数二：最大线程数---  餐厅最大员工数                 --不能小于0，最大数量>=核心线程数量--
//     参数三：空闲线程最大存活时间--- 临时员工空闲多长时间被辞退(值)    --不能小于0--
//     参数四：时间单位 ---临时员工空闲多长时间被辞退(单位)    --时间单位--
//     参数五：任务队列 ---排队的客户                         --不能为null--
//     参数六：创建线程工厂 ---从哪里招人                      --不能为null--
//     参数七：任务的拒绝策略 ---排队的人过多，超出顾客请下次再来(拒绝服务)   --不能为null--
    public static void main(String[] args) {
        ThreadPoolExecutor pool = new ThreadPoolExecutor(2,5,2,TimeUnit.SECONDS,new ArrayBlockingQueue<>(10), Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());
        pool.submit(new MyRunnable());
        pool.submit(new MyRunnable());
        pool.shutdown();
    }
}
```

#### 2.3 四大拒绝策略

RejectedExecutionHandler是jdk提供的一个任务拒绝策略接口，它下面存在4个子类。

```
ThreadPoolExecutor.AbortPolicy: 		    丢弃任务并抛出RejectedExecutionException异常。是默认的策略。
ThreadPoolExecutor.DiscardPolicy： 		   丢弃任务，但是不抛出异常 这是不推荐的做法。
ThreadPoolExecutor.DiscardOldestPolicy：    抛弃队列中等待最久的任务 然后把当前任务加入队列中。
ThreadPoolExecutor.CallerRunsPolicy:        调用任务的run()方法绕过线程池直接执行。
```

明确线程池最多同时可执行的任务数 = 队列容量 + 最大线程数

### 3. volatile

 JMM规定:

​     1.被volatitle关键字，修饰的变量，被修改时，会立即把数据刷新到主内存

​     2.每次访问被volatitle关键字修改的变量时，会把本地内存中的变量副本，清空

```java
//以基金案例, 通过在关键词前修饰符volatile
package com.itheima.myvolatile;
public class Money {
    public static volatile int money = 100000;
}
```



### 4. 原子性

- 除了long和double之外的所有基本数据类型的直接赋值(例如:int count=0单一这一行是具有原子性的)

#### 4.1 AtomicInteger

#### 4.2 Hashtable

#### 4.3 ConcurrentHashMap

#### 4.5 CountDownLatch

```java
/*
    老师类
 */
public class Teacher extends Thread {
    private CountDownLatch cdl;

    public Teacher(CountDownLatch cdl, String name) {
        this.cdl = cdl;
        super.setName(name);
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+"已经准备好讲课了,正在等待同学们到齐...");
        try {
            cdl.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName()+"开始讲java多线程了");
    }
}
--------------------------------------------------------------------
/**
 * 学生类
 */
public class Student extends Thread {
    private CountDownLatch cdl;
    public Student(CountDownLatch cdl, String name) {
        this.cdl = cdl;
        super.setName(name);
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+"到了");
        cdl.countDown();
    }
}
--------------------------------------------------------------------
public class Test02 {
    public static void main(String[] args) {
        CountDownLatch cdl = new CountDownLatch(10);

        Teacher t = new Teacher(cdl,"张老师");

        boolean flag = true;
        for (int i = 1; i <= 10; i++) {
            Student ts = new Student(cdl,"学生" + i);
            ts.start();
            if(flag){
                t.start();
                flag = false;
            }
        }
    }
}
```

#### 4.6 Semaphore

## ==day15 网络编程==

- ipconfig：查看本机IP地址
- ping IP地址：检查网络是否连
- 127.0.0.1 （localhost）：可以代表本机地址
- InetAdress类: 此类表示Internet协议（IP）地址

| 说明                                                         | 方法名                                    |
| ------------------------------------------------------------ | ----------------------------------------- |
| 确定主机名称的IP地址。主机名称可以是机器名称，也可以是IP地址 | static InetAddress getByName(String host) |
| 获取此IP地址的主机名                                         | String getHostName()                      |
| 返回文本显示中的IP地址字符串                                 | String getHostAddress()                   |



### 1. 三要素

#### 1.1 IP地址 : 设备在网络中的地址, 是唯一的标识

- 全称"互联网协议地址", 也称IP地址. 是分配给上网设备的数字标签. 常见的IP分类为:ipv4和ipv6

  - IPv4: 32bit(4字节)	=>	192.168.1.66

  - IPv6: 采用128位地址长度, 分成8组 => 2001:DB8:0:23:8:800:200C:417A

    FF01:0:0:0:0:0:0:1101 => FF01::1101

  - ipconfig: 查看本机IP地址

  - ping IP地址: 检查网络是否联通

  - 特殊IP地址: 127.0.0.1: 是回送地址也称本地回环地址, 可以代表本机的IP地址, 一般用来测试使用

```java
public class Test {
    public static void main(String[] args) throws UnknownHostException {
        InetAddress address = InetAddress.getByName("SaddyFire");

        String hostName = address.getHostName();
        System.out.println(hostName);

        byte[] ip = address.getAddress();
        System.out.println(ip);
    }
}
```



#### 1.2 端口 : 应用程序在设备中唯一的标识

- 端口号: 用两个字节表示的整数, 它的取值范围是0~65535.

  ​			其中0~1023之间的端口号用于一些知名的网络服务或者应用.

  ​			我们自己使用1024以上的端口即可

  ​			注: 一个端口号只能被一个应用程序使用

#### 1.3 协议 : 数据在网络中传输的规则

UDP协议

- 用户数据报协议(User Datagram Protocol)

- UDP是`面向无连接`通信协议.

  速度快,有大小限制一次最多发送64K, 数据不安全, 易丢失数据.

```java
public class Test02 {
    public static void main(String[] args) throws IOException {
        //创建数据包通讯端对象
        DatagramSocket ds = new DatagramSocket();
        String s = "我正在发送";
        byte[] bytes = s.getBytes();
        InetAddress address = InetAddress.getByName("127.0.0.1");
        int port = 10000;
        //创建打包对象
        DatagramPacket dp = new DatagramPacket(bytes,bytes.length,address,port);

        ds.send(dp);

        ds.close();
    }
}
```

```java
public class Test03 {
    //如果接收端在启动之后没有收到数据, 那么会死等(阻塞)
    //在接收数据的时候, 需要调用一个getLength方法,表示接收到了多少字节
    public static void main(String[] args) throws IOException {
        //表示接收从10000端口接收数据的.
        DatagramSocket ds = new DatagramSocket(10000);

        byte[] bytes = new byte[1024];
        DatagramPacket dp = new DatagramPacket(bytes, bytes.length);
        ds.receive(dp);

        //byte[] data = dp.getData();
        int len = dp.getLength();
        System.out.println(new String(bytes,0,len));

        ds.close();
    }
}
```

小练习

```java
public class ClientDemo {
    public static void main(String[] args) throws IOException {
        DatagramSocket ds = new DatagramSocket();
        Scanner sc = new Scanner(System.in);
        while (true) {

            System.out.println("请输入要发送的文字");
            String next = sc.next();
            if(next.equals("886")){
                break;
            }
            //转成字节数组
            byte[] bytes = next.getBytes();
            //定义IP
            InetAddress address = InetAddress.getByName("127.0.0.1");
            //定义端口
            int port = 10000;
            //打包数据(传输数据字节数组, 长度, IP, 端口)
            DatagramPacket dp = new DatagramPacket(bytes,bytes.length,address,port);

            ds.send(dp);
        }

        ds.close();
    }
}
-----------------------------------------------
public class ServerDemo {
    public static void main(String[] args) throws IOException {
        DatagramSocket ds = new DatagramSocket(10000);

        while (true) {
            byte[] bytes = new byte[1024];
            DatagramPacket dp = new DatagramPacket(bytes, bytes.length);

            ds.receive(dp);

            byte[] data = dp.getData();
            int len = dp.getLength();
            System.out.println(new String(data ,0, len));
        }

        //ds.close();
    }
}
```

- UDP的三种通信方式
  - 单播
  - 组播

```java
public class ClientDemo {
    public static void main(String[] args) throws IOException {
        DatagramSocket ds = new DatagramSocket();

        String s = "我正在发送文件";
        byte[] bytes = s.getBytes();
        InetAddress address = InetAddress.getByName("224.0.1.0");
        int port = 10000;
        DatagramPacket dp = new DatagramPacket(bytes, bytes.length, address, port);

        ds.send(dp);

        ds.close();
    }
}
-----------------------------------------------------
public class ServerDemo {
    public static void main(String[] args) throws IOException {
        MulticastSocket ms = new MulticastSocket(10000);

        DatagramPacket dp = new DatagramPacket(new byte[1024], 1024);

        //把当前计算机绑定一个组播地址, 表示添加到这一组中,
        ms.joinGroup(InetAddress.getByName("224.0.1.0"));

        ms.receive(dp);

        byte[] data = dp.getData();
        int length = dp.getLength();
        System.out.println(new String(data,0,length));

        ms.close();
    }
}
```

- 
  - 广播

```java
public class ClientDemo {
    public static void main(String[] args) throws IOException {
        DatagramSocket ds = new DatagramSocket();

        String s = "我正在写代码";
        byte[] bytes = s.getBytes();
        InetAddress address = InetAddress.getByName("255.255.255.255");
        int port = 10000;

        DatagramPacket dp = new DatagramPacket(bytes, 0, bytes.length, address, port);

        ds.send(dp);

        ds.close();
    }
}
-----------------------------------------------------
public class ServerDemo {
    public static void main(String[] args) throws IOException {
        DatagramSocket ds = new DatagramSocket(10000);

        DatagramPacket dp = new DatagramPacket(new byte[1024],1024);

        ds.receive(dp);

        byte[] data = dp.getData();

        int len = dp.getLength();

        System.out.println(new String(data,0,len));
    }
}
```

TCP协议

- 传输控制协议(Transmission Control Protocol)

- TCP协议是`面向连接`的通信协议

  速度慢, 没有大小限制, 数据安全.

```java
public class ClientDemo {
    public static void main(String[] args) throws IOException {
        //创建Socket对象
        Socket socket = new Socket("127.0.0.1",10000);
        //获取一个IO流
        OutputStream os = socket.getOutputStream();
        os.write("hello".getBytes());

        //释放资源
        os.close();
        socket.close();
    }
}
-----------------------------------------------------
public class ServerDemo {
    public static void main(String[] args) throws IOException {
        //创建Socket对象
        ServerSocket ss = new ServerSocket(10000);
        //等待客户端连接
        Socket accept = ss.accept();
        //获得输入流对象
        InputStream is = accept.getInputStream();
        int b;
        while ((b = is.read()) != -1) {
            System.out.print((char) b);
        }
        //释放资源
        is.close();
        ss.close();
    }
}
```



### 2. 文件的上传与下载



---



## ==day16 类加载器==

## 1. 类加载器

双亲委派模型

## 2.反射



Class.forName(全类名)方法



构造

| 方法名                                                       | 说明                           |
| ------------------------------------------------------------ | ------------------------------ |
| Constructor<?>[] getDeclaredConstructors()                   | 返回所有构造方法对象的数组     |
| Constructor<?>[] getConstructors()                           | 返回所有公共构造方法对象的数组 |
| Constructor<T> getConstructor(Class<?>... parameterTypes)    | 返回单个公共构造方法对象       |
| Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes) | 返回单个构造方法对象           |

实例化

| 方法名                           | 说明                        |
| -------------------------------- | --------------------------- |
| T newInstance(Object...initargs) | 根据指定的构造方法创建对象  |
| setAccessible(boolean flag)      | 设置为true,表示取消访问检查 |

成员变量

| 方法名                              | 说明                           |
| ----------------------------------- | ------------------------------ |
| Field[] getFields()                 | 返回所有公共成员变量对象的数组 |
| Field[] getDeclaredFields()         | 返回所有成员变量对象的数组     |
| Field getField(String name)         | 返回单个公共成员变量对象       |
| Field getDeclaredField(String name) | 返回单个成员变量对象           |

赋值

| 方法名                             | 说明   |
| ---------------------------------- | ------ |
| void set(Object obj, Object value) | 赋值   |
| Object get(Object obj)             | 获取值 |

成员方法

| 方法名                                                       | 说明                                       |
| ------------------------------------------------------------ | ------------------------------------------ |
| Method[] getMethods()                                        | 返回所有公共成员方法对象的数组，包括继承的 |
| Method[] getDeclaredMethods()                                | 返回所有成员方法对象的数组，不包括继承的   |
| Method getMethod(String name, Class<?>... parameterTypes)    | 返回单个公共成员方法对象                   |
| Method getDeclaredMethod(String name, Class<?>... parameterTypes) | 返回单个成员方法对象                       |

使用方法

| 方法名                                    | 说明     |
| ----------------------------------------- | -------- |
| Object invoke(Object obj, Object... args) | 运行方法 |

综合题

![反射综合例题](E:\000AAB_class_heima\Java-se-advance\my-note\image\反射综合例题.png)

```java
/**
 * 创建一个Student 对象，并且name复制为张三  然后调用对象得show 方法
 * 
 */
public class class02 {
    public static void main(String[] args) throws NoSuchMethodException, ClassNotFoundException, NoSuchFieldException, InvocationTargetException, InstantiationException, IllegalAccessException {
        Class aClass = Class.forName("day17_reflect.class02.Student");      //第一种方法, 通过Class.forName()
        Class<Student> studentClass = Student.class;        //第二种方法, 通过类名.class

        Constructor constructor = aClass.getDeclaredConstructor();      //获取构造方法
        constructor.setAccessible(true);
        Student student = (Student) constructor.newInstance();      //构造方法实例化
        Field name = aClass.getDeclaredField("name");       //获取成员变量声明
        name.setAccessible(true);       //获取访问权限
        name.set(student,"张三");     //成员变量赋值
        Method show = aClass.getDeclaredMethod("show");     //获取成员方法声明
        show.setAccessible(true);       //获取访问权限
        show.invoke(student);       //调用方法

    }
}
```

