---
title: JDK8新特性
date: 2023-08-22 16:30:50
tags: Java
category: Java
---

## 一、类型推断的改进

类型推断(推导) 是 JDK7的特性，JDK8中做了改进。

## 二、默认方法

JDK8 新增了接口的默认方法，默认方法就是一个在接口中有具体实现的方法，即非抽象方法。

定义默认方法时需要使用 default 关键字进行修饰。

也可以定义静态方法，且不需要使用 default 修饰。

## 三、Lambda表达式

Lambda 表达式是一个匿名函数，允许把函数作为一个方法的参数。

Lambda 表达式可以认为是对匿名内部类的一种简化，使代码变的更简洁紧凑。

但并不是所有的匿名内部类都可以简化为Lambda表达式，只有**函数式接口的匿名内部类**才可以使用Lambda表达式来进行简化

> 函数式接口：就是接口中只有一个抽象方法，Lambda表达式正好是针对这个唯一的抽象方法使用的

Lambda表达式的语法：

```java
(parameters) -> expression
或
(parameters) -> { statement; }
```

注意:

+ **参数类型是可选的** : 可以省略参数的类型声明，由编译器统一识别，进行类型推断
+ **参数小括号是可选的** : 当方法只有一个参数时，可以省略参数小括号
+ **方法体大括号是可选的**: 当方法体只包含一条语句时，可以省略方法体大括号
+ **return关键字是可选的** : 当方法体只包含一条语句，且需要返回该语句的执行结果时，可以省略return

## 四、方法引用

方法引用是一种**语法糖**操作，作用是简化Lambda在调用已经存在的方法时的表达式。

方法引用可以直接引用已有Java类或对象的方法或构造方法，使用一对冒号 `::` 

方法引用的几种类型：

1. 静态方法引用（类名::静态方法名）
2. 实例方法引用（对象名::实例方法名）
3. 类的实例方法引用（类名::实例方法名）
4. 构造方法引用（类名::new）

## 五、函数式接口

函数式接口 Functional Interface：就是有且只有一个抽象方法的接口，但可以有多个非抽象方法。

函数式接口（的实现类的对象）可以隐式转换为Lambda表达式，进行简化。

Java 8 提供了一个特殊的注解**@FunctionalInterface**，用于标注一个接口是函数式接口

```java
/**
 * JDK8之前的函数式接口
 */
java.lang.Runnable
java.lang.Comparable<T>
java.util.Comparator<T>
java.io.FileFilter
java.io.FilenameFilter
java.lang.reflect.InvocationHandler
......
  
/**
 * JDK8新增的函数式接口，主要在java.util.function包中
 */  
// 消费型接口
java.util.function.Consumer<T> 
	void accept(T t); // 接收一个参数, 无返回

// 供给型接口
java.util.function.Supplier<T>
	T get(); // 无参数, 返回一个结果

// 函数型接口
java.util.function.Function<T, R>
	R apply(T t); // 接收一个参数, 返回一个结果

// 断言型接口
java.util.function.Predicate<T>
	boolean test(T t); // 接收一个参数, 返回一个布尔值的结果
```

## 六、Stream API

### 1. 简介

Java 8 API 添加了一个新的抽象称为Stream流，它可以让你以一种声明的方式来处理数据。

Stream主要用于集合操作，支持链式编程，极大的简化了代码。

Stream将要处理的元素集合看作为一种流，数据在流的管道中传输，可以在管道的节点上对元素进行处理，如：筛选、排序、聚合等。

元素流在管道中经过 **中间操作** 的处理，最后由 **终止操作** 得到前面处理的结果。

Stream流的操作步骤：

1. 获取Stream对象
2. 中间操作（实现要做的数据处理操作）
3. 终止操作

### 2. 获取Stream对象

三种方式：

1. 通过Collection接口中的stream方法或parallelStream方法获取集合的流
2. 通过Arrays类中的stream方法获取数组的流（串行流）
3. 通过Stream接口中的of方法获取一个或多个元素的流（串行流）

### 3. 基本使用

#### 3.1 Stream特点

Stream不会存储元素

Stream不会改变源对象，它会返回一个持有结果的新Stream对象。

