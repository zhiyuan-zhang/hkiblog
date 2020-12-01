---
title: 从JVM到调优
date: 2020/6/11
categories:
  - java
  - jvm
comments: true
abbrlink: 49197
img: /static/jvm.jpeg

---


# jvm

## 1. 虚拟机基础概念

### Class File Format

## jdk  -> jre -> jvm



### 

## 2. class文件结构



## 3. 内存加载过程 JMM

<font color=#F50A0A>3.1 Loading</font>

- 当class 加载到内存中 实际上生成了两块内容 
一个是将**二进制内容放到内存中** 与此同时 
<font color=red>**生成了一个.class对象 这个对象指向这个内容**</font>
后续所有的代码都会通过这个.class对象访问二进制内容
**类只会load一次**
- 所以用**EMUN**来实现单例就是这个原理 存放在 在jvm的matespacess中

- 类加载器 的机制及顺序

	- 1.bootstrap
- 最顶级的加载器 显示null
	
- 2.extension
	
	- 加载插件的加载器 jre/lib/ext/*.jar 下面的扩展jar包
	
- 3.app
	
	- 加载classPath指定内容
	
- 4.CustomClassLoader
	
	- 自定义ClassLoad
	
		- 可以自制类加载器
	
			- 代码混淆
				- 加密
	
	- 自定义加载器步骤
	
		- 1. extends ClassLoader
			- 2. overwrite findClass() -> defineClass(byte[] -> Class clazz)
			- 3. 加密
	
- ClassLoad 源码

	- findInCache -> parent.loadClass -> findClass()

- jvm是按需动态加载,采用双亲委派机制

	- 为什么要采用双亲委派机制

		- 为了防止使用类似String.class 不经过询问直接加载,导致会执行自定义逻辑, 
使用双亲委派后我们会直接返回已经加载过的内置.class

	- 自底向上检查该类是否已经加载 Parent方向
	- 自顶下下进行查找和加载child方向

- 关于jvm编译模式

	- 解释器- 解释语言
	- JIT just in Time compiler 代码编译
	- java采用的是混合编译模式  多次被调用的方法和多次被调用的循环 进行编译 
	- -Xmixed 默认混合模式
	- -Xint  使用解释模式 ,启动很快,执行稍慢
	- -Xcomp 使用纯编译模式,执行很快,启动慢

### <font color=#F50A0A>3.2 Linking</font>

- Verification

	- 来验证是否符合JVM语法规范  类似开头cafe babe 

- Preparation

	- 给静态成员变量赋默认值,  <font color=red>注意是默认值不是初始化值</font>

- Resolution

	- 将类、方法、属性等符号引用解析为直接引用
	- 常量池中的各种符号引用解析为指针、偏移量等内存地址的直接引用

### <font color=#F50A0A>3.3 Initializing ɪˈnɪʃəlaɪzɪŋ</font>

- 调用类初始化代码 <clinit>，给静态成员变量赋初始值





## 4. 运行时内存结构

### 每个线程都有单独的区域

-  <font color=#6AACDE>4.1 Program Counter  程序计数器</font>

  - 存放栈帧的下一步运行 
  - 虚拟机里面类似这样的循环  
  while( not end ) {
  ​	取PC中的位置，找到对应位置的指令；
  ​	执行该指令；
  ​	PC ++;
  }

- <font color=#6AACDE>4.2  JVM Stack</font>

	- 每个方法对应一个栈帧, 栈帧可以叠加 代表方法调用方法

		- 每个方法都有自己的栈帧,并且都拥有这四个属性

			- 1. Local Variable Table

				- 局部变量表  istore_3
				- 包括入参和方法内定义的变量
				- 非static 里面默认包含一个this

			- 2. Operand-Stack

				- 操作数栈 一个用来执行栈操作的 压栈和出栈 
				- 如果某个指令把一个值压入到操作数栈中，稍后另一个指令就可以弹出这个值来使用。

			- 3. Dynamic Linking

				- 动态链接   当一个方法里面有指向另一个类的符号引用时, 将符号引用替换为直接引用, 

			- 4. return address

				- a() -> b()，方法a调用了方法b, b方法的返回值放在什么地方
				- b方法的返回值放在A方法的栈顶 便于赋值
				- return后返回的地方 

		- A->B->C   栈帧也是 A上面是B  B上面是C

	- 指令集目前分为两种

		- 1 基于栈的指令集  JVM就是这样的

			- iadd
			- istore_2
			- load
			- pop
			- mul
			- sub
			- return
			- invoke

				- InvokeStatic

					- 调用静态方法使用的

				- InvokeVirtual

					- 多数方法使用这个  自带多态

				- InvokeInterface

					- List list = new ArrayList<String>(); 使用

				- InvokeSpecial

					- 可以直接调用的 不需要多态 
					- private  

				- InvokeDynamic

					- lambda 使用或者反射使用 

		- 2. <font color=red>基于寄存器的指令集  类似汇编  AX BX</font>

- <font color=#6AACDE>4.3 Native Method Stack</font>

	- 本地方法栈为虚拟机使用到的Native方法服务

### 共享区域

- <font color=#BD0CF7>4.4 Heap  堆</font>
  - 详细内容看 下面的 堆内逻辑分区
- <font color=#BD0CF7>4.5 method Area</font>
  - 装的各种各样的 class结构
  - 1. Perm Space (<1.8)
     字符串常量位于PermSpace
     FGC不会清理
     大小启动的时候指定，不能变
  - 2. Meta Space (>=1.8)
     字符串常量位于堆
     会触发FGC清理
     不设定的话，最大就是物理内存
- <font color=#BD0CF7>4.6 Direct Memory</font>
  - JVM 使用未公开的Unsafe 可以直接访问内核空间的内存 (操作系统OS管理的内存) 
  - NIO包下ByteBuffer 提高效率, 实现zero copy
  - 在jvm中只保留一个引用,

  - 可以扩展至更大的内存空间。比如超过1TB甚至比主存还大的空间
    - 理论上能减少GC暂停时间（节约了大量的堆内内存）
    - 它的持久化存储可以支持快速重启，同时还能够在测试环境中重现生产数据
    - 堆外内存能够提升IO效率

    - 堆内内存由JVM管理，属于“用户态”；而堆外内存由OS管理，属于“内核态”。
    如果从堆内向磁盘写数据时，数据会被先复制到堆外内存，即内核缓冲区，然后再由OS写入磁盘，使用堆外内存避免了数据从用户内向内核态的拷贝。
- <font color=#BD0CF7>4.7 Run-Time Constant Pool</font>
  - 常量池的数据

---



## JMM (java memory model)

### 硬件层数据一致性问题

- JMM 有很多协议 一般使用intel的MESI 

	- MESI 一共有四种状态来表示当前缓存的状态

- 现代CPU的数据一致性是靠 缓存锁 + 总线锁来实现
- 读取缓存是以cache line为单位, 目前是64bytes位 一块为单位
- 四块为一个单位的合并写技术 WCbuffer
- 伪共享 就是  当内存发现这一块的数据中有一个发生改变就会重新读取整个块的内容 这样就会导致  即使你用不到的数据发生变动 也会导致重新加载内存的问题

	- 在多线程与高并发期间Disruptor 就采用了在游标前后加上 7个空的long类型 填充缓存块

### 乱排序问题

- cpu会将无关的指令并行执行 但有时候也会出现问题
- 处理乱序的规范

	- 硬件内存屏障 X86 cpu硬件级别的内存屏障 利用汇编指令来完成 

		- save fence 写操作 写之前必须完成这个指令前的所有操作
		- load fence 读操作 读操作之前必须在这个指令前的所有操作
		- mfence 读写操作  这条指令前的所有读写必须完成了才能操作后续的指令
		- 原子指令，如x86上的”lock …” 指令是一个Full Barrier，执行时会锁住内存子系统来确保执行顺序，甚至跨多个CPU。Software Locks通常使用了内存屏障或原子指令来实现变量可见性和保持程序顺序

	- JVM级别的内存屏障

		- LoadLoad屏障  (读操作)

			- 语句Load1; LoadLoad; Load2，
			- 在执行之前必须保证之前的读取数据访问完成

		- StoreStore屏障 (写操作)

			- 语句Store1; StoreStore; Store2，
			- 在执行之前保证1的写入操作 对其他处理器可见

		- LoadStore屏障

			- 语句Load1; LoadStore; Store2，
			- 在后续写入操作被刷出之前,必须保证之前的操作读取完毕

		- StoreLoad屏障

			- 语句Store1; StoreLoad; Load2，
			- 在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。

### volatile实现细节

- 字节码层面

	- Access flags  = volatile

- JVM层面

	- volatile内存区的读写 都加屏障
	- StoreStoreBarrier

volatile 写操作

StoreLoadBarrier
	\- LoadLoadBarrier

volatile 读操作

LoadStoreBarrier

- OS/硬件层面

  - windows lock 指令实现

  - 可以用工具 hsdis 查看 

    

### Synchronized 实现细节

- 字节码层面

	- ACC_SYNCHRONIZED
monitorenter monitorexit  monitorexit

- JVM层面的实现

	- 调用了 C C++ 指令

- OS/硬件层面的实现

	- X86  Lock  cmpxchg XXX 指令 



---



## 5. jvm 常用命令 

### -开头的 标准参数

### -X 开头的是非标准参数

### -XX 是不建议使用的参数

### 常见垃圾回收器的组合参数设定 1.8

- 一般不会使用的

	- -XX:+UseParNewGC = ParNew + SerialOld
	- -XX:+UseSerialGC = Serial New (DefNew) + Serial Old

- 常见的几种组合

	- -XX:+UseConc(urrent)MarkSweepGC = ParNew + CMS + Serial Old
	- -XX:+UseParallelGC = Parallel Scavenge + Parallel Old (1.8默认) 【PS + SerialOld】
	- -XX:+UseParallelOldGC = Parallel Scavenge + Parallel Old
	- -XX:+UseG1GC = G1

- Linux下1.8版本默认的垃圾回收器到底是什么？

  - 1.8.0_181 默认 Copy MarkCompact （暂时无法确定）
  - 1.8.0_222 默认 PS + PO

### GC 日志打印参数

- -XX:+PrintGC 输出GC日志
- -XX:+PrintGCDetails 输出GC的详细日志
- -XX:+PrintGCTimeStamps 输出GC的时间戳（以基准时间的形式）
- -XX:+PrintGCDateStamps 输出GC的时间戳（以日期的形式，如 2013-05-04T21:53:59.234+0800）
- -XX:+PrintHeapAtGC 在进行GC的前后打印出堆的信息
- -Xloggc:../logs/gc.log 日志文件的输出路径
-  -XX:+HeapDumpOnOutOfMemoryError   发生异常 自动存储
- 例如: java  -XX:+HeapDumpOnOutOfMemoryError   -XX:+PrintGCDateStamps -XX:+PrintGCDetails -Xloggc:./gclog -jar zhxt-1.01-SNAPSHOT.jar

---



## 6. <font color=red>GC 与调优</font>

### 什么是垃圾

- 没有任何引用指向的一个对象或者多个对象

### 怎么查找到它

- Reference Count

	- 无法解决循环引用 

- Root Searching

	- 哪些可以成为根节点

		-  static references in method area 本地方法区的静态引用
		- class
		- run-time constant pool  常量池
		- native method stack JNI指针
		- JVM Stack

### GC 垃圾回收算法

- Mark-Sweep 标记清除

	- 通过根节点找到所有在用的,然后清除掉其他没用的
	- 优点: 存活对象多的情况下效率较高
缺点: 容易产生碎片,需要扫描两次

- Copying

	- 将内存一分为二 
优点: 适合存活对象较少 , 只需要扫描一次, 没有碎片
缺点: 移动复制对象,需要调整指向引用

- Mark-Compact

	- 扫描两次 需要将所以在使用的块移动到一起
	- 优点: 不会有碎片产生, 不会让内存减半
缺点: 需要移动对象,效率偏低,不适合存货对象较多的区域

### JVM 内存分代模型

- 逻辑物理都分代

	- POPS, CMS , 

- 逻辑分代,物理不分代

	- G1

- 逻辑物理都不分代

	- Epsilon ZGC Shenandoah

### 堆内逻辑分区

- new / young 新生代  1

	- eden 伊甸区 8 
	- survivor 1
	- survivor 1

- old tenured老年代  3 
- Method area

### Garbage Collectors

- Serial
- Seral Old
- ParNew (Parallel New)
- <font color = #55ED55 >CMS </font>
- 老年代垃圾回收器 
	- 清理过程
	
	- 初始标记
	
		- 找到根节点并且标记出来
	
	- 并发标记
	
		- 找出根节点下的所有关联节点
	
	- 重新标记
	
		- 由于并发标记并会停止线程工作 可能导致一些漏标或者错标问题,所以这个时候需要STW来重新整理下
	
	- 并发清理
	
		- 将其他的清理掉 这个时候的垃圾就浮动垃圾 等待下一次回收
	
- 缺点
	
	- Memory Fragmentation
	
		- 内存碎片的问题 当有新的对象在老年代放不下的时候会产生一次较长时间的STW 然后会有Seral Old 来标记移动对象 这样会效率很慢
			- 解决1.  保证老年代有足够的空间 -XX:CMSSInitiatingOccupancyFraction 60%
	
	- Floating Garbage
	
		- 浮动垃圾过多
	
- 算法
	
	- 三色标记 + Incremental Update
	
		- 三色标记 是在并发标记的过程中 将标记的所有对象归结为三类 黑,白,灰, 
	黑色代表自己和子节点标记完成,
灰色代表自己完成子节点未标记完成,
白色代表自己和子节点都没标记
		- 漏标问题 
	应为并发过程中还有线程会引用 所以会产生一个情况 
灰色的子节点标记删除了,
黑色新增一个白色的子节点,
这样导致子节点会不扫描
-  Incremental Update
			- 增量更新,关注引用的增加,将黑色重新标记为灰色,下次就会重新扫描
	
- Parallel Scavenge

	- copy 算法

- Parallel Old

	- mark-compact 算法

- <font color = #55ED55 >G1  1.8成熟  1.9 默认</font>
- 算法
	
	- 三色标记 + SATB
	
		- 漏标问题
	
			- SATB(Snapshot at the beginning) 将所有的删除操作放在一个栈里面 保证下一次直接扫描这个栈里面的引用就可以了
	
	- 将所有内存 分为 Eden, Survivor, Old , Humongous(分配大对象)
		- 传统YGC找一个对象是否存活非常麻烦 需要去遍历所有的堆空间, 这样非常消耗时间
		- G1 是把所有关联的内存当成一个card,存储在card table里, 然后每个card里面有一个标识 
		- RSet = RemenberedSet 
	记录了其他Region中的对象到本Region的引用
垃圾回收器只需要扫描这个RSet就能知道谁引用了
	-  新老年代比例 G1会根据上次YGC的时间来控制新老年代的比例 一般在5%-60% 之间
		- 什么时候回FGC 
	当内存分配过快的时候 回产生FGC 
- 扩内存
			- 提高CPU性能
			- 降低MixedGC的出发阈值,让MixedGC提前发生 默认是45%
		
	- XX:InitiatingHeapOccupacyPercent
	
- Card table  bitmap
	
- ZGC

	- 算法

		- ColoredPointers + 写屏障

- Epsilon

	- 什么都不做的垃圾回收器

- Shenandoah

	- 收费
	- 算法

		- ColoredPointers + 读屏障

### Tunning

- 基本概念

	- 吞吐量 优先

		- PS + PO 算法支撑
		- F=N*R/T
F：吞吐量；N：并发虚拟用户数；R：每个虚拟用户发出的请求数量；T：性能测试所用的时间

	- 响应时间 优先

		- G1

	- 淘宝一年来最高并发 54W
	- 12306 百万并发

- 什么是调优

	- 1. 根据需求进行JVM规划和预调优
	- 2. 优化运行JVM环境
	- 3. 解决各种JVM在运行过程中的问题 OOM

- 调优开始步骤

	- 需要有具体业务场景
	- 需要有压力测试报告指明调优方向,
	- 了解业务场景

		- 1. 响应时间、停顿时间 [CMS G1 ZGC] （需要给用户作响应）
		- 2. 吞吐量 = 用户时间 /( 用户时间 + GC时间) [PS]

	- 选择回收器组合
	- 计算内存需求( 经验值 1.5G 16G)
	- 选定CPU (越高越好)
	- 设定年代大小, 升级年龄
	- 设定日志参数

		- 1. -Xloggc:/opt/xxx/logs/xxx-xxx-gc-%t.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=20M -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCCause
		- 2. 或者每天产生一个日志文件

	- 观察日志情况

- 调优步骤

	- 查询当前机器所有的进程 
top  11056

或者 jps 查询当前所有java进程

查询指定进程中的所有线程执行状态
top -Hp 11056

查询所有的线程执行状况
jstack  11056

启动信息查询
jinfo 88768

打印堆栈信息 
jstat -gc 88768 500

找出哪个对象占用内存最多
jmap -histo 88768 | head -20

导出
jmap -dump:format=b,file=xxx pid：
线上系统，内存特别大，jmap执行期间会对进程产生很大影响，甚至卡顿（电商不适合）


- 调优工具Arthas

	- jvm == jinfo
	- thread

		- thread 54 
- dashboard == top
	- heapdump
	
	- heapdump /tmp/dump.hprof
		- 分析文件 jhat -J -mx=512M dump.hprof
	- jad
- redefine 热替换 (经测试 真实场景无法使用,只能使用无依赖的demo)

## 面试题

### 1. 对象的创建过程

- 1. 加载到内存
- 2. 申请对象内存 
- 3. 成员变量赋默认值 
- 4. 调用构造方法 <init>

	- 1. 成员变量 顺序 赋初始值
	- 2. 执行构造方法语句

### 2,内存布局

- 1. 普通对象

	- 1. markword 对象头
	- 2. ClassPointer指针 -XX:+UseConpressedClassPointers 为4个字节  不开启为8个字节
	- 3. 实例数据 -XX: +UserCompressedOops 为4个字节 不开启为8个字节
	- 4. padding 对其 8 的倍数

- 2. 数组对象

	- 1. 对象头：markword 8
	- 2. ClassPointer指针同上
	- 3. 数组长度：4字节
	- 4. 数组数据
	- 5. 对齐 8的倍数

### 3. 一个空对象在内存中占用多少个字节

- 普通对象

	- 对象头markword 8个字节
	- 指针 本来是8个字节 但是64位系统为压缩到4个字节
	- 本来是8+4 = 12 后续padding补齐 <font color = #55ED55 >一共16个字节</font>

- 数组对象

	- 比普通对象多个 数组长度 4个字节

### 4. 对象头具体包含什么

- 4位的 分代年龄
- 1位的偏向锁
- 2位的锁标志
- 31位的 hash code

### 5. 对象怎么定位

- 句柄池

	- 通过一个间接指针来指到两个指针上

		- 其中一个指定 T.clss
		- 另外一个指定具体内容

- 直接指针 (HotSpot使用)

	- A指向内容 内容包含B.class

### 6. 对象的分配过程

- 栈上分配

	- 线程私有小对象
	- *逃逸分析

- 查看大不大 如果大的直接old
- TLAB

	- Thread Loocal Allocation Buffer
	- 多线程的时候每个人分配占用1%的eden区域来分配对象,提高效率

- eden

### 对象何时进入老年代

- 除了CMS是6次之外 其他的都是15

### 分配担保

- YGC期间 当Sruvivor区内存空间不够了 空间担保直接进入老年代

### 动态年龄

- Survivor 的 区域大于50% , 找出年龄最大的放在老年代

