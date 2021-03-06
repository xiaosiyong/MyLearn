# Java一些基础

## 1、Java平台的理解？

Java 本身是一种面向对象的语言，最显著的特性有两个方面，一是所谓的“书写一次，到处运行”（Write once, run anywhere），能够非常容易地获得跨平台能力；另外就是垃圾收集（GC, Garbage Collection），Java 通过垃圾收集器（Garbage Collector）回收分配内存，大部分情况下，程序员不需要自己操心内存的分配和回收。

我们日常会接触到 JRE（Java Runtime Environment）或者 JDK（Java Development Kit）。 JRE，也就是 Java 运行环境，包含了 JVM 和 Java 类库，以及一些模块等。而 JDK 可以看作是 JRE 的一个超集，提供了更多工具，比如编译器、各种诊断工具等。

对于“Java 是解释执行”这句话，这个说法不太准确。我们开发的 Java 的源代码，首先通过 Javac 编译成为字节码（bytecode），然后，在运行时，通过 Java 虚拟机（JVM）内嵌的解释器将字节码转换成为最终的机器码。但是常见的 JVM，比如我们大多数情况使用的 Oracle JDK 提供的 Hotspot JVM，都提供了 JIT（Just-In-Time）编译器，也就是通常所说的动态编译器，JIT 能够在运行时将热点代码编译成机器码，这种情况下部分热点代码就属于编译执行，而不是解释执行了。

我们通常把 Java 分为编译期和运行时。这里说的 Java 的编译和 C/C++ 是有着不同的意义的，Javac 的编译，编译 Java 源码生成“.class”文件里面实际是字节码，而不是可以直接执行的机器码。Java 通过字节码和 Java 虚拟机（JVM）这种跨平台的抽象，屏蔽了操作系统和硬件的细节，这也是实现“一次编译，到处执行”的基础。

## 2、Exception和Error区别

Exception 和 Error 都是继承了 Throwable 类，在 Java 中只有 Throwable 类型的实例才可以被抛出（throw）或者捕获（catch），它是异常处理机制的基本组成类型。

Exception 和 Error 体现了 Java 平台设计者对不同异常情况的分类。Exception 是程序正常运行中，可以预料的意外情况，可能并且应该被捕获，进行相应处理。

Error 是指在正常情况下，不大可能出现的情况，绝大部分的 Error 都会导致程序（比如 JVM 自身）处于非正常的、不可恢复状态。既然是非正常情况，所以不便于也不需要捕获，常见的比如 OutOfMemoryError 之类，都是 Error 的子类。

Exception 又分为可检查（checked）异常和不检查（unchecked）异常，可检查异常在源代码里必须显式地进行捕获处理，这是编译期检查的一部分。前面我介绍的不可查的 Error，是 Throwable 不是 Exception。

不检查异常就是所谓的运行时异常，类似 NullPointerException、ArrayIndexOutOfBoundsException 之类，通常是可以编码避免的逻辑错误，具体根据需要来判断是否需要捕获，并不会在编译期强制要求。

## 3、谈谈final、finally、 finalize有什么不同

final 可以用来修饰类、方法、变量，分别有不同的意义，final 修饰的 class 代表不可以继承扩展，final 的变量是不可以修改的，而 final 的方法也是不可以重写的（override）。

finally 则是 Java 保证重点代码一定要被执行的一种机制。我们可以使用 try-finally 或者 try-catch-finally 来进行类似关闭 JDBC 连接、保证 unlock 锁等动作。

finalize 是基础类 java.lang.Object 的一个方法，它的设计目的是保证对象在被垃圾收集前完成特定资源的回收。finalize 机制现在已经不推荐使用，并且在 JDK 9 开始被标记为 deprecated。



























1、数组的下标为什么从0开始
从数组的内存模型来看，”下标“更确切的含义应该是”offset“，偏移，如果用a来表示数组的首地址，a[0]就是偏移为0的位置，也就是首地址，a[k]表示偏移k个
type_size的位置，所以计算a[k]公式：
a[i]_address = base_address + i * data_type_size
如果从1开始计算，则就变成了：
a[k]_address = base_address + (k-1)*type_size
每次随机访问元素多了一次减法运算，对于这种基础数据结构来说，优化尽可能做到极致，对于CPU来说，多了一次减法指令。

2、数组和链表有什么区别
最简单的区别：数组支持随机访问，根据下标随机访问的时间复杂度为O(1)，链表适合插入、删除。

3、针对数组类型，很多语言提供了容器类，比如Java中的ArrayList，C++中的vector，那么，什么时候用数组，什么时候用容器呢？
拿Java举例：
1）ArrayList 最大优势是可以将很多对数组操作的细节封装起来，比如插入、删除时搬移数据等
2）支持动态扩容
数组优势：
1）Java ArrayList无法存储基本类型，比如int、long，需要封装成对应的Integer、Long类，而Autoboxing、Unboxing会有性能消耗；
2）如果数组大小事先已知，并且对数据的操作简单，用不到ArrayList的大部分方法时，直接用数组
3）当表示多维数组时，数组更直观。Object[][],如果用容器：ArrayList<ArrayList> array.

4、递归
N个台阶，可以一次走两步，一次走一步，问有多少总走法
常规解法：直接递归，f(n)=f(n-1)+f(n-2)
弊端就是会重复计算





