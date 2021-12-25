##  01 字节流

## 1.1 FileInputStream:	字节输入流

## 1.2 FileOutputStream:   字节输出流

##### 3.10字节流-定义小数组拷贝【重点】（视频23）（8‘’）

 1.方法

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| public int read(byte b[]) throws IOException                 | 从输入流中读出8192个字节到缓冲数组中，再从缓冲数组中取出数组b.length个字节到数组b中 |
| public void write(byte b[], int off, int len) throws IOException | 将数组b中的元素，从下标0开始，向缓冲数组中写入len个字节，当缓冲数组满时，一次性写入目的地 |

2.代码实现

```java
public class Class06_flag {
    /**
     * throws   底层抛给上层, 由顶层处理   dao 抛
     * try () catch(){}     顶层try()catch(){}    自己处理异常
     * throw    创建一个异常
     */
    public static void main(String[] args) throws IOException {
        FileInputStream fis = new FileInputStream("E:\\000AAB_class_heima\\Java-se-advance\\my-note\\mine.md");
        FileOutputStream fos = new FileOutputStream("java-se-advanced\\c.txt");
        //method1(fis, fos);  //单字符复制字节流

        byte[] chars = new byte[1024];
        int len;
        //fis.read(byte[] b) -> 返回的是获取到字符的长度
        while((len = fis.read(chars)) != -1){
            //write(byte[] b,off,int len) = write(读写的数组,起点,读写长度)
            fos.write(chars,0,len);
        }
        fis.close();
        fos.close();
    }

    private static void method1(FileInputStream fis, FileOutputStream fos) throws IOException {
        int read;
        //fis.read() -> 返回的是读取的字符
        while ((read = fis.read()) != -1){
            fos.write(read);
        }
        fis.close();
        fos.close();
    }
}
```



## 4. 字节缓冲流(buff 增强)

#### **BufferedOutputStream**: 字节缓冲输出流

#### **BufferedIntputStream**: 字节缓冲输入流

| 方法名                                 | 说明                   |
| -------------------------------------- | ---------------------- |
| BufferedOutputStream(OutputStream out) | 创建字节缓冲输出流对象 |
| BufferedInputStream(InputStream in)    | 创建字节缓冲输入流对象 |

4.3字节缓冲流一次读写一个字节数组【重点 】（视频27）（6‘’）

 1.方法

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| public int read(byte b[]) throws IOException                 | 从输入流中读出8192个字节到缓冲数组中，再从缓冲数组中取出数组b.length个字节到数组b中 |
| public void write(byte b[], int off, int len) throws IOException | 将数组b中的元素，从下标0开始，向缓冲数组中写入len个字节，当缓冲数组满时，一次性写入目的地 |

2.代码实现

```java
package com.itheima.output;

import java.io.*;

public class OutputDemo12 {
    public static void main(String[] args) throws IOException {
        //缓冲流结合数组，进行文件拷贝

        //创建一个字节缓冲输入流
        BufferedInputStream bis = new BufferedInputStream(new FileInputStream("bytestream\\a.avi"));

        //创建一个字节缓冲输出流
        BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("bytestream\\copy.avi"));

        byte [] bytes = new byte[1024];
        int len;
        while((len = bis.read(bytes)) != -1){
            bos.write(bytes,0,len);
        }

        bis.close();
        bos.close();
    }
}
```

