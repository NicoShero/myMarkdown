# 参考资料
* [周志明. 深入理解 Java 虚拟机 [M]. 机械工业出版社, 2011.]
* [Chapter 2. The Structure of the Java Virtual Machine](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5.4)
* [Jvm memory Getting Started with the G1 Garbage Collector](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/G1GettingStarted/index.html)
* [JNI Part1: Java Native Interface Introduction and “Hello World” application](http://electrofriends.com/articles/jni/jni-part1-java-native-interface/)
* [Memory Architecture Of JVM(Runtime Data Areas)](https://hackthejava.wordpress.com/2015/01/09/memory-architecture-by-jvmruntime-data-areas/)
* [JVM Run-Time Data Areas](https://hackthejava.wordpress.com/2015/01/09/memory-architecture-by-jvmruntime-data-areas/)
* [Android on x86: Java Native Interface and the Android Native Development Kit](http://www.drdobbs.com/architecture-and-design/android-on-x86-java-native-interface-and/240166271)
* [深入理解 JVM(2)——GC 算法与内存分配策略](https://crowhawk.github.io/2017/08/10/jvm_2/)
* [深入理解 JVM(3)——7 种垃圾收集器](https://crowhawk.github.io/2017/08/15/jvm_3/)
* [JVM Internals](http://blog.jamesdbloom.com/JVMInternals.html)
* [深入探讨 Java 类加载器](https://www.ibm.com/developerworks/cn/java/j-lo-classloader/index.html#code6)
* [Guide to WeakHashMap in Java](http://www.baeldung.com/java-weakhashmap)
* [Tomcat example source code file (ConcurrentCache.java)](https://alvinalexander.com/java/jwarehouse/apache-tomcat-6.0.16/java/org/apache/el/util/ConcurrentCache.java.shtml)

# 运行时数据区域  
![](assets/Java虚拟机-8c56cfe3.png)

## 程序计数器
记录正在执行的虚拟机字节指令的地址(本地方法则为空)

## Java 虚拟机栈
每个 Java 方法在执行的同时会创建一个栈帧用于存储局部变量表，操作数栈，常量池引用等信息。从方法调用直至执行完成的过程，对应着一个栈帧在 Java 虚拟机中入栈和出栈的过程。   

![](assets/Java虚拟机-f98d312b.png)

可以通过 -Xss 来制定每个线程的 Java 虚拟机栈内存的大小，在 1.4 中默认为 256K，在 1.5 中默认为 1M

>java -Xss2M HackTheJava

该区域可能的异常有：
* 线程栈深度超过最大值，Stack OverflowError 异常；
* 栈进行动态扩展时无足够内存，OutOfMemoryError 异常

## 本地方法栈
本地方法栈与 Java 虚拟机栈类似，它们之间的区别不过是本地方法栈为本地方法服务。    
本地方法一般是用其他语言( C、C++ 或汇编语言等) 编写的，并且被编译为基于本机硬件和操作系统的程序，对待这些方法需要特别处理。

![](assets/Java虚拟机-9155a209.png)

## 堆
所有对象都在这里分配内存，垃圾收集的主要区域 ("GC堆")。   
现代的垃圾收集器基本都是采用分代收集算法，其主要思想是针对不同类型的对象采取不同的垃圾回收算法。可以将堆分成两块：
* 新生代
* 老年代

堆不需要连续内存，并且可以动态增加其内存，增加失败会抛出 OutOfMemoryError 异常。   
可以通过 -Xms 和 -Xmx 两个参数指定一个程序的堆内存大小，第一个参数设置初始值，第二个参数设置最大值。    

> java -Xms1M -Xmx2M HackTheJava

## 方法区
用于存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。    
和堆一样，不需要连续内存，可以动态扩展，扩展失败后抛出 OutOfMemoryError 异常。    
对这块区域进行垃圾回收的主要目标是对常量池的回收和对类的卸载，但是一般比较难实现。   
HotSpot 虚拟机把它当成永久代来进行垃圾回收。但很难确定永久代的大小，因为它收很多因素影响，并且每次 Full GC 之后永久代大小都会改变，所以经常抛出 OutOfMemoryError。    
1.8 以后，移除永久代，把方法区移至堆和元空间中。元空间存储类的元信息，静态变量和常量池等放入对重。   

## 运行时常量池
方法区的一部分，Class 文件中的常量池在类加载后会进入这个区域，除了编译器生产的，还允许动态生成，例如 String 类的 intern().

## 直接内存
1.4 中新引入了 NIO 类，它可以使用 Native 函数库直接分配堆外内存，然后通过 Java 堆里的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，避免了堆内外内存来回拷贝。


# 垃圾收集
主要针对堆和方法区。程序器、虚拟机栈和本地方法栈这三个区域属于线程私有，生命周期短，随线程结束而消失，所以不需要回收。

## 判断是否需要回收
### 1. 引用计数算法
为对象添加一个引用计数器，对象增加一个引用就加一，引用失效就减一。计数器为 0 的对象可以被回收。   

在两个对象出现循环引用的情况下，此时引用计数器永远不为 0，导致无法对它们进行回收。正式因为循环引用的存在，因此 Java 不适用该方法。

    public class Test {

        public Object instance = null;
        public static void main(String[] args) {
            Test a = new Test();
            Test b = new Test();
            a.instance = b;
            b.instance = a;
            a = null;
            b = null;
            doSomething();
        }
    }

上述代码中 a,b 互相引用，此时及时将 a b 的引用去除，两个对象还是存在相互的引用，导致它们无法被回收。

### 2. 可达性分析算法
以 GC Roots 为起始点搜索，所有可达的对象都是存活的，不可达的对象可以被回收。   
GC Roots 一般包含：
* 虚拟机栈中局部变量表中引用的对象
* 本地方法栈中 JNI 中引用的对象
* 方法区中类静态属性引用的对象
* 方法区中的常量引用的对象

![](assets/Java虚拟机-38e835fb.png)

### 3. 方法区的回收
因为方法去主要存放永久代对象，而永久代对象回收率比新生代低很多，所以方法去上的回收性价比不高。   
主要目标是对常量池的回收和对类的卸载。   
为了避免内存溢出，在大量使用反射和动态代理的场景都需要虚拟机具备类卸载功能。    
类的卸载需要满足三大条件，就算都满足了也不一定会被卸载：
* 该类所有的实例都已被回收
* 加载该类的 ClassLoader 被回收
* 该类对应的 Class 对象没有在任何地方被引用，也就无法再任何地方通过反射访问该类方法。


### 4. finalize()
类似 C++ 的析构函数，用于关闭外部资源，但是 try-finally 等方式可以做的跟高，并且运行代价高，无法保证对象调用顺序，因此不建议使用。    
当对象可回收，如果需要执行 finallize() 方法，那么就有可能在该方法中让对象被重新引用，实现自救。不过自救只能进行一次，如果经过 finalize() 方法成功自救过一次，后续回收不会调用该方法。


## 引用类型
### 1. 强引用
被强引用关联的对象不会被回收。   
使用 new Object() 的方式创建强引用。   

### 2. 软引用
被软引用关联的对象只有在内存不够的情况下才会被回收。    
使用 SoftReference 类创建软引用。

    Object obj = new Object();
    SoftReference<Object> sf = new SoftReference<Object>(obj);
    obj = null;  // 使对象只被软引用关联

### 3. 弱引用
被弱引用关联的对象一定会被回收，也就是说他只能存活到下一次垃圾回收前。   
使用 WeakReference 类创建弱引用。    

    Object obj = new Object();
    WeakReference<Object> wf = new WeakReference<Object>(obj);
    obj = null;

### 4. 虚引用
被虚引用关联的对象不会对其存活时间造成影响，唯一的作用是对象被回收时会收到一个系统通知。    
使用 PhantomReference 创建虚引用。

    Object obj = new Object();
    PhantomReference<Object> pf = new PhantomReference<Object>(obj, null);
    obj = null;


## 垃圾收集算法
### 1. 标记 - 清除

![](assets/Java虚拟机-ed5e576d.png)

标记阶段，检查每个对象是否存活，如果存活，在对象头部打上标记。    
在清除阶段，会回收对象并清除标志位，并配置空闲链表记录空闲分块。    
分配时，寻找空间大于或等于新对象大小的分块。如果分块等于对象大小则直接返回，否则会根据对象大小划分成两个块，返回等于对象大小的块，剩余的返还空闲链表。   
不足：
* 标记和清除过程效率不高
* 会产生大量不连续的内存碎片，导致无法给大对象分配内存。

### 2. 标记 - 整理

![](assets/Java虚拟机-f1479bf4.png)

让所有存活对象都想一段移动，然后清理掉边界以外的内存。   
优点：
* 不会产生内存碎片
缺点：
大量移动对象，效率较低

### 3. 复制

![](assets/Java虚拟机-d17e9566.png)

将内存划分为大小相等的两块，当一面用完以后，将存活的对象复制到另一面，并清空使用过的内存。   
主要缺点是减少了可用内存，主要用于新生代，并且不再是大小相等的两块，而是一块 Eden 和两块 Survivor。每次使用的是 Eden 和一块 Survivor，回收时将对象复制到另一块 survivor，最后清空其他的区域。    
HotSpot 默认大小是 8:1:1 ,保证了内存利用率到 90% ，如果存活的对象超过 10% 需要担保保存到老年代。

### 4. 分代收集
根据对象存活周期将内存分为几块，不同分块选用不同收集方式。
* 新生代： 复制算法
* 老年代： 标记-清除 或 标记-整理


## 垃圾收集器

![](assets/Java虚拟机-16904c85.png)

以上是 HotSpot 的七个垃圾收集器，连线表示垃圾收集器可以配合使用    
* 单线程与多线程：表示垃圾收集器使用一个或多个线程
* 串行与并行：串行是指垃圾回收和用户程序交替进行，并行是指同时进行 (CMS 和 G1 可以并行)。

### 1. Serial

![](assets/Java虚拟机-50922a24.png)

Serial 翻译为串行，也就是说它以串行的方式执行。   
它是单线程的收集器，只会使用一个线程进行垃圾收集工作。   
简单高效，单 CPU 环境下，拥有最高的单线程收集效率。    
它是 Client 场景下的默认新生代收集器，因为该场景下内存一般不大，收集两百兆垃圾的定顿时间在百毫秒内。

### 2. ParNew

![](assets/Java虚拟机-db0ffa9a.png)

是 Serial 的多线程版本。    
是 Server 场景下默认的新生代收集器，出了性能原因，目前只有它可以和 CMS 配合使用。

### 3. Parallel Scavenge

多线程收集器，不同于其他收集器致力于缩减停顿时间，它的目的是实现吞吐量的可控，被称为 "吞吐量优先" 收集器 (吞吐量是指 CPU 执行用户程序的时间占比)。     
停顿时间越短就越适合用户交互的程序，提高用户体验。而吞吐量高则可以高效利用 CPU 时间，尽快完成任务，适合后台运算切交互较少的任务。   
缩短停顿时间是以牺牲吞吐量和新生代空间换取的： 新生代空间变小，垃圾回收频繁，吞吐量下降    
只需要把基本的内存数据设置好（如-Xmx设置最大堆），然后使用-XX：MaxGCPauseMillis参数（更关注最大停顿时间）或GCTimeRatio参数（更关注吞吐量）给虚拟机设立一个优化目标，那具体细节参数的调节工作就由虚拟机完成了。


### 4. 分代收集
现在的虚拟机一般根据对象的生存周期将内存划分为几块，每块都有各自的回收算法，一般分为新生代和老年代。    
* 新生代：复制算法(8:1:1)
* 老年代：标记清除 或 标记整理算法


## 垃圾收集器

![](assets/Java虚拟机-16904c85.png)

上图中连线的收集器可配合使用。   
* 单线程与多线程：指收集器使用的线程个数，单线程只使用一个
* 串行与并行：串行指的是垃圾收集器与用户程序交替执行，并行是指同时执行。(目前只有 CMS 和 G1 是并行的)

### 1. Serial

![](assets/Java虚拟机-50922a24.png)

Serail 翻译为串行，是以串行方式执行的。它是单线程的收集器，优点是简单高效，单 CPU 环境下，没有线程交互的开销，所有拥有最高的单线程收集效率。    
它是 Client 下默认的新生代收集器，该场景下内存一般不会很大，一两百兆垃圾的手机时间一般在一百毫秒内。

### 2. ParNew

![](assets/Java虚拟机-db0ffa9a.png)

Serial 的多线程版本，它是 Server 场景下默认的新生代收集器，出了性能原因外，主要原因是出了 Serial 以外只有它能搭配 CMS 使用。


### 3. Parallel Scavenge
与 ParNew 一样是多线程收集器，他的目标是达到一个而控制的吞吐量，因此被曾为吞吐量优先收集器。（吞吐量是指 CPU 时间中用户程序时间和垃圾收集时间占比），高吞吐量适合在后台运算尽快的出结果的场景。       
停顿时间缩短是靠减少回收空间，提高回收频率换来的，降低了吞吐量。    
可以通过配置开关参数打开 GC自适应调节策略，打开自适应策略后虚拟机会自行配置新生代大小，ES比例，今生老年代年龄等参数，来提供吧最大的吞吐量或最合适的停顿时间。

参数|描述
:--:|:--:
-Xms|初始堆大小
-Xmx|最大堆大小
-XX:NewSize=n|设置年轻代大小
-XX:NewRatio=n|设置年轻代和年老代的比值.如:为3,表示年轻代与年老代比值为1:3,年轻代占整个年轻代年老代和的1/4
-XX:SurvivorRatio=n|年轻代中Eden区与两个Survivor区的比值.注意Survivor区有两个.如:3,表示Eden:Survivor=3:2,一个Survivor区占整个年轻代的1/5
-XX:MaxPermSize=n|设置持久代大小

#### 收集器设置

参数|描述
:--:|:--:
-XX:+UseSerialGC|设置串行收集器
-XX:+UseParallelGC|设置并行收集器
-XX:+UseParalledlOldGC|设置并行年老代收集器
-XX:+UseConcMarkSweepGC|设置并发收集器

#### 垃圾回收统计信息

参数|描述
:--:|:--:
-XX:+PrintGC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Xloggc:filename

#### 并行收集器设置

参数|描述
:--:|:--:
-XX:MaxGCPauseMillis    |控制最大垃圾收集停顿时间
-XX:GCTimeRatio         |设置吞吐量大小
-XX:+UseAdaptivesSizePolicy |GC自适应的调节策略,打开这个参数就不用指定新生代的大小，Eden和Survior区的比例了，晋升老年代对象年龄等细节参数，只需要设定堆的 -Xmx 和并行收集器的最大吞吐量和最大收集时间就行了
-XX:ParallelGCThreads=n|设置并行收集器收集时使用的CPU数.并行收集//线程数.
-XX:MaxGCPauseMillis=n|设置并行收集最大暂停时间
-XX:GCTimeRatio=n|设置垃圾回收时间占程序运行时间的百分比.公式为1/(1+n)

#### 并发收集器设置

参数|描述
:--:|:--:
-XX:+CMSIncrementalMode|设置为增量模式.适用于单CPU情况.
-XX:ParallelGCThreads=n|设置并发收集器年轻代收集方式为并行收集时,使用的CPU数.并行收集线程数.


### 4. Serial Old

![](assets/Java虚拟机-c1a7d6de.png)

是 Serail 的老年代版本，也是 Client 场景下的虚拟机使用。如果在 Server 场景下，他有两大用途：
* 1.5 及之前的版本中与 Parallel Scavenge 收集器搭配使用
* 作为 CMS 收集器的后备方案，在并发收集发生 Concurrent Mode Failure 时使用


### 5. Parallel Old

![](assets/Java虚拟机-dbb34951.png)

是 parallel Scavenge 收集器的老年代版本。    
注重吞吐量和 CPU 资源敏感的场合，都可以优先考虑 Parallel Scavenge + Parallel Old 的组合


### 6. CMS

![](assets/Java虚拟机-8c78d43c.png)

分为以下四个阶段:
* 初始标记： 标记 GC Roots 直接关联的对象，速度快，需要停顿
* 并发标记： 进行 GC Roots Tracing，耗时长，不需要停顿
* 重新标记： 修正并发标记时，用户运行期间发生变化的那部分对象，需要停顿
* 并发清除： 不需要停顿

在整个过程中耗时最长的并发标记和并发清除过程中，收集器线程可以与用户线程一起运行

缺点：
* 吞吐量低
* 无法处理浮动垃圾，可能出现 Concurrent Mode Failure (CMF)。浮动垃圾是指并发清除阶段用户继续运行产生的垃圾，这部分只能等到下一次 GC 。由于浮动垃圾存在，需要预留出一部分内存，意味着 CMS 不能和其他收集器一样等到老年代满了在进行回收，雨果预留内存无法容纳浮动垃圾就会产生 CMF。这时候虚拟机会临时启用 Serial Old 替代 CMS。
* 标记-清除导致的空间碎片导致老年代对象找不到足够大的连续碎片，提前导致 Full GC 的发生

### 7. G1（Garbage-First）
它是面向服务端应用的垃圾收集器，在多 CPU 和大内存的场景下有很好的性能。 HosSpot 希望他能替代 CMS。    
其他收集器都作用于新生代或者老年代，而 G1 可以同时回收新生代和老年代。

![](assets/Java虚拟机-552e1716.png)

G1 把堆划分成若干大小相等的区域 (Region)，新生代和老年代不再物理隔离。

![](assets/Java虚拟机-f5ce3e3a.png)

通过引入区域的概念，使得一整块的内存空间划分成若干独立的小空间，每个空间可以独立的进行垃圾回收，这种回收的灵活性使可预测的停顿时间模型成为可能。通过记录每个 Region 垃圾回收时间以及回收所获得的空间 (历史经验值)，并维护一个优先列表，每次根据允许的收集时间优先回收价值最大的 Region 。

每个 Region 都有一个 Remembered Set，用来记录该 Region 对象的引用对象所在的 Region 地址，通过 Remembered Set 使得进行可达性分析使避免了全堆扫描。

![](assets/Java虚拟机-6a9a13bc.png)

如果不计算维护 Remembered Set 的操作，G1 运行大致可以分为以下步骤：
* 初始标记
* 并发标记
* 最终标记：修正并发标记过程中用户程序继续运作导致标记发生变化的一部分标记记录，虚拟机将这段时间对象变化记录在线程的 Remembered Set Logs 中，最终标记需要把这些数据合并到 Remembered Set 中，需要停顿线程，也可并发执行。
* 筛选回收：首先堆各个 Region 中回收价值和成本进行排序，根据用户期望 GC 停顿时间指定回收计划

特点如下：
* 空间整合：整体来看是基于标记-整理算法，从局部 (两个 Region) 来看是基于复制算法实现的，这意味着运行期间不会产生内存碎片
* 可预测的停顿：能让使用者明确在一个长度在 M 毫秒内，消耗在 GC 上的时间不超过 N 毫秒。

# 内存分配和回收策略
## Minor GC 和 Full GC
* Minor GC : 回收新生代，因为新生代对象存活时间很短，因此 Minor GC 执行频繁，执行也快
* Full GC：回收新生代和老年代，老年代存活周期长，因此 Full GC 执行很少，会比 Minor GC 耗时长很多

## 内存分配策略
### 1. 对象优先分配在 Eden
大多数情况下分配在 Eden，当 Eden 空间不足时，发起 Minor GC。

### 2. 大对象直接进入老年代
当有需要连续内存空间的对象时候，可能会提前触发垃圾收集以保证有足够的空间分配给新对象。   
通过设置 -XX:PretenureSizeThreshold 参数，当对象大于该值时直接赋值在老年代中，避免内存空间中大量复制。

### 3. 长期存活对象进入老年代
为对象定义年龄计数器，每在 Minor GC 中存活一次，年龄就加一，增大到一定年龄就会进入老年代中。   
-XX:MaxTenuringThreshold 来定义年龄阈值。

### 4. 动态对象年龄判断
如果 Survivor 区中同年龄的对象总和大于 Survivor 区大小的一般，那么所有年龄大于等于该年龄的对象都可以进入老年区。

### 5. 空间分配担保
发生 Minor GC 前，检查老年代中最大连续可用空间是否大于新生代所有对象的大小，如果是的话那么本次 Minor GC 是安全的。   
如果不成立的话虚拟机会查看 HandlePromotionFailure 的值是否允许担保失败，如果允许的话会继续检查老年代中最大连续可用空间是否大于历次晋升到老年代对象的总大小的平均值，如果大于则进行一次 Minor GC，否则不允许冒险，直接进行 Full GC。


## Full GC 的触发条件
对于 Minor GC ，触发条件非常简单， Eden 空间满了就触发一次。    
对于 Full GC 相对复杂很多，如下。

### 1. 调用 Sysytem.gc()
建议虚拟机执行一次 Full GC ，无法直接触发 Full GC ，不建议该方法。

### 2. 老年代空间不足
老年代空间不足大部分是因为大对象可以配置直接进入老年代，年龄到了晋升到老年代。   
避免创建大对象和数组，或者通过 -Xmn 调整新生代大小，让对象在新生代被消化，或者上调年龄让对象在新生代多存活一段时间。   

### 3. 空间分配担保失败
使用复制算法的 Minor GC 需要老年代空间做担保，担保失败则执行 Full GC。

### 4. 1.7 及以前的永生代空间不足
1.7 以前 HotSpot 的方法区用的是永久代实现的，其中存放 Class的信息、常量、静态变量等数据。    
当系统中要加载的类、反射的类和调用的方法较多时，永久代会被占满，未配置 CMS GC 的情况下就会发生 Full GC。如果经过 Full GC 仍然会受不了，抛出 OutOfMemory 异常。    

### 5. Concurrent Mode Failure
执行 CMS GC 过程中同时有对象要放入老年代，而此时老年代空间不足 (可能是 GC 过程中浮动垃圾导致的暂时性空间不足)，会导致 Concurrent Mode Failure 错误，触发 Full GC


# 类加载机制
类是在运行期间第一次使用时运动加载的，不是一次性加载所有类。    

## 类的生命周期

![](assets/Java虚拟机-b0e9af42.png)

包括七个阶段:
* 加载（Loading）
* 验证（Verification）
* 准备（Preparation）
* 解析（Resolution）
* 初始化（Initialization）
* 使用（Using）
* 卸载（Unloading）


## 类加载过程
包括了类加载、验证、准备、解析和初始化 5 个阶段。

### 1. 加载
加载是类加载的一个阶段没注意不要混淆。   
加载过程完成以下三件事：
* 通过类的完全限定名称获取定义该类的二进制流
* 将该字节流表示的静态存储结构转换为方法区的运动时存储接口
* 在内存中生成一个代表该类的 Class 对象，作为方法区中该类各种数据的访问入口

其中二进制字节流可以从以下方式中获取：
* 从 ZIP 包读取，成为 JAR、EAR、WAR 格式的基础
* 从网络中获取，最典型的是 Applet
* 运行时计算生产，例如动态代理技术，在 java.lang.reflect.Proxy 使用 ProxyGenerator.generateProxyClass 的 代理类的二进制字节流
* 由其他文件生产，例如由 JSP 文件生成对应的 Class 类

### 2. 验证
确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机的自身安全

### 3. 准备
类变量是被 static 修饰的变量，准备阶段为类变量分配内存并设置初始值，使用的是方法区的内存。   
实例变量不会在这阶段分配内存，而是在对象实例化时随着对象一起分配在堆中。应该注意到，实例化不是类加载的一个过程，类加载发生在所有实例化操作之前，且只执行一次，实例化会进行很多次    
类初始化时如果类变量是对象，那么一般会初始化为默认值，int 则初始化为 0.

    public static int value = 123;

如果是常量，则初始化为表达式定义的值，下列初始化成 123

    public static final int value = 123;


### 4. 解析
将常量池的符号引用替换为直接引用的过程。    
其中解析过程在某些情况下可以在初始化阶段之后再开始，这是为了支持 Java 的动态绑定

### 5. 初始化
初始化阶段才真正执行类中定义的 Java 程序代码。初始化阶段是虚拟机执行类构造器 <clinit>() 方法的过程，准备阶段，类变量已经获得了初始值，在初始化阶段，根据程序员通过程序定制的主观计划去操作类变量和其他资源       
<clinit>() 是由编译器自动收集类中所有类变量的复制动作和静态语句块中的语句合并产生的，编译器收集的顺序由语句在源文件中出现的顺序决定。静态语句块只能访问到定义在它之前的类变量，定义在他之后的类变量只能赋值，不能访问。

    public class Test {
      static {
         i = 0; //可以为后面的 i 赋值
         System.out.println(i); // 但无法访问后面的值
      }
      static int i = i;
    }

由于父类的 <clinit>() 方法先执行，也就意味着父类中定义的静态语句快的执行要优先于子类。

    static class  Parent {
      public staitc int A = 1;
      static {
        A = 2;
      }
    }

    static class Sub extends Parent {
      public static int B = A;
    }

    public static void main(String[] args) {
      System.out.println(Sub.B);
    }

接口中不可以使用动态语句块，但仍然有类变量初始化的复制操作，因此接口与类一样都会生成 <clinit>() 方法。但执行接口的 <clinit>() 方法不需要优先执行父类的，只有当父接口中定义的变量使用时，父接口才会初始化。另外，接口的实现类在初始化时也不会执行接口的 <clinit>() 方法。    

虚拟机会保证一个类的 <clinit>() 方法在多线程环境下被正确的加锁和同步，如果多个线程同事初始化一个类，只会有一个线程实际去执行，其他线程会则塞等待，直到活动线程执行 <clinit()> 方法完毕，


## 类初始化时机
### 1. 主动引用
虚拟机规范中并没有强制约束合适进行加载，只规定以下情况必须对类进行初始化：
* 遇到 new、getstatic、putstatic、invokestatic 这四条字节码指令时，如果类没有初始化就需要先初始化。最常见的场景有： 使用 new 实例化对象的时候；读取或设置一个类的静态字段 (除了被 fianl 修饰的字段和在编译期把结果放入长两次的静态字段除外)；调用类的静态方法时
* 使用 java.lang.reflect 包的方法对类进行反射调用时，未初则初
* 初始化一个类的时候，其父类未初则初
* 当虚拟机启动时，用户指定一个要执行的主类 (包含 main() 方法的那个类)，先初始化这个主类    
* 使用 1.7 的动态语言支持时，如果一个 java.lang.invoke.MethodHandle 实例最后的解析结果为 REF_getStatic，REF_putStatic，REF_invokeStatic 的方法句柄，并且这个方法句柄对应的类未初则初

### 2. 被动引用
被动引用常有的场景：
* 通过子类应用父类的静态字段，不会导致子类的初始化

      SYstem,out,println(Subclass.value)

* 通过数组定义来引用类，不会触发此类的初始化。该过程会对数组类进行初始化，数组类是一个由虚拟机自动生成的。直接继承自 Object 的子类，其中包含了数组的属性和方法。   
* 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。

## 类和类记载器
两个类相等，需要类本身相等，并且使用同一个类加载器进行加载。这是因为每一个类加载器都拥有一个独立的类名称空间。     
这里的相等，包括类的 Class 对象的 equals() 方法、isAssignableFrom() 方法、instance() 方法的返回结果为 true.也包括使用 instanceof 关键字做对象所属关系判定结果为 true。

## 类加载器分类
从 Java 虚拟机的角度来讲，只存在以下两种不同的类加载器：
* 启动类加载器 (Bootstrap ClassLoader)  使用 C++ 实现，是虚拟机自身的一部分；
* 其他类加载器 使用 Java 实现，独立于虚拟机，继承自抽象类 java.lang.ClassLoader

从 Java 开发人员的角度看，类加载器可以划分的更细致一些:
* 启动类加载器 此类加载器负责将存放在 <JRE_HOME>/lib 目录中的活着呗 -Xbootclasspath 参数指定的路径中，并且是虚拟机识别的 (名称不符合的也不会被加载)，类库加载到虚拟机内存中。启动类加载器无法直接被 Java 程序引用，用户在编写自定义类加载器时，需要把任务委派给启动类加载器时，直接用 null 代替即可
* 扩展类加载器 这个类加载器是由 ExtClassloader 实现的。他负责将 <JAVA_HOME>/lib/ext 或者被 java.ext.dir 系统变量指定路径中的所有类库加载到内存中，开发者直接使用扩展类加载器
* 应用程序类加载器 这个类加载器是由 AppClassLoader 实现的，由于这个类加载器时 ClassLoader 中的 getSystemClassLoader() 方法的返回值，因此一般称为系统类加载器。他负责加载用户类路径 (ClassPath) 上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况这个加载器就是程序中默认的类加载器。

## 双亲委派模型
应用程序是由三种类加载器互相配合从而实现类加载，除此之外还可以加入自己定义的类加载器。   
下图展示了类加载器之间的层次关系，成为双亲委派模型 (Parents Delegation Model)。除了顶层的启动类加载器外，其他的类加载器都要有自己的父类加载器。这里的父子关系一般通过组合关系实现，而不是继承关系

![](assets/Java虚拟机-a688688e.png)

### 1. 工作过程
一个类加载起首先将类加载请求转发到父类加载器，只有当父类加载器无法完成时才尝试自己加载。

### 2. 好处
使得 Java 类随着它的类加载器一起具有一种带有优先级的层次关系，从而使得基础类得到统一。    
例如所有加载器中的 Object 类，启动类加载器的优先级最高，程序中所有的 Object 都是它。

### 3. 实现
先检查是否加载过，没有则让父类加载器去加载，当父类加载器加载失败时抛出 ClassNotFoundException，此时尝试自己去加载

    public abstract class ClassLoader {
        // The parent class loader for delegation
        private final ClassLoader parent;

        public Class<?> loadClass(String name) throws ClassNotFoundException {
            return loadClass(name, false);
        }

        protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
            synchronized (getClassLoadingLock(name)) {
                // First, check if the class has already been loaded
                Class<?> c = findLoadedClass(name);
                if (c == null) {
                    try {
                        if (parent != null) {
                            c = parent.loadClass(name, false);
                        } else {
                            c = findBootstrapClassOrNull(name);
                        }
                    } catch (ClassNotFoundException e) {
                        // ClassNotFoundException thrown if class not found
                        // from the non-null parent class loader
                    }

                    if (c == null) {
                        // If still not found, then invoke findClass in order
                        // to find the class.
                        c = findClass(name);
                    }
                }
                if (resolve) {
                    resolveClass(c);
                }
                return c;
            }
        }

        protected Class<?> findClass(String name) throws ClassNotFoundException {
            throw new ClassNotFoundException(name);
        }
    }

## 自定义类加载器
以下代码中的 FileSystemClassLoader 是自定义类加载器，继承自 java.lang.ClassLoader，用于加载文件系统上的类。它首先根据类的全名在文件系统上查找类的字节代码文件（.class 文件），然后读取该文件内容，最后通过 defineClass() 方法来把这些字节代码转换成 java.lang.Class 类的实例。

    java.lang.ClassLoader 的 loadClass() 实现了双亲委派模型的逻辑，自定义类加载器一般不去重写它，但是需要重写 findClass() 方法。

    public class FileSystemClassLoader extends ClassLoader {

        private String rootDir;

        public FileSystemClassLoader(String rootDir) {
            this.rootDir = rootDir;
        }

        protected Class<?> findClass(String name) throws ClassNotFoundException {
            byte[] classData = getClassData(name);
            if (classData == null) {
                throw new ClassNotFoundException();
            } else {
                return defineClass(name, classData, 0, classData.length);
            }
        }

        private byte[] getClassData(String className) {
            String path = classNameToPath(className);
            try {
                InputStream ins = new FileInputStream(path);
                ByteArrayOutputStream baos = new ByteArrayOutputStream();
                int bufferSize = 4096;
                byte[] buffer = new byte[bufferSize];
                int bytesNumRead;
                while ((bytesNumRead = ins.read(buffer)) != -1) {
                    baos.write(buffer, 0, bytesNumRead);
                }
                return baos.toByteArray();
            } catch (IOException e) {
                e.printStackTrace();
            }
            return null;
        }

        private String classNameToPath(String className) {
            return rootDir + File.separatorChar
                    + className.replace('.', File.separatorChar) + ".class";
        }
    }
