---
title: 多线程与高并发(基础知识)
date: 2019/12/28
categories:
  - other
  - java
  - linux
  - db
  - 工具
typora-root-url: ../
typora-copy-images-to: ../static
abbrlink: 7

---



# JUC

## 基本知识

### 实现方式

- 继承thread

	- .strat()

- 实现runable接口

	- new Thread(new MyRunable()).start()

- 匿名函数实现

	- new Thread(()->{
            System.out.println("Hello Lambda!");
        }).start();

- 启动线程的三种方式 1：Thread 2: Runnable 3:Executors.newCachedThrad

### yield

- 让出资源进入等待队列

### join

- 调用其他线程

### 线程状态迁移图

![线程状态迁移图](/static/线程状态迁移图.png)

### wait & notify

- notify 不释放锁 ,需要手动将当前线程wait, 让出锁  notify才能生效
- notifyAll 是让所有线程都醒过来去争抢锁  

### wait 和 await

-  await//此时当前线程释放lock锁，进入[等待状态]，等待其他线程执行 signal()时才有可能执行
- signal() //此时当前线程释放obj锁，随机唤醒一个处于等待状态的线程，继续执行wait后面的程序。
-  * 使用wait要用notify/notifyAll唤醒
-  * 使用await要用signal/signalAll唤醒

### synchronized 悲观锁

- 在对象头中有一个64位的对象头 markword其中有一个2位的标识符 00 - 11 共四个状态来标志这个锁对象
- 还有一位来标识是否是偏向锁
- this 和 锁住某个对象意义是一样的
- 锁方法和锁住整块代码意义也是一样的
- synchronized static 锁住的是 XX.class
- sychronized 是可重入锁 , 同一个锁可以相互调用,也就是说如果锁的是同一个对象那么可以相互调用
- sychronized 如果抛出异常会释放锁,这样的话会导致其他线程拿到锁之后执行业务逻辑,所以需要做try处理
- synchronized 锁的对象不能是String常量 ,Integer ,Long 
- synchronized(this) 的锁升级过程 

  1. 无锁状态
  当一个线程访问同步块并获取锁时，会在对象头和栈帧的锁记录里存储偏向的线程ID，
  以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁，
  
  只需测试Mark Word里线程ID是否为当前线程。
  因为没人和它竞争所以就没必要加锁
  
  如果后来有线程进来对比发现线程ID和自己不一样 则发生竞争
  
  
  
  2. 偏向锁
  如果判断发现ThreadID和自己不一样，则下一步需要判断偏向锁的标识。如果标识被设置为0（表示当前是无锁状态），则使用CAS(原子操作)竞争锁；
  
  如果标识设置成1（表示当前是偏向锁状态），则尝试使用CAS将对象头的偏向锁指向当前线程，触发偏向锁的撤销。
  
  偏向锁只有在竞争出现才会释放锁。当其他线程尝试竞争偏向锁时，程序到达全局安全点后（没有正在执行的代码），它会查看Java对象头中记录的线程是否存活，如果没有存活，那么锁对象被重置为无锁状态，
  
  其它线程可以竞争将其设置为偏向锁；
  
  如果存活，那么立刻查找该线程的栈帧信息，如果还是需要继续持有这个锁对象，那么暂停当前线程，撤销偏向锁，升级为轻量级锁，
  
  如果线程1不再使用该锁对象，那么将锁对象状态设为无锁状态，重新偏向新的线程。
  
  3.轻量级锁
  
  线程在执行同步块之前，
  
  JVM会先在当前线程的栈帧中创建用于存储锁记录的空间，并将对象头的MarkWord复制到锁记录中，即Displaced Mark Word。
  
  然后线程会尝试使用CAS将对象头中的Mark Word替换为指向锁记录的指针。如果成功，当前线程获得锁。如果失败，表示其他线程在竞争锁，当前线程使用自旋来获取锁。
  
  当自旋次数达到一定次数时，锁就会升级为重量级锁。
  
  轻量级锁解锁时，会使用CAS操作将Displaced Mark Word替换回到对象头，如果成功，表示没有竞争发生。
  
  如果失败，表示当前锁存在竞争，
  
  锁已经被升级为重量级锁，则会释放锁并唤醒等待的线程。
  
  
  4. 当自旋超过10次之后 我们会进入os的等待队列 退出CPU的资源 然后向系统资源申请重量级锁
  
  *注意：为了避免无用的自旋，轻量级锁一旦膨胀为重量级锁就不会再降级为轻量级锁了；偏向锁升级为轻量级锁也不能再降级为偏向锁。一句话就是锁可以升级不可以降级，但是偏向锁状态可以被重置为无锁状态。

