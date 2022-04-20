### 01 常用转换符
![[Pasted image 20220415085603.png]]

### 02 测试举例
```java
{
    //%s %c
    String str;
    str = String.format("Hello %s%c","world",'!');
    System.out.println(str);
    //%b
    str = String.format("%b", 10>3);
    System.out.println(str);
    str = String.format("%b", 2>=3);
    System.out.println(str);
    //%d %x %o
    str = String.format("十进制：%d", 10);
    System.out.println(str);
    str = String.format("十六进制：%x", 10);
    System.out.println(str);
    str = String.format("八进制：%o", 10);
    System.out.println(str);
    //%f %a %g %e
    str = String.format("浮点数：%f", 3.14159);
    System.out.println(str);
    str = String.format("十六进制浮点数：%a", 3.14159);
    System.out.println(str);
    str = String.format("通用浮点类型：%g", 3.1415926);
    System.out.println(str);
    str = String.format("指数形式：%e", 3.14159);
    System.out.println(str);
    //%h %% %n 
    str = String.format("散列码：%h", "123456");
    System.out.println(str);
    str = String.format("百分之九十：%d%%", 90);
    System.out.println(str);
    str = String.format("测试到此结束！%n");
    System.out.println(str);
}
```