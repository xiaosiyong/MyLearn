### Java基础

#### 垃圾回收

对象在物理内存空间被划分为两部分，新生代（young generate）和老年代（old generate）

**新生代**：大部分的新创建对象分配在新生代。因为大部分对象很快就会变得不可达，所以它们被分配在新生代，然后消失不再。当对象从新生代移除时，我们称之为"**minor GC**"。

**老年代**：存活在新生代中但未变为不可达的对象会被复制到老年代。一般来说老年代的内存空间比新生代大，所以在老年代GC发生的频率较新生代低一些。当对象从老年代被移除时，我们称之为"**major GC**"(或者**full GC**)。

![gc图](../images/java-gc.png)

# Java多线程
线程是JVM执行任务的最小单元

### 1、JVM中线程的状态
    * New
    * Runable（Ready<===>Running）
    * Blocked
    * Terminated
    * Waiting
    * Time_Waiting
### 2、状态转换

![state](../images/multi-thread.png)

当创建一个线程的时候，线程处在New状态，运行Thread的Start方法后，线程进入Runnable可运行状态。
这个时候，所有可运行状态的线程并不能马上运行，而是需要先进入就绪状态等待线程调度，如图中间的Ready状态。在获取到Cpu后才能进入运行状态，如图中的Running。运行状态可以随着不同条件转换成除New以外的其他状态。

先看左边，在运行态中的线程进入Synchronized同步块或者同步方法时，如果获取锁失败，则会进入到Blocked状态。当获取到锁后，会从Blocked状态恢复到就绪状态。

再看右边，运行中的线程还会进入等待状态，这两个等待一个是有超时时间的等待，例如调用Object.wait、Thread.join等。另外一个时无超时的等待，例如调用Thread.join或者Locksupport.park。

这两种等待都可以通过Notify或Unpark结束等待状态恢复到就绪状态。

最后是线程运行完成结束时，如图下方，线程状态变成Terminated

## 3、线程池
Executors工具类中提供了5种线程池的创建方法

    * 固定大小线程池，特点是线程数固定，使用无界队列，适用于任务数量不均匀的场景、对内存压力不敏感，但系统负载比较敏感的场景；
    
    * Cached线程池，特点是不限制线程数，适用于要求低延迟的短期任务场景；
    
    * 单线程线程池，也就是一个线程的固定线程池，适用于需要异步执行但需要保证任务顺序的场景；
    
    * Scheduled线程池，适用于定期执行任务场景，支持按固定频率定期执行和按固定延时定期执行两种方式；
    
    * 工作窃取线程池，使用的ForkJoinPool，是固定并行度的多任务队列，适合任务执行时长不均匀的场景。
[原文链接](https://mp.weixin.qq.com/s?__biz=MjM5MTE1NTQ4Mg==&mid=2649731439&idx=1&sn=64046fcaad7be914dcf922ec0a34cad7&chksm=bea2c51a89d54c0c6d258d4eae50b3a477fcf8227bc360591403dc9282ffce3f9c5bbc46b581&token=1490087409&lang=zh_CN#rd)

#### ThreadPoolExcutor