### volatile

- 1.线程间可见

	- 缓存一致性协议

- 2.禁止jvm重排序

	- 线程在分配内存的时候 分三步 1.申请内存,2赋值,3引用指向, 当执行步骤为132的时候  会出现还没有赋值但是对象不为空的问题
	- JMM模式

		- jmm模型中有8个指令完成数据的读写, 其中 load store 指令相互组成的4个内存屏障实现指令重排序

- 注意: 和synchronized 不同的是  volatile可以保证可见性但并不能保证原子性, synchronized可以都保证.

### ThreadLocal

- 线程间各占一份 互不影响 
- static ThreadLocal<Person> tl = new ThreadLocal<>();
- 使用完成后必须remove调 不然会产生内存泄露

## JUC同步锁

### CAS (Compare And Set ) 无锁优化| 自旋锁 | 乐观锁

- atomic包中的都是CAS操作
- cas(V, Expected, NewValue) 有三个参数需要注意, v本次的对象, Expected 期望的值, NewValue 新的值 如果发现期望的值和现在的不一样那么就重新轮询一次获取新值
- CAS自旋原理

	- 在解释一下 
cas (操作对象， 期望值，更新值)

如果发现期望值不一样 则重新读取内存当中的值
if(当前值== 期望值){
 当前值 = 更新值
}else{
 重新获取值
}


- CPU源语支持 不存在指令重排序
- ABA问题 

	- 解决方法就是加一个版本号 不停地增加 AtomicStampedReference

### increment 高并发递增值的方法

-  sync
-  atomicXXXX
-  LongAdder 高并发情况下效率最高

	- 采用跳表实现 分段锁

### ReentrantLock

-  类似snychronized  
可以使用 .lock()  nulock() 来锁代码 
- trylock(time)
 尝试去拿锁， 如果规定时间内拿不到锁则继续执行，拿到了就加锁执行
- lockInterruptibly 
在一个线程等待锁的过程中可以被打断
 如果一个线程开启锁的方式是lockInterruptibly()那么这个线程可以调用.interrupt()被打断

-  new ReentrantLock(true)
 如果设置成公平锁，新的线程在竞争锁的时候回去检查队列里面是否有线程在等待 如果有则进入队列进行排队取锁


### CountDownLatch 门闩

-  使用环境是在于并发转换文件 当转换完成后需要继续执行业务 
 Thread[] threads = new Thread[100];
    CountDownLatch latch = new CountDownLatch(threads.length);

    当执行完成某个后需要调用latch.countDown();
    当全部执行完成后 latch.await()后继续之后的逻辑


### CyclicBarrier 线程栅栏

-  当任务达到一定数量后再执行并发操作
- 积攒一定数量的任务当数量达到阈值后开始执行任务

### Phaser  (feize)

- 升级版的CyclicBarrier , 可以控制执行阶段 就是一个业务分为十个阶段,每个阶段都需要并发执行,只有当上个阶段的都执行完才能执行下一个阶段,
- arriveAndAwaitAdvance()
- arriveAndDeregister()

### ReadWriteLock

