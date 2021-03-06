            

## 反射、GC、JVM优化面经

**类加载器与反射**

**简述java****类加载机制?**

虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验，解析和初始化，最终形成可以被虚拟机直接使用的java类型。

**描述一下JVM****加载Class****文件的原理机制？**

Java中的所有类，都需要由类加载器装载到JVM中才能运行。类加载器本身也是一个类，而它的工作就是把class文件从硬盘读取到内存中。在写程序的时候，我们几乎不需要关心类的加载，因为这些都是隐式装载的，除非我们有特殊的用法，像是反射，就需要显式的加载所需要的类。

类装载方式，有两种 ：

1.隐式装载， 程序在运行过程中当碰到通过new 等方式生成对象时，隐式调用类装载器加载对应的类到jvm中，

2.显式装载， 通过class.forname()等方法，显式加载需要的类

Java类的加载是动态的，它并不会一次性将所有类全部加载后再运行，而是保证程序运行的基础类(像是基类)完全加载到jvm中，至于其他类，则在需要的时候才加载。这当然就是为了节省内存开销。

**说一下类装载的执行过程？**

类装载分为以下 5 个步骤：

加载：根据查找路径找到相应的 class 文件然后导入；

验证：检查加载的 class 文件的正确性；

准备：给类中的静态变量分配内存空间；

解析：虚拟机将常量池中的符号引用替换成直接引用的过程。符号引用就理解为一个标示，而在直接引用直接指向内存中的地址；

初始化：对静态变量和静态代码块执行初始化工作。

 **什么是类加载器，类加载器有哪些?**

宏观来看只有两种类加载器：启动类加载器（c++实现）和其他所有的类加载器（java语言）。

主要有一下四种类加载器:

**启动类加载器**(Bootstrap ClassLoader)：用来加载java核心类库，无法被java程序直接引用。

