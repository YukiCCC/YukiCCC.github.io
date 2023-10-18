---
title: Java面试题-二(JVM)
date: 2023-09-18 13:14:23
tags: 面试技巧
category: Java
---
## JVM 有哪些组成部分

- JVM 是运行在操作系统之上的，它与硬件没有直接的交互

![img](https://raw.githubusercontent.com/YukiCCC/figure/main/690102-20160726145503372-1841575655.png)

- 如下图所示：

![img](https://raw.githubusercontent.com/YukiCCC/figure/main/690102-20160726145530263-378108880.png)

整个JVM 分为四部分：

1.  Class Loader 类加载器 : 类加载器的作用是加载类文件到内存

2.  Execution Engine 执行引擎: 执行引擎也叫做解释器(Interpreter) ，负责解释命令，提交操作系统执行。

3.  Native Interface 本地接口

- 本地接口的作用是融合不同的编程语言为Java 所用，它的初衷是融合C/C++ 程序，Java 诞生的时候是C/C++ 横行的时候，要想立足，必须有一个聪明的、睿智的调用C/C++ 程序，于是就在内存中专门开辟了一块区域处理标记为native 的代码，它的具体做法是Native Method Stack 中登记native 方法，在Execution Engine 执行时加载native libraies 。目前该方法使用的是越来越少了，除非是与硬件有关的应用，比如通过Java 程序驱动打印机，或者Java 系统管理生产设备，在企业级应用中已经比较少见，因为现在的异构领域间的通信很发达，比如可以使用Socket 通信，也可以使用Web Service 等等。

4. Runtime data area 运行数据区

- 运行数据区是整个JVM 的重点。我们所有写的程序都被加载到这里，之后才开始运行，Java 生态系统如此的繁荣，得益于该区域的优良自治。

## JVM加载class文件的原理机制 

- Java中的所有类，都需要由类加载器装载到JVM中才能运行。类加载器本身也是一个类，而它的工作就是把class文件从硬盘读取到内存中。在写程序的时候，我们几乎不需要关心类的加载，因为这些都是隐式装载的，除非我们有特殊的用法，像是反射，就需要显式的加载所需要的类。

- 类装载方式，有两种 

  - 1.隐式装载， 程序在运行过程中当碰到通过new 等方式生成对象时，隐式调用类装载器加载对应的类到jvm中
  - 2.显式装载， 通过class.forname()等方法，显式加载需要的类 

- Java类的加载是动态的，它并不会一次性将所有类全部加载后再运行，而是保证程序运行的基础类(像是基类)完全加载到jvm中，至于其他类，则在需要的时候才加载。这当然就是为了节省内存开销。

- Java的类加载器有三个，对应Java的三种类:（java中的类大致分为三种：  1.系统类  2.扩展类 3.由程序员自定义的类 ）

  -- Bootstrap Loader // 负责加载**系统类** (指的是内置类，像是String，对应于C#中的System类和C/C++标准库中的类)
        | 
       \- - ExtClassLoader  // 负责加载**扩展类**(就是继承类和实现类)
               | 
             \- - AppClassLoader  // 负责加载应用类(**程序员自定义的类**)

- 原理机制

  1. 装载:查找和导入class文件;

  2. 连接:

     (1)检查:检查载入的class文件数据的正确性;

     (2)准备:为类的静态变量分配存储空间;

     (3)解析:将符号引用转换成直接引用(这一步是可选的)

  3. 初始化:初始化静态变量，静态代码块。

## 常见的 jvm 调优的参数有哪些?

（1）-Xms20M

表示设置JVM启动内存的最小值为20M，必须以M为单位

（2）-Xmx20M

表示设置JVM启动内存的最大值为20M，必须以M为单位。将-Xmx和-Xms设置为一样可以避免JVM内存自动扩展。大的项目-Xmx和-Xms一般都要设置到10G、20G甚至还要高

（3）-verbose:gc

表示输出虚拟机中GC的详细情况

（4）-Xss128k

表示可以设置虚拟机栈的大小为128k

（5）-Xoss128k

表示设置本地方法栈的大小为128k。不过HotSpot并不区分虚拟机栈和本地方法栈，因此对于HotSpot来说这个参数是无效的

（6）-XX:PermSize=10M

表示JVM初始分配的永久代（方法区）的容量，必须以M为单位

（7）-XX:MaxPermSize=10M

表示JVM允许分配的永久代（方法区）的最大容量，必须以M为单位，大部分情况下这个参数默认为64M

（8）-Xnoclassgc

表示关闭JVM对类的垃圾回收

（9）-XX:+TraceClassLoading

表示查看类的加载信息

（10）-XX:+TraceClassUnLoading

表示查看类的卸载信息

（11）-XX:NewRatio=4

表示设置 年轻代（包括Eden和两个Survivor区）/老年代 的大小比值为1：4，这意味着年轻代占整个堆的1/5

（12）-XX:SurvivorRatio=8

表示设置2个Survivor区：1个Eden区的大小比值为2:8，这意味着Survivor区占整个年轻代的1/5，这个参数默认为8

（13）-Xmn20M

表示设置年轻代的大小为20M

（14）-XX:+HeapDumpOnOutOfMemoryError

表示可以让虚拟机在出现内存溢出异常时Dump出当前的堆内存转储快照

（15）-XX:+UseG1GC

表示让JVM使用G1垃圾收集器

（16）-XX:+PrintGCDetails

表示在控制台上打印出GC具体细节

（17）-XX:+PrintGC

表示在控制台上打印出GC信息

（18）-XX:PretenureSizeThreshold=3145728

表示对象大于3145728（3M）时直接进入老年代分配，这里只能以字节作为单位

（19）-XX:MaxTenuringThreshold=1

表示对象年龄大于1，自动进入老年代,如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象在年轻代的存活时间，增加在年轻代被回收的概率。

（20）-XX:CompileThreshold=1000

表示一个方法被调用1000次之后，会被认为是热点代码，并触发即时编译

（21）-XX:+PrintHeapAtGC

表示可以看到每次GC前后堆内存布局

（22）-XX:+PrintTLAB

表示可以看到TLAB的使用情况

（23）-XX:+UseSpining

开启自旋锁

（24）-XX:PreBlockSpin

更改自旋锁的自旋次数，使用这个参数必须先开启自旋锁

（25）-XX:+UseSerialGC

表示使用jvm的串行垃圾回收机制，该机制适用于丹cpu的环境下

（26）-XX:+UseParallelGC

表示使用jvm的并行垃圾回收机制，该机制适合用于多cpu机制，同时对响应时间无强硬要求的环境下，使用-XX:ParallelGCThreads=设置并行垃圾回收的线程数，此值可以设置与机器处理器数量相等。

（27）-XX:+UseParallelOldGC

表示年老代使用并行的垃圾回收机制

（28）-XX:+UseConcMarkSweepGC

表示使用并发模式的垃圾回收机制，该模式适用于对响应时间要求高，具有多cpu的环境下

（29）-XX:MaxGCPauseMillis=100

设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值。

（30）-XX:+UseAdaptiveSizePolicy

设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低响应时间或者收集频率等，此值建议使用并行收集器时，一直打开

## 说一下jvm有哪些垃圾回收器

- 先图解虚拟机所包含的收集器：

![img](https://img2018.cnblogs.com/blog/1326194/201810/1326194-20181017145352803-1499680295.png)

图中展示了7种作用于不同分代的收集器，3个属于年轻代、3个属于年老代，G1属于横跨年轻代和年老代的算法。

1. **新生代收集器**：Serial、ParNew、Parallel Scavenge

2. **老年代收集器**：CMS、Serial Old、Parallel Old

3. **整堆收集器**： G1

- 几个相关概念：
  - **并行收集**：指多条垃圾收集线程并行工作，但此时用户线程仍处于等待状态。
  - **并发收集**：指用户线程与垃圾收集线程同时工作（不一定是并行的可能会交替执行）。用户程序在继续运行，而垃圾收集程序运行在另一个CPU上。

  - **吞吐量**：即CPU用于运行用户代码的时间与CPU总消耗时间的比值（吞吐量 = 运行用户代码时间 / ( 运行用户代码时间 + 垃圾收集时间 )）。例如：虚拟机共运行100分钟，垃圾收集器花掉1分钟，那么吞吐量就是99%

### Serial 收集器

Serial收集器是最基本的、发展历史最悠久的收集器。

**特点：**单线程、简单高效（与其他收集器的单线程相比），对于限定单个CPU的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程手机效率。收集器进行垃圾回收时，必须暂停其他所有的工作线程，直到它结束（Stop The World）。

**应用场景**：适用于Client模式下的虚拟机。

**Serial / Serial Old收集器运行示意图**

**![img](https://img2018.cnblogs.com/blog/1326194/201810/1326194-20181017164151556-1187071653.png)**

 

### ParNew收集器

ParNew收集器其实就是Serial收集器的多线程版本。

除了使用多线程外其余行为均和Serial收集器一模一样（参数控制、收集算法、Stop The World、对象分配规则、回收策略等）。

**特点**：多线程、ParNew收集器默认开启的收集线程数与CPU的数量相同，在CPU非常多的环境中，可以使用-XX:ParallelGCThreads参数来限制垃圾收集的线程数。

　　　和Serial收集器一样存在Stop The World问题

**应用场景**：ParNew收集器是许多运行在Server模式下的虚拟机中首选的新生代收集器，因为它是除了Serial收集器外，唯一一个能与CMS收集器配合工作的。

**ParNew/Serial Old组合收集器运行示意图如下：**

 ![img](https://img2018.cnblogs.com/blog/1326194/201810/1326194-20181017170542230-1929674942.png)

 

### Parallel Scavenge 收集器

与吞吐量关系密切，故也称为吞吐量优先收集器。

**特点**：属于新生代收集器也是采用复制算法的收集器，又是并行的多线程收集器（与ParNew收集器类似）。

该收集器的目标是达到一个可控制的吞吐量。还有一个值得关注的点是：GC自适应调节策略（与ParNew收集器最重要的一个区别）

**GC自适应调节策略**：Parallel Scavenge收集器可设置-XX:+UseAdptiveSizePolicy参数。当开关打开时不需要手动指定新生代的大小（-Xmn）、Eden与Survivor区的比例（-XX:SurvivorRation）、晋升老年代的对象年龄（-XX:PretenureSizeThreshold）等，虚拟机会根据系统的运行状况收集性能监控信息，动态设置这些参数以提供最优的停顿时间和最高的吞吐量，这种调节方式称为GC的自适应调节策略。

Parallel Scavenge收集器使用两个参数控制吞吐量：

- XX:MaxGCPauseMillis 控制最大的垃圾收集停顿时间
- XX:GCRatio 直接设置吞吐量的大小。

### Serial Old 收集器

Serial Old是Serial收集器的老年代版本。

**特点**：同样是单线程收集器，采用标记-整理算法。

**应用场景**：主要也是使用在Client模式下的虚拟机中。也可在Server模式下使用。

Server模式下主要的两大用途（在后续中详细讲解···）：

1. 在JDK1.5以及以前的版本中与Parallel Scavenge收集器搭配使用。
2. 作为CMS收集器的后备方案，在并发收集Concurent Mode Failure时使用。

Serial / Serial Old收集器工作过程图（Serial收集器图示相同）：

**![img](https://img2018.cnblogs.com/blog/1326194/201810/1326194-20181017164151556-1187071653.png)**

### Parallel Old 收集器

是Parallel Scavenge收集器的老年代版本。

**特点**：多线程，采用标记-整理算法。

**应用场景**：注重高吞吐量以及CPU资源敏感的场合，都可以优先考虑Parallel Scavenge+Parallel Old 收集器。

**Parallel Scavenge/Parallel Old收集器工作过程图：**

**![img](https://img2018.cnblogs.com/blog/1326194/201810/1326194-20181017215237165-1960446438.png)**

### CMS收集器

一种以获取最短回收停顿时间为目标的收集器。

**特点**：基于标记-清除算法实现。并发收集、低停顿。

**应用场景**：适用于注重服务的响应速度，希望系统停顿时间最短，给用户带来更好的体验等场景下。如web程序、b/s服务。

**CMS收集器的运行过程分为下列4步：**

**初始标记**：标记GC Roots能直接到的对象。速度很快但是仍存在Stop The World问题。

**并发标记**：进行GC Roots Tracing 的过程，找出存活对象且用户线程可并发执行。

**重新标记**：为了修正并发标记期间因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录。仍然存在Stop The World问题。

**并发清除**：对标记的对象进行清除回收。

 CMS收集器的内存回收过程是与用户线程一起并发执行的。

 CMS收集器的工作过程图：

![img](https://img2018.cnblogs.com/blog/1326194/201810/1326194-20181017221500926-2071899824.png)

CMS收集器的缺点：

- 对CPU资源非常敏感。
- 无法处理浮动垃圾，可能出现Concurrent Model Failure失败而导致另一次Full GC的产生。
- 因为采用标记-清除算法所以会存在空间碎片的问题，导致大对象无法分配空间，不得不提前触发一次Full GC。

### G1收集器

一款面向服务端应用的垃圾收集器。

**特点如下：**

- 并行与并发：G1能充分利用多CPU、多核环境下的硬件优势，使用多个CPU来缩短Stop-The-World停顿时间。部分收集器原本需要停顿Java线程来执行GC动作，G1收集器仍然可以通过并发的方式让Java程序继续运行。

- 分代收集：G1能够独自管理整个Java堆，并且采用不同的方式去处理新创建的对象和已经存活了一段时间、熬过多次GC的旧对象以获取更好的收集效果。

- 空间整合：G1运作期间不会产生空间碎片，收集后能提供规整的可用内存。

- 可预测的停顿：G1除了追求低停顿外，还能建立可预测的停顿时间模型。能让使用者明确指定在一个长度为M毫秒的时间段内，消耗在垃圾收集上的时间不得超过N毫秒。

**G1为什么能建立可预测的停顿时间模型？**

- 因为它有计划的避免在整个Java堆中进行全区域的垃圾收集。G1跟踪各个Region里面的垃圾堆积的大小，在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的Region。这样就保证了在有限的时间内可以获取尽可能高的收集效率。

**G1与其他收集器的区别**：

- 其他收集器的工作范围是整个新生代或者老年代、G1收集器的工作范围是整个Java堆。在使用G1收集器时，它将整个Java堆划分为多个大小相等的独立区域（Region）。虽然也保留了新生代、老年代的概念，但新生代和老年代不再是相互隔离的，他们都是一部分Region（不需要连续）的集合。

**G1收集器存在的问题：**

- Region不可能是孤立的，分配在Region中的对象可以与Java堆中的任意对象发生引用关系。在采用可达性分析算法来判断对象是否存活时，得扫描整个Java堆才能保证准确性。其他收集器也存在这种问题（G1更加突出而已）。会导致Minor GC效率下降。

**G1收集器是如何解决上述问题的？**

- 采用Remembered Set来避免整堆扫描。G1中每个Region都有一个与之对应的Remembered Set，虚拟机发现程序在对Reference类型进行写操作时，会产生一个Write Barrier暂时中断写操作，检查Reference引用对象是否处于多个Region中（即检查老年代中是否引用了新生代中的对象），如果是，便通过CardTable把相关引用信息记录到被引用对象所属的Region的Remembered Set中。当进行内存回收时，在GC根节点的枚举范围中加入Remembered Set即可保证不对全堆进行扫描也不会有遗漏。

**如果不计算维护 Remembered Set 的操作，G1收集器大致可分为如下步骤：**

**初始标记**：仅标记GC Roots能直接到的对象，并且修改TAMS（Next Top at Mark Start）的值，让下一阶段用户程序并发运行时，能在正确可用的Region中创建新对象。（需要线程停顿，但耗时很短。）

**并发标记**：从GC Roots开始对堆中对象进行可达性分析，找出存活对象。（耗时较长，但可与用户程序并发执行）

**最终标记**：为了修正在并发标记期间因用户程序执行而导致标记产生变化的那一部分标记记录。且对象的变化记录在线程Remembered Set Logs里面，把Remembered Set Logs里面的数据合并到Remembered Set中。（需要线程停顿，但可并行执行。）

**筛选回收**：对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间来制定回收计划。（可并发执行）

**G1收集器运行示意图：**

![img](https://img2018.cnblogs.com/blog/1326194/201810/1326194-20181017225802481-709835773.png) 

## 什么是GC？为什么要有GC？

GC（Garbage Collection）是垃圾收集的意思，负责清除对象并释放内存。Java 提供的 GC 功能可以自动检测对象是否超过作用域从而达到自动回收内存的目的，从而防止内存泄漏。

## 讲一下java内存模型

- 在 JDK1.2 之前，Java的内存模型实现总是从主存（即共享内存）读取变量，是不需要进行特别的注意的。
- 而在当前的 Java 内存模型下，线程可以把变量保存本地内存比如机器的寄存器）中，而不是直接在主存中进行读写。这就可能造成一个线程在主存中修改了一个变量的值，而另外一个线程还继续使用它在寄存器中的变量值的拷贝，造成数据的不一致。

![](https://raw.githubusercontent.com/YukiCCC/figure/main/113.1.jpg)

- 要解决这个问题，就需要把变量声明为volatile，这就指示 JVM，这个变量是不稳定的，每次使用它都到主存中进行读取。
  说白了， volatile 关键字的主要作用就是保证变量的可见性然后还有一个作用是防止指令重排序。

![](https://raw.githubusercontent.com/YukiCCC/figure/main/113.2.jpg) 

## Java中有哪几种引用？它们的含义和区别是什么？

从JDK1.2开始，Java中的引用类型分为四种，分别是：

①强引用（StrongReference）
②软引用（SoftRefernce）
③弱引用（WeakReference）
④虚引用（PhantomReference）

1. 强引用-StrongReference
   这种引用是平时开发中最常用的，例如String strong = new String('Strong Reference')，当一个实例对象具有强引用时，垃圾回收器不会回收该对象，当内存不足时，宁愿抛出OutOfMemeryError异常也不会通过回收强引用的对象，因为JVM认为强引用的对象是用户正在使用的对象，它无法分辨出到底该回收哪个，强行回收有可能导致系统严重错误。
2. 软引用-SoftReference
   如果一个对象只有软引用，那么只有当内存不足时，JVM才会去回收该对象，其他情况不会回收。软引用可以结合ReferenceQueue来使用，当由于系统内存不足，导致软引用的对象被回收了，JVM会把这个软引用加入到与之相关联的ReferenceQueue中。

![](https://raw.githubusercontent.com/YukiCCC/figure/main/132.1.png)
当系统内存不足时，触发gc，这个Book就会被回收，reference 将不为null。

3. 弱引用-WeakReference
   只有弱引用的对象，当JVM触发gc时，就会回收该对象。与软引用不同的是，不管是否内存不足，弱引用都会被回收。弱引用可以结合ReferenceQueue来使用，当由于系统触发gc，导致软引用的对象被回收了，JVM会把这个弱引用加入到与之相关联的ReferenceQueue中，不过由于垃圾收集器线程的优先级很低，所以弱引用不一定会被很快回收。下面通过一个主动触发gc的例子来验证此结论。

![](https://raw.githubusercontent.com/YukiCCC/figure/main/132.2.png)
![](https://raw.githubusercontent.com/YukiCCC/figure/main/132.3.png)

当然这不是每次都能复现，因为我们调用System.gc()只是告诉JVM该回收垃圾了，但是它什么时候做还是不一定的，但就我测试来看，只要多写几次System.gc()，复现的概率还是很高的。

4. 虚引用-PhantomReference
   如果一个对象只有虚引用在引用它，垃圾回收器是可以在任意时候对其进行回收的，虚引用主要用来跟踪对象被垃圾回收器回收的活动，当被回收时，JVM会把这个弱引用加入到与之相关联的ReferenceQueue中。与软引用和弱引用不同的是，虚引用必须有一个与之关联的ReferenceQueue，通过phantomReference.get()得到的值为null

   ![](https://raw.githubusercontent.com/YukiCCC/figure/main/132.4.png)

## 什么是双亲委派模式

- 双亲委派模型要求除顶层启动类加载器外其余类加载器都应该有自己的父类加载器；类加载器之间通过复用关系来复用父加载器的代码。
- 双亲委派模型工作工程：
  1.当Application ClassLoader 收到一个类加载请求时，他首先不会自己去尝试加载这个类，而是将这个请求委派给父类加载器Extension ClassLoader去完成。
  2.当Extension ClassLoader收到一个类加载请求时，他首先也不会自己去尝试加载这个类，而是将请求委派给父类加载器Bootstrap ClassLoader去完成。
  3.如果Bootstrap ClassLoader加载失败(在、lib中未找到所需类)，就会让Extension ClassLoader尝试加载。
  4.如果Extension ClassLoader也加载失败，就会使用Application ClassLoader加载。
  5.如果Application ClassLoader也加载失败，就会使用自定义加载器去尝试加载。
  6.如果均加载失败，就会抛出ClassNotFoundException异常。
- 双亲委派模型的实现过程：
  - 实现双亲委派模型的代码都集中在java.lang.ClassLoader的loadClass()方法中：
  - 首先会检查请求加载的类是否已经被加载过；
  - 若没有被加载过：
  - 递归调用父类加载器的loadClass();
  - 父类加载器为空后就使用启动类加载器加载；
  - 如果父类加载器和启动类加载器均无法加载请求，则调用自身的加载功能。
- 双亲委派模型的优点：
  - Java类伴随其类加载器具备了带有优先级的层次关系，确保了在各种加载环境的加载顺序。
    保证了运行的安全性，防止不可信类扮演可信任的类