- 共享锁 | 排他锁 | 读写锁
- 写锁 是排他锁 , 读锁是共享锁. 
- 类似读写分离吧, 读操作可以并发,写操作在后续执行 
static ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    static Lock readLock = readWriteLock.readLock();
    static Lock writeLock = readWriteLock.writeLock();

### Semaphoer

- 信号量 , 信号灯 0 - 1 
- 可以设置数量
 Semaphore s = new Semaphore(2);
- 可以设置公平锁 
 Semaphore s = new Semaphore(2, true);
- 使用场景 限流 (车道-收费站)
- semaphoer和lock的区别

### Exchanger

- 线程间通讯
-  s = exchanger.exchange(s);
- 当线程运行到这段代码后会阻塞等待交换

### Locksupport

- part 停车 或者 停止
- unpart 开始 或者 启动

## AQS (AbstractQueuedSynchronized)

### ReentrantLock -- sync -- AQS

### 维护了一个双向链表队列- 所有人去抢 status这个状态值 它用volatile 修饰 后续会用CAS操作更改值

### 锁住链表最后一个节点然后用CAS来操作

### 公平锁和非公平锁的区别是会去尝试通过CAS来操作如果不成功则进入队列 成功抢到锁则运行

## 强软弱虚 引用

### 强  new Object()

### 软 SoftReference

- 软引用在系统内存不够用的时候会回收
- 主要用来做缓存

### 弱waekReference

- 只要遭遇GC,就会回收
只要进行一次GC,就会回收
- TheadLocal 使用了弱引用中的entry
entry继承自弱引用 , 如果当前线程存在则不会回收,如果当前线程消失可能会存在内存泄露,但是使用弱引用泄露的概率就很小了

### 虚phantomReterence

- 当你的框架或者是项目需要使用对外内存可能会用到这个引用
-      一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，
     也无法通过虚引用来获取一个对象的实例。
     为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。
     虚引用和弱引用对关联对象的回收都不会产生影响，如果只有虚引用活着弱引用关联着对象，
     那么这个对象就会被回收。它们的不同之处在于弱引用的get方法，虚引用的get方法始终返回null,
     弱引用可以使用ReferenceQueue,虚引用必须配合ReferenceQueue使用。

     jdk中直接内存的回收就用到虚引用，由于jvm自动内存管理的范围是堆内存，
     而直接内存是在堆内存之外（其实是内存映射文件，自行去理解虚拟内存空间的相关概念），
     所以直接内存的分配和回收都是有Unsafe类去操作，java在申请一块直接内存之后，
     会在堆内存分配一个对象保存这个堆外内存的引用，
     这个对象被垃圾收集器管理，一旦这个对象被回收，
     相应的用户线程会收到通知并对直接内存进行清理工作。

     事实上，虚引用有一个很重要的用途就是用来做堆外内存的释放，
     DirectByteBuffer就是通过虚引用来实现堆外内存的释放的。

## 同步容器

### Collections 单个数据进出

- List

	- 完全没锁 线程不安全, 需要手动加锁来处理任务
	- CopyOnWrite  写-时-复制

		- 也就是说 在写的时候复制一份对象  如果一个业务写的业务较少 读取的业务较多的时候 可以考虑用这个， 其本质上来讲就是一个ReadWriteLock
		- CopyOnWriteList

			- /**
 * Appends the specified element to the end of this list.
 *
 * @param e element to be appended to this list
 * @return {@code true} (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        //重点是这一句话
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}

		- CopyOnWriteSet
	
			- /**
     * A version of addIfAbsent using the strong hint that given
     * recent snapshot does not contain e.
     */
    private boolean addIfAbsent(E e, Object[] snapshot) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] current = getArray();
            int len = current.length;
            if (snapshot != current) {
                // Optimize for lost race to another addXXX operation
                int common = Math.min(snapshot.length, len);
                for (int i = 0; i < common; i++)
                    if (current[i] != snapshot[i] && eq(e, current[i]))
                        return false;
                if (indexOf(e, current, common, len) >= 0)
                        return false;
            }
            Object[] newElements = Arrays.copyOf(current, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }

	- ArrayList

		- 未考虑多线程安全（未实现同步）

