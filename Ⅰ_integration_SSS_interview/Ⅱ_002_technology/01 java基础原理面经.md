           

## 1. String、StringBuffer、StringBuilder 三者区别是什么

**可变性：**

String类中使用字符数组保存字符串，private final char value[]，所以string对象是不可变的。

StringBuilder与StringBuffer都继承自AbstractStringBuilder类，在AbstractStringBuilder中也是使用字符数组保存字符串，char[] value，这两种对象都是可变的。

**线程安全性**：

String中的对象是不可变的，也就可以理解为常量，线程安全。

StringBuffer对方法加了同步锁或者对调用的方法加了同步锁（Synchronized），所以是线程安全的。

StringBuilder并没有对方法进行加同步锁，所以是非线程安全的。

**性能**：

每次对String 类型进行改变的时候，都会生成一个新的String对象，然后将指针指向新的String 对象。

StringBuffer每次都会对StringBuffer对象本身进行操作，而不是生成新的对象并改变对象引用。

相同情况下使用StirngBuilder 相比使用StringBuffer 仅能获得10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

对于三者使用的总结

如果要操作少量的数据用 = String

单线程操作字符串缓冲区 下操作大量数据 = StringBuilder

多线程操作字符串缓冲区 下操作大量数据 = StringBuffer

## 1.1、\=\=和equals的区别？

在基本数据类型间的比较是比较值是否相同，用在引用数据类型是比较内存地址是否相同；

equals一般都是重写了equals()方来来对两个对象的内容比较，如果不重写，使用的就是Object中的equals方法，比较的是对象的内存地址。

## 1.2、你了解String常量池吗？

String常量池就是“字符串常量池”，位于JVM的堆内存中，专门用来存储字符串常量，可以提高内存的使用率，避免开辟多块空间存储相同的字符串，在创建字符串时 JVM 会首先检查字符串常量池，如果该字符串已经存在池中，则返回它的引用，如果不存在，则实例化一个字符串放到池中，并返回其引用。

## 1.3、String是不可变的吗？

String中的内容不可变。一但在常量池中创建，就无法直接修改。但是String的指针可以改变，从一个字符串的内存地址转为指向另一个字符串的内存地址，看起来像是内容被修改了。