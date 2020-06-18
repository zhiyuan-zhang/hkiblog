---
title: 浅谈jvm
date: 2020/4/10
categories:
  - java
comments: true
abbrlink: 49197
img: /static/jvm.jpeg
---



### GC的基础知识

#### 1.什么是垃圾

简单来讲**没有任何引用指向的一个对象或者多个对象（循环引用）就是垃圾.**

引用又分为强引用，软引用，弱引用，虚引用。 

- 强引用（Strong Reference）：这类引用是Java程序中最普遍的。只要强引用还存在，垃圾收集器就永远不会回收掉被引用的对象,当内存不够的时候只会抛出OOM异常

```java
Object  obj = new Object();
```

- 软引用（Soft Reference）：它用来描述一些可能还有用，但并非必须的对象。在**系统内存不够用时**，这类引用关联的对象将被垃圾收集器回收。JDK1.2之后提供了SoftReference类来实现软引用。

```java
SoftReference  soft = new SoftReference(user);
```

- 弱引用（Weak Reference）：它也是用来描述非须对象的，但它的强度比软引用更弱些，被弱引用关联的对象只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时，**无论当前内存是否足够，都会回收掉**只被弱引用关联的对象。在JDK1.2之后，提供了WeakReference类来实现弱引用。

 

- 虚引用（Phantom Reference）：最弱的一种引用关系，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的**是希望能在这个对象被收集器回收时收到一个系统通知**。JDK1.2之后提供了PhantomReference类来实现虚引用。



#### 2.如何寻找垃圾


1. 引用计数

   java没有在用这种算法,它需要单独维护一个引用计数器 目前OkHttp,netty在用故不在本章节讨论范围.

2. 根可达算法

所谓根集(Root Set)就是正在执行的Java程序可以访问的引用变量（注意：不是对象）的集合(包括局 部变量、参数、类变量)，程序可以使用引用变量访问对象的属性和调用对象的方法。

 这种算法的基本思路：

（1）通过一系列名为“GC Roots”的对象作为起始点，寻找对应的引用节点。

（2）找到这些引用节点后，从这些节点开始向下继续寻找它们的引用节点。 

（3）重复（2）。

（4）搜索所走过的路径称为引用链，当一个对象到GC Roots没有任何引用链相连时，就证明此对象是不可用的。



常见的根节点(起始点)
（1）虚拟机栈中引用的对象（栈帧中的本地变量表）；
（2）方法区中的常量引用的对象；final 常量名=值;
（3）方法区中的类静态属性引用的对象 static；
（4）本地方法栈中JNI（Native方法）的引用对象。
（5）活跃线程。


在根搜索算法中，要真正宣告一个对象死亡，至少要经历两次标记过程：

1.如果对象在进行根搜索后发现没有与GC Roots相连接的引用链，那它会被第一次标记并且进行一次筛选。筛选的条件是此对象是否有必要执行 finalize（）方法。当对象没有覆盖finalize（）方法，或finalize（）方法已经被虚拟机调用过，虚拟机将这两种情况都视为没有必要执行。

2.如果该对象被判定为有必要执行finalize（）方法，那么这个对象将会被放置在一个名为F-Queue队列中，并在稍后由一条由虚拟机自动建立的、低优先级的Finalizer线程去执行finalize（）方法。finalize（）方法是对象逃脱死亡命运的最后一次机会（因为一个对象的finalize（）方法最多只会被系统自动调用一次），稍后GC将对F-Queue中的对象进行第二次小规模的标记，如果要在finalize（）方法中成功拯救自己，只要在finalize（）方法中让该对象重新引用链上的任何一个对象建立关联即可。而如果对象这时还没有关联到任何链上的引用，那它就会被回收掉。


#### 3.常见的垃圾回收算法

1. 标记清除 - 位置不连续 产生碎片
2. 拷贝算法 - 没有碎片，浪费空间
3. 标记压缩 - 没有碎片，效率偏低

#### 4.JVM内存分代模型（用于分代垃圾回收算法）