- Set
- vector

	- Vector 线程安全 大部分关键方法都加了锁 
但是可能出现锁与锁之间的共享问题 还是会出现线程不安全
就是两个方法虽然都加了锁但是 调用过程中会出现问题


- Queue 面对高并发准备

	- (*高并发,多线程) 内部使用的是CAS来操作数据
	- ConcurrentLinkedQueue

		- 	和Queue 的区别
	offer -> add  会有返回值
	peek -> get 
	poll -> get remove


	- ArrayBlockingQueue
	
		- 数组阻塞队列  
strs.offer("aaa", 1, TimeUnit.SECONDS);当内容满了之后就会等待阻塞 如果后面加上时间参数则会等待规定的时间后返回false 继续执行

	- LinkedBlockQueue
	
		- 链表实现的阻塞队列 
在ConcurrentLinkedQueue基础上添加了方法
	put 如果满了就等待
	take 如果空了就等待
	底层实现是利用了part 和unpart  类似之前面试题用到的 wait notify


	- DelayQueue
	
		- 	按照等待时间排序  也是阻塞队列的一种 BlockingQueue
	按照时间进行任务调度可以用这个
	本质上是PriorityQueue
	
	- SychronusQueue 同步
	
		- 本质上和exchange线程差不多 
	线程size为0  所以不能add 不能往里面装东西 
	会阻塞等待消费者进行消费 
	put
	take


	- TransferQueue
	
		- 需要得到一个线程完成的标记或者是结果 可以用这个

### Map 键值对K-V 

- HashMap

	- HashTable

		- 刚开始设计的时候只有这个,所有方法都加了锁的 线程是安全的 但是性能不是很高

	- HashMap

		- 后来意识到之后就把锁都给取消了 , 所有方法都没有加锁  线程不安全  但是性能较高

	- SynchronizedHashTable

		- 后来觉得锁的粒度放细一点,只在具体调用函数的代码块来锁 其实和HashTable本质上并没什么区别.

	- ConcurrentHashMap  无序集合 高并发使用 

		- 采用了跳表的形式虽然写的时候不如 HashTable高 但是读起来性能高很多

	- ConcurrentSkipListMap

		- 有序集合线程安全高并发使用  跳表实现原理 读取效率高

	- LinkedHashMap

		- 里面维护了一个 LinkedKeySet
		- 非线程安全 

- TreeMap
- WeakHashMap
- identityHashMap

## 面试题 

### 背过 

- 面试题：写一个固定容量同步容器，拥有put和get方法，以及getCount方法，
 * 能够支持2个生产者线程以及10个消费者线程的阻塞调用

	- 注意点:  1.wait 和await的区别  2. lock 和unlock  要全部包住代码 

- put get

### 并发和并行

- Concurrent VS Parallel

并发指的是任务同时到达某个接口

并行指的是多个CPU同时去处理某个事情


并行是并发的子集

## 线程池

### ExecutorServices ->  Executor   线程池 

- 里面拥有很多操作线程的方法 

	- awaitTermination
	- invokeAll
	- invokeAny
	- isShutdown
	- isTerminated
	- shutdown
	- shutdownNow
	- submit

- ExecutorServeices.submit 之后是异步执行的 不影响主线程

### 基本知识

- Callable(String)  一个有返回值的线程
- Future

	- Future<String> submit = executorService.submit(() -> {
            return "";
        });
    submit.get();
	- 线程池提交一个线程之后会返回这个线程的future 我们可以get它的返回值 提交是异步 get是阻塞的

-  FutureTask

	- 用来存放Callable的返回值 --public FutureTask(Callable<V> callable)
	- 如果想拿到的话 直接调用 FutureTask.get() 这个方法是阻塞的
	相当于直接拿到这个线程的返回值 不需要生成线程池

- CompletableFuture

	- 组合各个线程的返回结果然后统一返回一个结果 

