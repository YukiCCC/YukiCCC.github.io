---
title: Java-IO流
date: 2023-07-01 20:15:14
tags: Java
category: Java
---
## 1. File

### 1.1 File与流

![image-20220427214839278](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220427214839278.png)



![image-20220427215013469](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220427215013469.png)

### 1.2 File练习

#### 例1

```java
/*使用File对象，在C盘创建aaa/bbb/ccc的文件夹，在此文件夹下创建1.txt【C:/aaa/bbb/ccc/1.txt】
 如果该文件存在，删除1.txt
 如果该文件不存在，创建1.txt*/
import java.io.File;
import java.io.IOException;

public class FileTest01 {
    public static void main(String[] args) throws IOException {
        //虚拟 目录 file
        File file = new File("C:\\aaa\\bbb\\ccc\\1.txt");
        if (file.exists()){
            file.delete();
        }else{
            //创建文件夹
            file.getParentFile().mkdirs();
            //创建 文件
            file.createNewFile();
        }
    }
}
```



#### 例2

```java
/*使用File对象  listFiles() 方法
自定义类 implements FilenameFilter
C:\Program Files\Java\jdk1.8.0_321\bin
目录下，所有的 .exe结尾的文件打印出来。*/
import java.io.File;
import java.io.FilenameFilter;
import java.io.IOException;
import java.util.stream.Stream;

public class FileTest02 {
    public static void main(String[] args) throws IOException {
        //虚拟 目录 file
        File file = new File("C:\\Program Files\\Java\\jdk1.8.0_361\\bin");
        String[] ay = file.list((dir, name) -> name.endsWith(".exe"));
        Stream.of(ay).forEach(System.out::println);
    }
}
```



## 2. 节点流

### 2.1 字节流

![image-20220427215437892](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220427215437892.png)

### 2.2 节点流练习

#### 例3

```java
/*使用FileOutputStream 向1.txt文件
写入： A~Z 字符
在 文件 结尾：a~z字符*/
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.OutputStream;

public class FileOutputStreamTest03 {
    public static void main(String[] args) throws IOException {
        //输出流 自动创建文件
        OutputStream os = new FileOutputStream("1.txt",true) ;
        //写 byte 比如：A
        for (int i = 'A'; i <='Z'; i++) {
            os.write(i);
        }

        /*for (int i = 'A'; i <='Z'; i++) {
            os.write(i);
        }*/
        //建议： 可选
        //os.flush();
        os.close();
    }
}
```

#### 例4

```java
/*使用FileInputStream  IDEA具体的JAVA文件 比如： Test1.java
打印该Java文件中所有的内容*/

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;

public class FileInputStreamTest04 {
    public static void main(String[] args) throws IOException {
        //输入流  注意：目标数据源 一定存在的
        InputStream is = new FileInputStream("Test1.java") ;



        //读
        /*while (true){
            int  data = is.read() ;
           //文件结尾
            if (data==-1){
               break;
            }
           System.out.print((char)data);
        }*/
        int data = 0 ;
        while((data = is.read())!=-1){
            System.out.print((char)data);
        }
        is.close();
    }
}
```

### 2.3 字符流

![image-20220427215913839](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220427215913839.png)

### 2.4 节点流

![image-20220427220004274](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220427220004274.png)

#### 输入流

![image-20220427220041383](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220427220041383.png)

![image-20220427220134833](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220427220134833.png)

#### 输出流

![image-20220427220212141](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220427220212141.png)

![image-20220427220233194](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220427220233194.png)

#### 例5

```java
/*使用流 实现把图片从C盘拷贝D盘。*/
import java.io.*;

public class PicCopyTest05 {
    public static void main(String[] args) throws IOException {
        //1.输入流  怼到 目标数据源
        InputStream is = new FileInputStream("D:\\Java180_2\\doc\\day01-html基础\\res\\mm.jpg") ;
        //2.输出流   项目下
        OutputStream os = new FileOutputStream("meimei.jpg") ;
        //3. 读
        while (true){
            int  data = is.read() ;
            //文件结尾
            if (data==-1){
                break;
            }
            //4.写
            os.write(data);
        }

        System.out.println("====图片拷贝结束=========");
    }
}
```



## 3. 处理流

### 3.1 流嵌套

![image-20220427220342981](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220427220342981.png)

### 3.2 处理流

![image-20220427220441730](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220427220441730.png)

### 3.3 调包侠

https://commons.apache.org/proper/commons-io/description.html

![image-20220427225034882](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220427225034882.png)





## 4. 对象流

![image-20220427220539197](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220427220539197.png)

#### 例6

```java
/*使用对象流  实现对对象实现读写功能*/
//写入
import java.io.*;

public class ObjectOutputStreamTest11 {
    public static void main(String[] args) throws IOException {
        //1.对象 创建商品对象  【A.瞬时状态  -- JVM 内存】
        Goods goods = new Goods("G1001", "苹果", 4.5D);

        //2. 写入 4.txt 文件  节点文件流
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("4.txt")) ;
        /*OutputStream os = new FileOutputStream("4.txt") ;
        ObjectOutputStream oos = new ObjectOutputStream(os) ;*/

        //【B.持久状态  -- 文件系统】
        //goods.setName("香蕉");
        //对象流 写入
        oos.writeObject(goods);

        oos.flush();
        oos.close();
        //【C.脱管状态 -- 脱离管理】
        //goods.setName("菠萝");
    }
}

//读取
import java.io.*;

public class ObjectInputStreamTest12 {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        //1.读取
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("4.txt")) ;

        Object o = ois.readObject();

        ois.close();

        System.out.println(o);

    }
}


```