1. 新生代 + 老年代 + 永久代（1.7）/ 元数据区(1.8) Metaspace
   1. 永久代 元数据 - Class
   2. 永久代必须指定大小限制 ，元数据可以设置，也可以不设置，无上限（受限于物理内存）
   3. 字符串常量 1.7 - 永久代，1.8 - 堆
   4. MethodArea逻辑概念 - 永久代、元数据

2. 新生代 = Eden + 2个suvivor区 
   1. YGC回收之后，大多数的对象会被回收，活着的进入 s0
   2. 再次YGC，活着的对象eden + s0 -> s1
   3. 再次YGC，eden + s1 -> s0
   4. 年龄足够 -> 老年代 （15 CMS 6）
   5. suvivor区装不下 -> 老年代

3. 老年代
   1. 顽固分子
   2. 老年代满了FGC Full GC
4. GC Tuning (Generation)
   1. 尽量减少FGC
   2. MinorGC = YGC
   3. MajorGC = FGC

#### 5.常见的垃圾回收器

1. Serial 年轻代 串行回收
2. PS 年轻代 并行回收
3. ParNew 年轻代 配合CMS的并行回收
4. SerialOld 
5. ParallelOld
6. ConcurrentMarkSweep 老年代 并发的， 垃圾回收和应用程序同时运行，降低STW的时间(200ms)
7. G1(10ms)
8. ZGC (1ms) PK C++
9. Shenandoah
10. Eplison

1.8默认的垃圾回收：PS + ParallelOld

#### 6.JVM调优第一步，了解生产环境下的垃圾回收器组合






### jvm运行时数据分析

在区域中一般分为五块


#### 运行时数据区域
1. 计数器
	通过改变这个计数器的值来获取下一条需要执行的字节码,包括判断
	循环,跳转,异常处理
	
	2. 方法区(method Area)
		
		该区域用来存放我们生成的各种对象信息
		
	
3. 虚拟机栈(VM stack)
	每个类 都会在执行的时候创建一个栈帧(javap -c -s xxx.class)来存放关于这个类的所有信息.
	包括方法内的局部变量,以及各种操作帧数, 动态链接(各种引用)及方法的出口返回值等等。每个方法的调用到结束意味着虚拟机的栈桢的入栈到出栈,
	
4. 堆(heap) 
	在遇到new指令后 我们会根据相对应的内存引用 来找如果有就引用  没有的话就执行类加载 Java 类加载过程那一套东西
	
5. 本地方法栈(native mathod stack)
	存放本地原生调用方法和虚拟机栈一样只不过内容是原生计算机内的C方法native是与C++联合开发的时候用的 所以一般开发不会用到

前面在堆中说了类引用会检查是否已经加载过当前需要的类 

如果没有加载则会去加载 那么类的加载过程就是下面要说的


当程序用到某个类但这个类还没有加载到内存中,jvm会通过加载,连接,初始化来对这个类进行初始化

1. 加载

jvm通过类的全类名来获取字节流 (也就是.class文件) 然后根据字节流创建class对象  创建完成后写入到内存中,然后放在运行时区域的方法区内(method Area)  然后在堆(heap)中创建出对应的Class对象

2. 链接

分为三部分 

验证语法  满足jvm虚拟机规范 

准备阶段  为类的静态static 分配内存 设置默认值

解析阶段  将符号引用替换为内存引用(直接引用)

3. 初始化

将static修饰的方法或者变量进行初始化,如果有父级引用优先加载父类 

那么这个类的加载机制又是什么 它是通过什么来加载的

采用的双亲委派机制

当加载某个类的时候先去询问父类节点是否可以加载 

这样既避免了重复加载也防止了注入还提高了效率


### 常见垃圾回收器组合参数设定：(1.8)

* -XX:+UseSerialGC = Serial New (DefNew) + Serial Old
  
  * 小型程序。默认情况下不会是这种选项，HotSpot会根据计算及配置和JDK版本自动选择收集器
