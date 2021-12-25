


## day15 网络编程

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



## day16 类加载器

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

