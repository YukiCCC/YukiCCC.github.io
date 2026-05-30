---
title: Java-捕获异常
date: 2023-07-03 19:09:06
tags: Java
category: Java
---
## 捕获异常

使用 try 和 catch 关键字可以捕获异常。try/catch 代码块放在异常可能发生的地方。
try/catch代码块中的代码称为保护代码，使用 try/catch 的语法如下：
```java
try
{
   // 程序代码
}catch(ExceptionName e1)
{
   //Catch 块
}
```
Catch 语句包含要捕获异常类型的声明。当保护代码块中发生一个异常时，try 后面的 catch 块就会被检查。
如果发生的异常包含在 catch 块中，异常会被传递到该 catch 块，这和传递一个参数到方法是一样。
### 实例:
下面的例子中声明有两个元素的一个数组，当代码试图访问数组的第四个元素的时候就会抛出一个异常。
```java
// 文件名 : ExcepTest.java
import java.io.*;
public class ExcepTest{
 
   public static void main(String args[]){
      try{
         int a[] = new int[2];
         System.out.println("Access element three :" + a[3]);
      }catch(ArrayIndexOutOfBoundsException e){
         System.out.println("Exception thrown  :" + e);
      }
      System.out.println("Out of the block");
   }
}
```
以上代码编译运行输出结果如下
```
Exception thrown  :java.lang.ArrayIndexOutOfBoundsException: 3
Out of the block
```

## 多重捕获块
一个 try 代码块后面跟随多个 catch 代码块的情况就叫多重捕获。
多重捕获块的语法如下所示：
```java
try{
   // 程序代码
}catch(异常类型1 异常的变量名1){
  // 程序代码
}catch(异常类型2 异常的变量名2){
  // 程序代码
}catch(异常类型3 异常的变量名3){
  // 程序代码
}
```
上面的代码段包含了 3 个 catch块。
可以在 try 语句后面添加任意数量的 catch 块。
如果保护代码中发生异常，异常被抛给第一个 catch 块。
如果抛出异常的数据类型与 ExceptionType1 匹配，它在这里就会被捕获。
如果不匹配，它会被传递给第二个 catch 块。
如此，直到异常被捕获或者通过所有的 catch 块。
### 实例:
该实例展示了怎么使用多重 try/catch。
```java
try {
    file = new FileInputStream(fileName);
    x = (byte) file.read();
} catch(FileNotFoundException f) { // Not valid!
    f.printStackTrace();
    return -1;
} catch(IOException i) {
    i.printStackTrace();
    return -1;
}
```
## throws/throw 关键字
在Java中， throw 和 throws 关键字是用于处理异常的。
throw 关键字用于在代码中抛出异常，而 throws 关键字用于在方法声明中指定可能会抛出的异常类型。

### throw 关键字
throw 关键字用于在当前方法中抛出一个异常。
通常情况下，当代码执行到某个条件下无法继续正常执行时，可以使用 throw 关键字抛出异常，以告知调用者当前代码的执行状态。
例如，下面的代码中，在方法中判断 num 是否小于 0，如果是，则抛出一个 IllegalArgumentException 异常。
```java
public void checkNumber(int num) {
  if (num < 0) {
    throw new IllegalArgumentException("Number must be positive");
  }
}
```
### throws 关键字
throws 关键字用于在方法声明中指定该方法可能抛出的异常。当方法内部抛出指定类型的异常时，该异常会被传递给调用该方法的代码，并在该代码中处理异常。
例如，下面的代码中，当 readFile 方法内部发生 IOException 异常时，会将该异常传递给调用该方法的代码。在调用该方法的代码中，必须捕获或声明处理 IOException 异常。
```java
public void readFile(String filePath) throws IOException {
  BufferedReader reader = new BufferedReader(new FileReader(filePath));
  String line = reader.readLine();
  while (line != null) {
    System.out.println(line);
    line = reader.readLine();
  }
  reader.close();
}
```
一个方法可以声明抛出多个异常，多个异常之间用逗号隔开。
例如，下面的方法声明抛出 RemoteException 和 InsufficientFundsException：
```java
import java.io.*;
public class className
{
   public void withdraw(double amount) throws RemoteException,
                              InsufficientFundsException
   {
       // Method implementation
   }
   //Remainder of class definition
}
```
## finally关键字
finally 关键字用来创建在 try 代码块后面执行的代码块。
无论是否发生异常，finally 代码块中的代码总会被执行。
在 finally 代码块中，可以运行清理类型等收尾善后性质的语句。
finally 代码块出现在 catch 代码块最后，语法如下：
```java
try{
  // 程序代码
}catch(异常类型1 异常的变量名1){
  // 程序代码
}catch(异常类型2 异常的变量名2){
  // 程序代码
}finally{
  // 程序代码
}
```
### 实例:
```java
public class ExcepTest{
  public static void main(String args[]){
    int a[] = new int[2];
    try{
       System.out.println("Access element three :" + a[3]);
    }catch(ArrayIndexOutOfBoundsException e){
       System.out.println("Exception thrown  :" + e);
    }
    finally{
       a[0] = 6;
       System.out.println("First element value: " +a[0]);
       System.out.println("The finally statement is executed");
    }
  }
}
```
以上实例编译运行结果如下：
```
Exception thrown  :java.lang.ArrayIndexOutOfBoundsException: 3
First element value: 6
The finally statement is executed
```
注意下面事项：

1.catch 不能独立于 try 存在。
2.在 try/catch 后面添加 finally 块并非强制性要求的。
3.try 代码后不能既没 catch 块也没 finally 块。
4.try, catch, finally 块之间不能添加任何代码。