* -XX:+UseParNewGC = ParNew + SerialOld
  * 这个组合已经很少用（在某些版本中已经废弃）
  * https://stackoverflow.com/questions/34962257/why-remove-support-for-parnewserialold-anddefnewcms-in-the-future
* -XX:+UseConc<font color=red>(urrent)</font>MarkSweepGC = ParNew + CMS + Serial Old
* -XX:+UseParallelGC = Parallel Scavenge + Parallel Old (1.8默认) 【PS + SerialOld】
* -XX:+UseParallelOldGC = Parallel Scavenge + Parallel Old
* -XX:+UseG1GC = G1
* Linux中没找到默认GC的查看方法，而windows中会打印UseParallelGC 
  * java +XX:+PrintCommandLineFlags -version
  * 通过GC的日志来分辨

* Linux下1.8版本默认的垃圾回收器到底是什么？

  * 1.8.0_181 默认（看不出来）Copy MarkCompact
  * 1.8.0_222 默认 PS + PO

  

  java -XX:+PrintFlagsWithComments //只有debug版本能用



* JVM的命令行参数参考：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html

* JVM参数分类

  > 标准： - 开头，所有的HotSpot都支持
  >
  > 非标准：-X 开头，特定版本HotSpot支持特定命令
  >
  > 不稳定：-XX 开头，下个版本可能取消

  -XX:+PrintCommandLineFlags 

  -XX:+PrintFlagsFinal 最终参数值

  -XX:+PrintFlagsInitial 默认参数值


### PS GC日志详解

每种垃圾回收器的日志格式是不同的！

PS日志格式

![GC日志详解](./GC日志详解.png)

heap dump部分：

```powershell
eden space 5632K, 94% used [0x00000000ff980000,0x00000000ffeb3e28,0x00000000fff00000)
                            后面的内存地址指的是，起始地址，使用空间结束地址，整体空间结束地址
```

![GCHeapDump](GCHeapDump.png)

total = eden + 1个survivor


### 调优前的基础概念：

1. 吞吐量：用户代码时间 /（用户代码执行时间 + 垃圾回收时间）
2. 响应时间：STW越短，响应时间越好

所谓调优，首先确定，追求啥？吞吐量优先，还是响应时间优先？还是在满足一定的响应时间的情况下，要求达到多大的吞吐量...

问题：

科学计算，吞吐量。数据挖掘，thrput。吞吐量优先的一般：（PS + PO）

响应时间：网站 GUI API （1.8 G1）

### 什么是调优？

1. 根据需求进行JVM规划和预调优
2. 优化运行JVM运行环境（慢，卡顿）
3. 解决JVM运行过程中出现的各种问题(OOM)

### 调优，从规划开始

* 调优，从业务场景开始，没有业务场景的调优都是耍流氓
  
* 无监控（压力测试，能看到结果），不调优

* 步骤：
  1. 熟悉业务场景（没有最好的垃圾回收器，只有最合适的垃圾回收器）
     1. 响应时间、停顿时间 [CMS G1 ZGC] （需要给用户作响应）
     2. 吞吐量 = 用户时间 /( 用户时间 + GC时间) [PS]
  2. 选择回收器组合
  3. 计算内存需求（经验值 1.5G 16G）
  4. 选定CPU（越高越好）
  5. 设定年代大小、升级年龄
  6. 设定日志参数
     1. -Xloggc:/opt/xxx/logs/xxx-xxx-gc-%t.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=20M -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCCause
     2. 或者每天产生一个日志文件
  7. 观察日志情况
  

思维导图 查看地址:

https://www.processon.com/embed/5e902d83e401fd32b82a99c2




### 参考资料

1. [https://blogs.oracle.com/
    ](https://blogs.oracle.com/jonthecollector/our-collectors)[jonthecollector](https://blogs.oracle.com/jonthecollector/our-collectors)[/our-collectors](https://blogs.oracle.com/jonthecollector/our-collectors)
2. https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html
3. http://java.sun.com/javase/technologies/hotspot/vmoptions.jsp
5. 《深入理解JVM虚拟机》