**平台****/****扩展类加载器**(extensions class loader)：它用来加载 Java 的扩展库。Java 虚拟机的 实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载Java类。（JDK9之后，扩展类加载器被重命名为平台类加载器）。

**系统类加载器**（system class loader）：它根据 Java 应用的类路径（CLASSPATH）来加载 Java 类。一般来说，Java 应用的类都是由它来完成加载的。可以通过 ClassLoader.getSystemClassLoader()来获取它。

用户**自定义类加载器**，通过继承 java.lang.ClassLoader类的方式实现。

**什么是反射机制？**

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；

对于任意一个对象，都能够调用它的任意一个方法和属性；

这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

**什么是静态编译，什么是动态编译？**

静态编译：在编译时确定类型，绑定对象

动态编译：运行时确定类型，绑定对象

**反射机制的优缺点有哪些？**

优点： 运行期类型的判断，动态加载类，提高代码灵活度。

缺点： 性能瓶颈：反射相当于一系列解释操作，通知 JVM 要做的事情，过程比直接的java代码要多了一步委托的过程，反射需要类加载器通过双亲委派模型实现动态编译，效率较低。

**什么是双亲委派机制？**

首先，JVM中有三大类加载器：启动类加载器（最顶层），平台类加载器（中层），系统类加载器（下层）。

双亲委派模型就是指一个类加载器收到了类加载请求，它不会直接自己先加载，而把请求委托给父类的加载器去执行，如果父类加载器还存在其父类加载器，就再往上委托，赴到最顶层的类加载器，如果父类加载器可以完成类加载任务，就成功返回，若不能，就向下传递，让子加载器去加载，这就是双亲委派模式。

双亲委派模型主要是用来保证同一个类只能被一个类加载器加载。

**怎么破坏双亲委派机制？**

一般在自定义类加载器中，我们不希望通过双亲委派机制一层层向上再下来，而是希望直接通过自己定义的类加载器直接实现类加载，来提升加载性能，比如Tomcat中的web容器类加载器就是破坏了双亲委托模式的，里面的WebApplicationClassLoader除了核心类库外，都是优先加载自己路径下的Class。

要打破双亲委派机制，只要在重写loadclass的过程中，不遵从JVM规范就行了，也就是不盲目优先向Parednt的ClassLoader查找即可。

**你在哪些场景下用过反射？**

反射在框架中有频繁的被使用，比如JDK动态代理，Spring中的注入属性，调用方法等。

反射更多是为了灵活舍弃一部分性能，自己使用一般用在工具类中，比如频繁通过参数名来调用指定的方法时，可以用通过反射去匹配指定的方法名，然后实现功能。

**Java****中获取反射的三种方法是什么？**

1、类名.class属性；2、对象名.getClass()方法；3、Class.forName(全类名)方法

**反射可以获取“私有”方法或构造函数吗或私有成员变量吗？**

可以。有专门反射私有构造函数的方法 clazz.getDeclaredConstructor(int.class);来读取私有的构造函数，私有成员变量和私有方法也一样，但用这个方法读取完还需要设置一下**暴力反射**才行：c.setAccessible(true)。

**JMV**

**说一下 JVM** **的主要组成部分及其作用？**

![](file:///C:/Users/SADDYF~1/AppData/Local/Temp/msohtmlclip1/01/clip_image001.png)

JVM包含两个子系统和两个组件，两个子系统为Class loader(类装载)、Execution engine(执行引擎)；两个组件为Runtime data area(运行时数据区)、Native Interface(本地接口)。

Class loader(类装载)：根据给定的全限定名类名(如：java.lang.Object)来装载class文件到Runtime data area中的method area。

Execution engine（执行引擎）：执行classes中的指令。

Native Interface(本地接口)：与native libraries交互，是其它编程语言交互的接口。

Runtime data area(运行时数据区域)：这就是我们常说的JVM的内存。

顺序 ：

1、首先通过编译器把 Java 代码转换成字节码，类加载器（ClassLoader）再把字节码加载到内存中，将其放在运行时数据区（Runtime data area）的方法区内。

2、而字节码文件只是 JVM 的一套指令集规范，并不能直接交给底层操作系统去执行，因此需要特定的命令解析器执行引擎（Execution Engine），将字节码翻译成底层系统指令。

3、再交由 CPU 去执行，而这个过程中需要调用其他语言的本地库接口（Native Interface）来实现整个程序的功能。

**下面是Java****程序运行机制详细说明**

Java程序运行机制步骤：

1、首先利用IDE集成开发工具编写Java源代码，源文件的后缀为.java；

2、再利用编译器(javac命令)将源代码编译成字节码文件，字节码文件的后缀名为.class；

3、运行字节码的工作是由解释器(java命令)来完成的。

![image.png](file:///C:/Users/SADDYF~1/AppData/Local/Temp/msohtmlclip1/01/clip_image003.png)

从上图可以看，java文件通过编译器变成了.class文件，接下来类加载器又将这些.class文件加载到JVM中。

其实可以一句话来解释：类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个 java.lang.Class对象，用来封装类在方法区内的数据结构。

 

**说一下 JVM** **运行时数据区？【重要】**

Java 虚拟机在执行 Java 程序的过程中会把它所管理的内存区域划分为若干个不同的数据区域。这些区域都有各自的用途，以及创建和销毁的时间，有些区域随着虚拟机进程的启动而存在，有些区域则是依赖线程的启动和结束而建立和销毁。Java 虚拟机所管理的内存被划分为如下几个区域：

![image.png](file:///C:/Users/SADDYF~1/AppData/Local/Temp/msohtmlclip1/01/clip_image005.png)

不同虚拟机的运行时数据区可能略微有所不同，但都会遵从 Java 虚拟机规范， Java 虚拟机规范规定的区域分为以下 5 个部分：

**1****、程序计数器（Program Counter Register****）：**当前线程所执行的字节码的行号指示器，个人感觉的他就是为多线程准备的，程序计数器是每个线程独有的，所以是线程安全的。它主要用于记录每个线程的执行情况。

**2****、Java** **虚拟机栈（Java Virtual Machine Stacks****）：**线程私有，用于存储局部变量表、操作数栈、动态链接、方法出口等信息；

**3****、本地方法栈（Native Method Stack****）：**线程私有，与虚拟机栈的作用是一样的，只不过虚拟机栈是服务 Java 方法的，而本地方法栈是为虚拟机调用 Native 方法服务的（Native方法是JVM底层的C语言对其它系统或硬件进行交互）；

· **4****、Java** **堆（Java Heap****）：**Java 虚拟机中内存最大的一块，是被所有线程共享的，几乎所有的对象实例都在这里分配内存；Java堆也叫GC堆，是垃圾收集器管理的主要区域，堆中可以细分为：新生代、老年代；再细致一点，新生代中又分为：Eden Space(伊甸园)、Survivor（ /səˈvaɪvər/）空间，Survivor空间又分为From区和to区。

**5****、方法区（Methed Area****）：**1.8之后方法区用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译后的代码等数据。

 

**方法区补充点：**

在JDK1.7以前HotSpot虚拟机使用永久代来实现方法区，永久代的大小在启动JVM时可以设置一个固定值（-XX:MaxPermSize），不可变。

在JDK1.7中 存储在永久代的部分数据就已经转移到Java Heap或者Native memory。譬如符号引用(Symbols)转移到了native memory，原本存放在永久代的字符常量池移出。但永久代仍存在于JDK 1.7中，并没有完全移除。

JDK1.8中进行了较大改动：

移除了永久代（PermGen），替换为元空间（Metaspace）；

永久代中的 class metadata 转移到了 native memory（本地内存，而不是虚拟机）；

永久代中的 interned Strings 和 class static variables 转移到了 Java heap；

永久代参数 （PermSize MaxPermSize） -> 元空间参数（MetaspaceSize MaxMetaspaceSize）

永久代（元空间）

在Java8中，永久代已经被移除，被一个称为“元数据区”（元空间，Metaspace）的区域所取代。  值得注意的是：元空间并不在虚拟机中，而是使用本地内存（之前，永久代是在jvm中）。

这样，解决了以前永久代的OOM问题，元数据和class对象存在永久代中，容易出现性能问题和内存溢出，毕竟是和老年代共享堆空间。java8后，永久代升级为元空间独立后，也降低了老年代GC的复杂度。

**深拷贝和浅拷贝**

浅拷贝（shallowCopy）只是增加了一个指针指向已存在的内存地址，

深拷贝（deepCopy）是增加了一个指针并且申请了一个新的内存，使这个增加的指针指向这个新的内存，使用深拷贝的情况下，释放内存的时候不会因为出现浅拷贝时释放同一个内存的错误。

浅复制：仅仅是指向被复制的内存地址，如果原地址发生改变，那么浅复制出来的对象也会相应的改变。

深复制：在计算机中开辟一块新的内存地址用于存放复制的对象。

 

**说一下堆栈的区别？**

1、物理地址：

堆的物理地址分配对对象是不连续的。因此性能慢些。在GC的时候也要考虑到不连续的分配，所以有各种算法。比如，标记-消除，复制，标记-压缩，分代（即新生代使用复制算法，老年代使用标记——压缩）

栈使用的是数据结构中的栈，先进后出的原则，物理地址分配是连续的。所以性能快。

2、内存分别：

堆因为是不连续的，所以分配的内存是在运行期确认的，因此大小不固定。一般堆大小远远大于栈。

栈是连续的，所以分配的内存大小要在编译期就确认，大小是固定的。

3、存放的内容：

堆存放的是对象的实例和数组。因此该区更关注的是数据的存储。

栈存放：局部变量，操作数栈，返回结果。该区更关注的是程序方法的执行。

程序的可见度：

堆对于整个应用程序都是共享、可见的。

栈只对于线程是可见的。所以也是线程私有。他的生命周期和线程相同。

 

**队列和栈是什么？有什么区别？**

队列和栈都是被用来预存储数据的。

操作的名称不同。队列的插入称为入队，队列的删除称为出队。栈的插入称为进栈，栈的删除称为出栈。

可操作的方式不同。队列是在队尾入队，队头出队，即两边都可操作。而栈的进栈和出栈都是在栈顶进行的，无法对栈底直接进行操作。

操作的方法不同。队列是先进先出（FIFO），即队列的修改是依先进先出的原则进行的。新来的成员总是加入队尾（不能从中间插入），每次离开的成员总是队列头上（不允许中途离队）。而栈为后进先出（LIFO）,即每次删除（出栈）的总是当前栈中最新的元素，即最后插入（进栈）的元素，而最先插入的被放在栈的底部，要到最后才能删除。

**堆里面的分区：Eden****，survival** **（from+ to****），老年代，各自的特点是什么？**

堆里面分为新生代和老生代（java8 取消了永久代，采用了 Metaspace-元空间），新生代包含 Eden+Survivor 区，survivor 区里面分为 from 和 to 区。

年轻代：

新创建的对象都会被分配到Eden区(如果该对象占用内存非常大，则直接分配到老年代区), 当Eden区内存不够的时候就会触发MinorGC（Survivor满不会引发MinorGC，而是将对象移动到老年代中），

在Minor GC开始的时候，对象只会存在于Eden区和Survivor from区，Survivor to区是空的。

Minor GC操作后，Eden区如果仍然存活（判断的标准是被引用了，通过GC root进行可达性判断）的对象，将会被移到Survivor To区。而From区中，对象在Survivor区中每熬过一次Minor GC，年龄就会+1岁，当年龄达到一定值(年龄阈值，可以通过-XX:MaxTenuringThreshold来设置，默认是15)的对象会被移动到年老代中，否则对象会被复制到“To”区。经过这次GC后，Eden区和From区已经被清空。

“From”区和“To”区互换角色，原Survivor To成为下一次GC时的Survivor From区, 总之，GC后，都会保证Survivor To区是空的。

奇怪为什么有 From和To，2块区域？这就要说到新生代Minor GC的算法了——复制算法。

把内存区域分为两块，每次使用一块，GC的时候把一块中的内容移动到另一块中，原始内存中的对象就可以被回收了，优点是避免内存碎片。

老年代：

随着Minor GC的持续进行，老年代中对象也会持续增长，导致老年代的空间也会不够用，最终会执行Major GC（MajorGC 的速度比 Minor GC 慢很多很多，据说10倍左右）。Major GC使用的算法是：标记清除（回收）算法或者标记压缩算法。

 标记清除（回收）：

1. 首先会从GC root进行遍历，把可达对象（存过的对象）打标记

2. 再从GC root二次遍历，将没有被打上标记的对象清除掉。

优点：老年代对象一般是比较稳定的，相比复制算法，不需要复制大量对象。之所以将所有对象扫描2次，看似比较消耗时间，其实不然，是节省了时间。举个栗子，数组 1,2,3,4,5,6。删除2,3,4，如果每次删除一个数字，那么5,6要移动3次，如果删除1次，那么5,6只需移动1次。

缺点：这种方式需要中断其他线程（STW），相比复制算法，可能产生内存碎片。

标记压缩：和标记清除算法基本相同，不同的就是，在清除完成之后，会把存活的对象向内存的一边进行压缩，这样就可以解决内存碎片问题。  当老年代也满了装不下的时候，就会抛出OOM（Out of Memory）异常。

 **Java****会存在内存泄漏吗？请简单描述**

内存泄漏是指不再被使用的对象或者变量一直被占据在内存中。理论上来说，Java是有GC垃圾回收机制的，也就是说，不再被使用的对象，会被GC自动回收掉，自动从内存中清除。

但是，即使这样，Java也还是存在着内存泄漏的情况，java导致内存泄露的原因很明确：长生命周期的对象持有短生命周期对象的引用就很可能发生内存泄露，尽管短生命周期对象已经不再需要，但是因为长生命周期对象持有它的引用而导致不能被回收，这就是java中内存泄露的发生场景。

**有遇到过栈溢出吗？一般是什么问题导致？**

栈溢出（StackOverflowError）是指栈内容全部被占用，而数据还要往里放。一般是递归错误或者出现死循环导致。

**对象创建方法，对象的内存分配，对象的访问定位。**

new 一个对象 。

 **java****字符串常量池、****class****常量池和运行时常量池的区别是什么？**

jvm的方法区里存放着类的版本，字段，方法，接口和常量池。

常量池里存储着字面量和符号引用。

![](file:///C:/Users/SADDYF~1/AppData/Local/Temp/msohtmlclip1/01/clip_image007.png)

**字符串常量池（string pool****）**

字符串常量池里的内容是在类加载完成，经过验证，准备阶段之后在堆中生成字符串对象实例，然后将该字符串对象实例的引用值存到string pool中（记住：string pool中存的是引用值而不是具体的实例对象，具体的实例对象是在堆中开辟的一块空间存放的）。string pool在每个HotSpot VM的实例只有一份，被所有的类共享。在jdk1.8后，将String常量池放到了堆中。

**class****常量池（符号引用）**

当java文件被编译成class文件之后，会在class文件中生成我们所说的class常量池，class文件中除了包含类的版本、字段、方法、接口等描述信息外，还有一项信息就是常量池表(constant pool table)，用于存放编译器生成的各种字面量(文本字符串、被声明为final的常量、基本数据类型的值)和符号引用(类和接口的全限定名、字段的名称和描述符、方法的名称和描述符)。

![](file:///C:/Users/SADDYF~1/AppData/Local/Temp/msohtmlclip1/01/clip_image009.png)

常量池中每一项常量都是一个表，常量表中共有（11+4+2=17）种结构不同的表结构数据（11）和动态语言相关的常量（4）以及之后完善的CONSTANT_Module_info和CONSTANT_Package_info两个常量（2）。

17个表的共同特点：表结构起始的第一位是u1类型的标志位（tag）,代表当前常量属于哪种常量类型。

**运行时常量池（直接引用）**

当类加载到内存中后，jvm就会将class常量池中的内容存放到运行时常量池中，由此可知，运行时常量池也是每个类都有一个。在上面我也说了，class常量池中存的是字面量和符号引用，也就是说他们存的并不是对象的实例，而是对象的符号引用值。而经过解析（resolve）之后，也就是把符号引用替换为直接引用，解析的过程会去查询字符串常量池，也就是我们上面所说的string pool，以保证运行时常量池所引用的字符串与字符串常量池中所引用的是一致的。

常量池（符号引用）-->运行时常量池（直接引用）-->字符串池常量池

字面量：比较接近java语言层面的常量概念，如文本字符串、被声明为final的常量值等。

符号引用：属于编译原理方面的概念，主要包括下面几类常量——

被模块导出或者开放的包

类和接口的全限定名

字段的名称和描述符

方法的名称和描述符

方法句柄和方法类型

动态调用点和动态常

直接引用：

直接引用和虚拟机的布局是相关的，不同的虚拟机对于相同的符号引用所翻译出来的直接引用一般是不同的。如果有了直接引用，那么直接引用的目标一定被加载到了内存中。

直接引用可以是： 

1：直接指向目标的指针。（个人理解为：指向对象，类变量和类方法的指针）

2：相对偏移量。 （指向实例的变量，方法的指针）

3：一个间接定位到对象的句柄。

**相关概念**

**1****、方法区中的运行时常量池**

运行时常量池是方法区的一部分。

CLass文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池，用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。

运行时常量池相对于CLass文件常量池的另外一个重要特征是具备动态性，Java语言并不要求常量一定只有编译期才能产生，也就是并非预置入CLass文件中常量池的内容才能进入方法区运行时常量池，运行期间也可能将新的常量放入池中，这种特性被开发人员利用比较多的就是String类的intern()方法。

**2****、常量池的好处**

常量池是为了避免频繁的创建和销毁对象而影响系统性能，其实现了对象的共享。

例如字符串常量池，在编译阶段就把所有的字符串文字放到一个常量池中。

（1）节省内存空间：常量池中所有相同的字符串常量被合并，只占用一个空间。

（2）节省运行时间：比较字符串时，==比equals()快。对于两个引用变量，只用==判断引用是否相等，也就可以判断实际值是否相等。

**java****内存模型**

首先，JAVA内存模型是指JMM，而不是指内存结构，内存结构是在物理上的区域划分，而JMM则是抽象概念上的划分。

JMM（内存模型）主要包括两块：**主内存**+**工作内存**。

主内存：多个线程间通信的共享内存称之为主内存，即，数据是多个线程工共享的，在物理内存结构上通常对应“堆”中的线程共享数据。

工作内存：多个线程各自对应自己的本地内存，即，数据只属于该线程自己的，在物理内存结构上通常对应“本地方法栈”中的线程私有数据。

Java内存模型规定了所有的变量都存储在主内存(Main Memory)中，每条线程还有自己的工作内存(Working Memory)，线程的工作内存中保存了被该线程使用到的变量的主内存副本拷贝，线程对变量的所有操作(读取、赋值等)都必须在工作内存中进行，而不能直接读写主内存中的变量，不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量值得传递均需要通过主内存来实现。

**JVM****优化**

**说一下 JVM** **调优的工具？**

JDK 自带了很多监控工具，都位于 JDK 的 bin 目录下，其中最常用的是 jconsole 和 jvisualvm 这两款视图监控工具。

jconsole：用于对 JVM 中的内存、线程和类等进行监控；

jvisualvm：JDK 自带的全能分析工具，可以分析：内存快照、线程快照、程序死锁、监控内存的变化、gc 变化等。

**常用的 JVM** **调优的参数都有哪些？**

-Xms2g：初始化堆大小为 2g；【常用】

-Xmx2g：堆最大内存为 2g；【常用】

-XX:NewRatio=4：设置年轻的和老年代的内存比例为 1:4；

-XX:SurvivorRatio=8：设置新生代 Eden 和 Survivor 比例为 8:2；

–XX:+UseParNewGC：指定使用 ParNew + Serial Old 垃圾回收器组合；

-XX:+UseParallelOldGC：指定使用 ParNew + ParNew Old 垃圾回收器组合；

-XX:+UseConcMarkSweepGC：指定使用 CMS + Serial Old 垃圾回收器组合；

-XX:+PrintGC：开启打印 gc 信息；

-XX:+PrintGCDetails：打印 gc 详细信息。

**GC-****垃圾收集器**

**简述Java****垃圾回收机制？**

在java中，程序员是不需要显示的去释放一个对象的内存的，而是由虚拟机自行执行。在JVM中，有一个垃圾回收线程，它是低优先级的，在正常情况下是不会执行的，只有在虚拟机空闲或者当前堆内存不足时，才会触发执行，扫面那些没有被任何引用的对象，并将它们添加到要回收的集合中，进行回收。

**GC****是什么？为什么要GC**

GC 是垃圾收集的意思（Gabage Collection）,内存处理是编程人员容易出现问题的地方，忘记或者错误的内存。

回收会导致程序或系统的不稳定甚至崩溃，Java 提供的 GC 功能可以自动监测对象是否超过作用域从而达到自动。

回收内存的目的，Java 语言没有提供释放已分配内存的显示操作方法。

 

**垃圾回收的优点和原理是什么？并考虑2****种回收机制**

java语言最显著的特点就是引入了垃圾回收机制，它使java程序员在编写程序时不再考虑内存管理的问题。

由于有这个垃圾回收机制，java中的对象不再有“作用域”的概念，只有引用的对象才有“作用域”。

垃圾回收机制有效的**防止内存泄露**，可以有效的使用可使用的内存。

垃圾回收器通常作为一个单独的低级别的线程运行，在不可预知的情况下对内存堆中已经死亡的或很长时间没有用过的对象进行清除和回收。

程序员不能实时的对某个对象或所有对象调用垃圾回收器进行垃圾回收。

垃圾回收有**分代复制垃圾回收、标记垃圾回收、增量垃圾回收**。

**垃圾回收器的基本原理是什么？垃圾回收器可以马上回收内存吗？有什么办法主动通知虚拟机进行垃圾回收？**

对于GC来说，当程序员创建对象时，GC就开始监控这个对象的地址、大小以及使用情况。

通常，GC采用有向图的方式记录和管理堆(heap)中的所有对象。通过这种方式确定哪些对象是"可达的"，哪些对象是"不可达的"。当GC确定一些对象为"不可达"时，GC就有责任回收这些内存空间。

可以。程序员可以手动执行System.gc()，通知GC运行，但是Java语言规范并不保证GC一定会执行。

 

**Java** **中都有哪些引用类型？**

强引用：发生 gc 的时候不会被回收。

软引用：有用但不是必须的对象，在发生内存溢出之前会被回收。

弱引用：有用但不是必须的对象，在下一次GC时会被回收。

虚引用（幽灵引用/幻影引用）：无法通过虚引用获得对象，用 PhantomReference 实现虚引用，虚引用的用途是在 gc 时返回一个通知。

**怎么判断对象是否可以被回收？**

垃圾收集器在做垃圾回收的时候，首先需要判定的就是哪些内存是需要被回收的，哪些对象是「存活」的，是不可以被回收的；哪些对象已经「死掉」了，需要被回收。

一般有两种方法来判断：

**1****、引用计数器法：**为每个对象创建一个引用计数，有对象引用时计数器 +1，引用被释放时计数 -1，当计数器为 0 时就可以被回收。它有一个缺点不能解决循环引用的问题；

**2****、可达性分析算法：**从 GC Roots 开始向下搜索，搜索所走过的路径称为引用链。当一个对象到 GC Roots 没有任何引用链相连时，则证明此对象是可以被回收的。

 

**在Java****中，对象什么时候可以被垃圾回收？**

当对象对当前使用这个对象的应用程序变得不可触及的时候，这个对象就可以被回收了。

**JVM****中的永久代中会发生垃圾回收吗？**

垃圾回收不会发生在永久代，如果永久代满了或者是超过了临界值，会触发完全垃圾回收(Full GC)。如果你仔细查看垃圾收集器的输出信息，就会发现永久代也是被回收的。这就是为什么正确的永久代大小对避免Full GC是非常重要的原因。请参考下Java8：从永久代到元数据区

(注：Java8中已经移除了永久代，新加了一个叫做元数据区（也叫元空间）的native内存区)

**什么是Full GC****？什么情况下会触发？**

Full GC是指清理整个堆空间——包括年轻代和老年代。

什么时候触发：

1.调用System.gc

2. 方法区空间不足

3.老年代空间不足，包括：

①新创建的对象都会被分配到Eden区，如果该对象占用内存非常大，则直接分配到老年代区，此时老年代空间不足。

②做minor gc操作前，发现要移动的空间（Eden区、From区向To区复制时，To区的内存空间不足）比老年代剩余空间要大，则触发full gc，而不是minor gc。

GC优化的本质，也是为什么分代的原因：减少GC次数和GC时间，避免全区扫描。

**说一下 JVM** **有哪些垃圾回收算法？**

1、标记-清除算法：标记无用对象，然后进行清除回收。缺点：效率不高，无法清除垃圾碎片。

2、复制算法：按照容量划分二个大小相等的内存区域，当一块用完的时候将活着的对象复制到另一块上，然后再把已使用的内存空间一次清理掉。缺点：内存使用率不高，只有原来的一半。

3、标记-整理算法：标记无用对象，让所有存活的对象都向一端移动，然后直接清除掉端边界以外的内存。

4、分代算法：根据对象存活周期的不同将内存划分为几块，一般是新生代和老年代，新生代基本采用复制算法，老年代采用标记整理算法。

**标记-****清除算法**

标记无用对象，然后进行清除回收。

标记-清除算法（Mark-Sweep）是一种常见的基础垃圾收集算法，它将垃圾收集分为**2****个阶段**：

1、标记阶段：标记出可以回收的对象。

2、清除阶段：回收被标记的对象所占用的空间。

标记-清除算法之所以是基础的，是因为后面讲到的垃圾收集算法都是在此算法的基础上进行改进的。

优点：实现简单，不需要对象进行移动。

缺点：标记、清除过程**效率低**，产生大量不连续的内存碎片，提高了垃圾回收的频率。

标记-清除算法的执行的过程如下图所示

![image.png](file:///C:/Users/SADDYF~1/AppData/Local/Temp/msohtmlclip1/01/clip_image011.png)

**复制算法**

为了解决标记-清除算法的效率不高的问题，产生了复制算法。它把内存空间划为两个相等的区域，每次只使用其中一个区域。垃圾收集时，遍历当前使用的区域，把存活对象复制到另外一个区域中，最后将当前使用的区域的可回收的对象进行回收。

优点：按顺序分配内存即可，实现简单、运行高效，不用考虑内存碎片。

缺点：可用的内存大小缩小为原来的一半，**对象存活率高时会频繁进行复制**。

复制算法的执行过程如下图所示

![image.png](file:///C:/Users/SADDYF~1/AppData/Local/Temp/msohtmlclip1/01/clip_image013.png)

**标记-****整理算法**

在新生代中可以使用复制算法，但是在老年代就不能选择复制算法了，因为老年代的对象存活率会较高，这样会有较多的复制操作，导致效率变低。标记-清除算法可以应用在老年代中，但是它效率不高，在内存回收后容易产生大量内存碎片。因此就出现了一种标记-整理算法（Mark-Compact）算法，与标记-整理算法不同的是，在标记可回收的对象后将所有存活的对象压缩到内存的一端，使他们紧凑的排列在一起，然后对端边界以外的内存进行回收。回收后，已用和未用的内存都各自一边。

优点：解决了标记-清理算法存在的内存碎片问题。

缺点：仍需要进行局部对象移动，一定程度上降低了效率。

标记-整理算法的执行过程如下图所示

![image.png](file:///C:/Users/SADDYF~1/AppData/Local/Temp/msohtmlclip1/01/clip_image015.png)

**分代收集算法**

当前商业虚拟机都采用分代收集的垃圾收集算法。分代收集算法，顾名思义是根据对象的存活周期将内存划分为几块。一般包括年轻代、老年代 和 永久代，如图所示：

![image.png](file:///C:/Users/SADDYF~1/AppData/Local/Temp/msohtmlclip1/01/clip_image017.png)

**说一下 JVM** **有哪些垃圾回收器？**

如果说垃圾收集算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现。下图展示了7种作用于不同分代的收集器，其中用于回收新生代的收集器包括Serial、PraNew、Parallel Scavenge，回收老年代的收集器包括Serial Old、Parallel Old、CMS，还有用于回收整个Java堆的G1收集器。不同收集器之间的连线表示它们可以搭配使用。

![image.png](file:///C:/Users/SADDYF~1/AppData/Local/Temp/msohtmlclip1/01/clip_image019.png)

Serial收集器（复制算法): 新生代单线程收集器，标记和清理都是单线程，优点是简单高效；

ParNew收集器 (复制算法): 新生代收并行集器，实际上是Serial收集器的多线程版本，在多核CPU环境下有着比Serial更好的表现；

Parallel Scavenge收集器 (复制算法): 新生代并行收集器，追求高吞吐量，高效利用 CPU。吞吐量 = 用户线程时间/(用户线程时间+GC线程时间)，高吞吐量可以高效率的利用CPU时间，尽快完成程序的运算任务，适合后台应用等对交互相应要求不高的场景；

Serial Old收集器 (标记-整理算法): 老年代单线程收集器，Serial收集器的老年代版本；

Parallel Old收集器 (标记-整理算法)： 老年代并行收集器，吞吐量优先，Parallel Scavenge收集器的老年代版本；

CMS(Concurrent Mark Sweep)收集器（标记-清除算法）： 老年代并行收集器，以获取最短回收停顿时间为目标的收集器，具有高并发、低停顿的特点，追求最短GC回收停顿时间。

G1(Garbage First)收集器 (标记-整理算法)： Java堆并行收集器，G1收集器是JDK1.7提供的一个新收集器，G1收集器基于“标记-整理”算法实现，也就是说不会产生内存碎片。此外，G1收集器不同于之前的收集器的一个重要特点是：G1回收的范围是整个Java堆(包括新生代，老年代)，而前六种收集器回收的范围仅限于新生代或老年代。

**新生代垃圾回收器和老年代垃圾回收器都有哪些？有什么区别？**

新生代回收器：Serial、ParNew、Parallel Scavenge

老年代回收器：Serial Old、Parallel Old、CMS

整堆回收器：G1

新生代垃圾回收器一般采用的是复制算法，复制算法的优点是效率高，缺点是内存利用率低；老年代回收器一般采用的是标记-整理的算法进行垃圾回收。

**详细介绍一下 CMS** **垃圾回收器？**

CMS 是英文 Concurrent Mark-Sweep 的简称，是以牺牲吞吐量为代价来获得最短回收停顿时间的垃圾回收器。对于要求服务器响应速度的应用上，这种垃圾回收器非常适合。在启动 JVM 的参数加上“-XX:+UseConcMarkSweepGC”来指定使用 CMS 垃圾回收器。

CMS 使用的是标记-清除的算法实现的，所以在 gc 的时候回产生大量的内存碎片，当剩余内存不能满足程序运行要求时，系统将会出现 Concurrent Mode Failure，临时 CMS 会采用 Serial Old 回收器进行垃圾清除，此时的性能将会被降低。

 

**简述分代垃圾回收器是怎么工作的？**

分代回收器有两个分区：老生代和新生代，新生代默认的空间占比总空间的 1/3，老生代的默认占比是 2/3。

新生代使用的是复制算法，新生代里有 3 个分区：Eden、To Survivor、From Survivor，它们的默认占比是 8:1:1，它的执行流程如下：

把 Eden + From Survivor 存活的对象放入 To Survivor 区；

清空 Eden 和 From Survivor 分区；

From Survivor 和 To Survivor 分区交换，From Survivor 变 To Survivor，To Survivor 变 From Survivor。

每次在 From Survivor 到 To Survivor 移动时都存活的对象，年龄就 +1，当年龄到达 15（默认配置是 15）时，升级为老生代。大对象也会直接进入老生代。

老生代当空间占用到达某个值之后就会触发全局垃圾收回，一般使用标记整理的执行算法。以上这些循环往复就构成了整个分代垃圾回收的整体执行流程。

 

**内存分配策略**

**简述java****内存分配与回收策率以及Minor GC****和Major GC****？**

所谓自动内存管理，最终要解决的也就是内存分配和内存回收两个问题。前面我们介绍了内存回收，这里我们再来聊聊内存分配。

对象的内存分配通常是在 Java 堆上分配（随着虚拟机优化技术的诞生，某些场景下也会在栈上分配，后面会详细介绍），对象主要分配在新生代的 Eden 区，如果启动了本地线程缓冲，将按照线程优先在 TLAB 上分配。少数情况下也会直接在老年代上分配。总的来说分配规则不是百分百固定的，其细节取决于哪一种垃圾收集器组合以及虚拟机相关参数有关，但是虚拟机对于内存的分配还是会遵循以下几种「普世」规则：

**对象优先在 Eden** **区分配**

多数情况，对象都在新生代 Eden 区分配。当 Eden 区分配没有足够的空间进行分配时，虚拟机将会发起一次 Minor GC。如果本次 GC 后还是没有足够的空间，则将启用分配担保机制在老年代中分配内存。

这里我们提到 Minor GC，如果你仔细观察过 GC 日常，通常我们还能从日志中发现 Major GC/Full GC。

Minor GC 是指发生在新生代的 GC，因为 Java 对象大多都是朝生夕死，所有 Minor GC 非常频繁，一般回收速度也非常快；

Major GC/Full GC 是指发生在老年代的 GC，出现了 Major GC 通常会伴随至少一次 Minor GC。Major GC 的速度通常会比 Minor GC 慢 10 倍以上。

**大对象直接进入老年代**

所谓大对象是指需要大量连续内存空间的对象，频繁出现大对象是致命的，会导致在内存还有不少空间的情况下提前触发 GC 以获取足够的连续空间来安置新对象。

前面我们介绍过新生代使用的是标记-清除算法来处理垃圾回收的，如果大对象直接在新生代分配就会导致 Eden 区和两个 Survivor 区之间发生大量的内存复制。因此对于大对象都会直接在老年代进行分配。

**长期存活对象将进入老年代**

虚拟机采用分代收集的思想来管理内存，那么内存回收时就必须判断哪些对象应该放在新生代，哪些对象应该放在老年代。因此虚拟机给每个对象定义了一个对象年龄的计数器，如果对象在 Eden 区出生，并且能够被 Survivor 容纳，将被移动到 Survivor 空间中，这时设置对象年龄为 1。对象在 Survivor 区中每「熬过」一次 Minor GC 年龄就加 1，当年龄达到一定程度（默认 15） 就会被晋升到老年代。