Stream中间操作都会返回一个持有结果的新Stream对象，可以对新的Stream继续执行其他的中间操作，这样的多个连续操作，可以串联成一个管道。

Stream的中间操作是延迟执行的，需要执行终止操作，才会真正的执行中间操作。

#### 3.2 惰性求值

多个中间操作可以连起来，形成一个流水线。除非流水线上触发了终止操作，否则中间操作不会被执行。

只有当发生终止操作时，才会一次性处理全部的操作，这种情况称为 **惰性求值** 。

#### 3.3 内部迭代

以前对集合遍历都是通过 `Iterator` 或 `For-Each` 的方式，这种显式的在集合外部进行迭代的方式称为 **外部迭代**。

在集合内部进行迭代的方式称为 **内部迭代**。例如：集合的forEach方法、Stream的操作的forEach方法（是一个终止操作）。

### 4. 中间操作

#### 4.1 筛选与切片

`filter(Predicate predicate)` 从流中获取符合条件的元素

`limit(long maxSize)` 截断流，使其元素不超过指定数量

`skip(long n)` 跳过多个元素，返回一个扔掉了 n 个元素的流，如果流中的元素不足 n，则返回一个空的流

`distinct()` 通过流中的元素的 hashCode和equals方法进行判断是否重复

#### 4.2 映射

`map(Function<T, R> mapper)` 将回调方法的操作应用到每一个元素上，将其映射成一个新的元素。

`flatMap(Function<T, Stream<R>> mapper)` 将流中的每一个元素都转换成一个流，然后将所有的流连成一个新的流。

`mapToInt(ToIntFunction<T> mapper)`  将流中的每一个元素转换成int类型的元素，然后返回包含所有转换后值的流

`mapToLong(ToLongFunction<T> mapper)` 将流中的每一个元素转换成long类型的元素，然后返回包含所有转换后值的流

`mapToDouble(ToDoubleFunction<T> mapper)`  将流中的每一个元素转换成double类型的元素，然后返回包含所有转换后值的流

`peek(Consumer<? super T> action)` 作用与map()类似，但无需返回值，属于消费型接口

#### 4.3 排序

`sorted()` 返回一个新的流，其中元素按照自然顺序升序排序。

`sorted(Comparator<T> comparator)` 根据比较器指定的排序规则，对流中的元素进行排序。

### 5. 终止操作

#### 5.1 查找与匹配

`forEach(Consumer c)`遍历流中的数据,该方法接收一个Consumer函数式接口，会将每一个流的元素交给函数

`allMatch(Predicate<T> predicate)` 元素全部满足要求时，才会返回 true，否则返回 false（短路操作）

`anyMatch(Predicate<T> predicate)` 元素只要有一个满足要求，就会返回 true，否则返回 false（短路操作）

`noneMatch(Predicate<? super T> predicate)` 元素全部不满足要求时，才会返回 true，否则返回 false（短路操作）

`findFirst()` 取出流中第一个元素，会把结果封装成 Optional 类型的对象。

`findAny()` 取出符合要求的一个元素(任意的、只要满足要求即可)。

`count()` 统计流中的元素个数。

`max(Comparator<T> comparator)` 找最大的元素。

`min(Comparator<T> comparator)` 找最小的元素。

#### 5.2 归约

归约：将流中的元素反复结合起来，得到一个值。

`reduce(BinaryOperator<T> accumulator)` 对流中的元素进行累计操作。前一次的操作结果会作为下一次操作的参数，最终返回一个 使用 Optional 封装的累计结果。

`reduce(T identity, BinaryOperator<T> accumulator)` 对流中的元素进行累计操作。根据初始值 identity 进行累计操作，最终返回一个和流中元素类型一致的结果。

#### 5.3 收集

`collect(Collector<T, A, R> collector)`  收集处理的结果，将流转换成指定集合，一般通过`Collectors`工具类指定集合类型

## 七、Optional 类型

Optional 是一个容器，可以保存 T 类型的值，或仅仅保存null。

Optional 主要的作用是用来解决空指针异常的。

Optional 提供了很多有用的方法，这样程序员就不用显式进行空值判断了。