### 两种线程池的自定义方式

- ThreadPoolExecutor

	- ThreadPoolExecutor -> AbstractExecutorService -> ExecutorService ->  Executor
	- ThreadPoolExecutor tpe = new ThreadPoolExecutor(2, 4,
            60, TimeUnit.SECONDS,
            new ArrayBlockingQueue<Runnable>(4),
            Executors.defaultThreadFactory(),
            new ThreadPoolExecutor.CallerRunsPolicy());

	- 1. corePoolSize 核心线程
	- 2. maxPoolSize 可扩展的最大线程
	- 3. keepAliveTime  如果一个线程空闲时间超过设定值则讲线程紫苑交给操作系统 核心线程永远活着 不过有个方法可以设置核心线程参不参与归还资源
	- 4. TimeUnit 时间单位
	- 5. queue  BolckingQueue线程队列
当核心线程满了之后 后续任务进入的等待队列
	- 6. threadFactory DefaultThreadFactory 线程池的工厂类
实现 ThreadFactory 接口  里面有一个newThread 自己去实现具体的线程
public Thread newThread(Runnable r) {
    Thread t = new Thread(group, r,
                          namePrefix + threadNumber.getAndIncrement(),
                          0);
    if (t.isDaemon())
    //设置线程守护
        t.setDaemon(false);
    if (t.getPriority() != Thread.NORM_PRIORITY)
    //设置优先级
        t.setPriority(Thread.NORM_PRIORITY);
    return t;
}

	- 7. rejectedExecutionHandler 拒绝策略 

线程池满了 并且 任务队列也满了 然后启动的非核心线程数也满了  会执行拒绝策略

		- JDK默认提供了四种拒绝策略 
		- Abort  抛异常
		- Discard 扔掉,不抛异常
		- DiscardOldest 扔掉排队时间最久的
		- CallerRuns 调用处理任务 哪个线程调用的哪个去处理 非异步处理
		- 我们也可以去自定义自己的拒绝策略
实现 implements RejectedExecutionHandler接口
完成rejectedExecution方法

			- 
自定义拒绝策略


static class MyHandler implements RejectedExecutionHandler {

    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        //log("r rejected")
        //save r kafka mysql redis
        //try 3 times
        if(executor.getQueue().size() < 10000) {
            //try put again();
        }
    }
}


- ForkJoinPool

	- ForkJoinPool -> AbstractExecutorService -> ExecutorService ->  Executor
	- 主要用来处理CPU密集型的任务 
	- 每个线程有自己的任务队列
	- 把大任务切分成一个一个的小任务来执行
然后再汇总结果
	- newWorkStealingPool
 new ForkJoinPool
(Runtime.getRuntime().availableProcessors(),
 ForkJoinPool.defaultForkJoinWorkerThreadFactory,
 null, true);

### JDK默认的线程池的实现

- Executors -线程池的工厂
- 1. SingleThreadExecutor

	- 单例的线程池 里面只有一个线程
new FinalizableDelegatedExecutorService
(new ThreadPoolExecutor(1, 1,
                        0L, TimeUnit.MILLISECONDS,
                        new LinkedBlockingQueue<Runnable>()));

- 2. newCachedThreadPool

	- 里面拥有核心线程池为0, 可以扩展最大的为int最大值
new ThreadPoolExecutor(0, Integer.MAX_VALUE,
						  60L, TimeUnit.SECONDS,
						  new SynchronousQueue<Runnable>());

- 3. newFixedThreadPool

	- 核心线程和扩展线程都是固定的值 可以通过计算 一般是CPU的核心线程数+1
new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());


- 4. newScheduledThreadPool

	- 定时任务线程池
可以用框架代替 
Quartz cron 

super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
      new DelayedWorkQueue());


- 以上四个线程池底层都是ThreadPoolExecutor来实现的

## Disruptor

### 维护一个环形数组队列 首尾相连

### 目前性能最高的MQ

