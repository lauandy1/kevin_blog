---
layout: post
title: Jvm原理及垃圾回收算法概览
author: andy
tags:  jvm gc java8 
categories:  jvm
excerpt: jvm架构、垃圾回收算法、Hotspot垃圾收集器、Java8中的改进
---

* TOC
{:toc}

# Jvm架构

## Java平台架构
![JavaPlatform.gif](/images/jvm/JavaPlatform.gif)

## Jvm架构
![JvmStructure.gif](/images/jvm/JvmStructure.gif)
---


# 垃圾回收算法

### 标记清除算法
* 标记和清除两个阶段的效率都不高，还会产生内存碎片。

### 复制算法
* Eden区和其中一块survivor区一块，回收时把这两块区域还存活的对象一次性复制到另外一块survivor区。
* Hotspot虚拟机默认Eden和Survivor大小比例是8:1，只有10%的内存会浪费。新生代中98%对象是朝生夕死的。
* 如果另外一块survivor区空间不足，会直接进入老年代。

### 标记整理算法
* 老年代的对象存活率较高，如果采用复制算法要进行较多的复制操作，因此用标记整理算法。

### 分代收集算法

# 垃圾回收的时间点
* 安全点：STW(stop the world）会在一个安全点进行，所有的线程会到达这个safepoint进行阻塞。所有线程采用主动式中断，gc会设置标志位，而各个线程会轮询这个标志位，在最近的安全点阻塞。当所有的线程都阻塞后，开始进行gc。
* 安全区域：安全点无法解决阻塞状态的线程主动轮询的问题，需要安全区域的概念，这个区域内任意地方开始GC都是安全的，当线程进入safe region后会标识，在这段时间如果发生gc，线程要离开safe region时会检查系统是否已经完成了根节点枚举或者是整个gc过程，如果未完成，必须要等待收到可以安全离开的信号再继续执行。

# 垃圾收集器
## HotSpot垃圾收集器
在虚拟机规范中并没有对垃圾回收器如何实现具体介绍，因此每个厂商的垃圾回收器可能会完全不同，但是我们介绍的是基于JDK1.7之后的Hotspot虚拟机（包括前面对Java虚拟机的介绍也是基于jdk1.7版本的）。在Hotspot中，虚拟机的收集器主要有下：
![hotspotGc.jpeg](/images/jvm/hotspotGc.jpeg)
可以看到垃圾收集器是按对象的分代来划分的，可以用双箭头连接的垃圾收集器表示两者可以配合使用。可以看到新生代垃圾收集器有Serial、ParNew、Parallel Scavenge，G1，属于老年代的垃圾收集器有CMS、Serial Old、Parallel Old和G1.其中的G1是一种既可以对新生代对象也可以对老年代对象进行回收的垃圾收集器。然而，在所有的垃圾收集器中，并没有一种普遍使用的垃圾收集器。在不同的场景下，每种垃圾收集器有各自的优势。

### Serial收集器
Serial收集器是最基本、发展历史最悠久的收集器。它是一种单线程垃圾收集器，这就意味着在其进行垃圾收集的时候需要暂停其他的线程，也就是之前提到的”Stop the world“。虽然这个过程是在用户不可见的情况下把用户正常的线程全部停掉，听起来有点狠，这点是很难让人接受的。Serial、Serial Old收集器的工作示意图如下：
![jvm_serial.jpeg](/images/jvm/jvm_serial.jpeg)
尽管由以上不能让人接受的地方，但是Serial收集器还是有其优点的：简单而高效，对于限定单个CPU的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得较高的手机效率。到目前为止，Serial收集器依然是Client模式下的默认的新生代垃圾收集器。

### ParNew收集器
可ParNew收集器是Serial收集器的多线程版本，ParNew收集器的工作示意图如下：
![jvm_parnew.jpeg](/images/jvm/jvm_parnew.jpeg)
ParNew收集器是许多运行在Server模式下的虚拟机中首选的新生代收集器。除去性能因素，很重要的原因是除了Serial收集器外，目前只有它能与CMS收集器配合工作。
但是，在单CPU环境中，ParNew收集器绝对不会有比Serial收集器更好的效果，甚至由于存在线程交互的开销，该收集器在通过超线程技术实现的两个CPU的环境中都不能百分之百地保证可以超越Serial收集器。然而，随着可以使用的CPU的数量的增加，它对于GC时系统资源的有效利用还是很有好处的。

### Parallel Scavenge收集器

Parallel Scavenge收集器是新生代垃圾收集器，使用复制算法，也是并行的多线程收集器。与ParNew收集器相比，很多相似之处，但是Parallel Scavenge收集器更关注可控制的吞吐量。吞吐量越大，垃圾收集的时间越短，则用户代码则可以充分利用CPU资源，尽快完成程序的运算任务。

Parallel Scavenge收集器使用两个参数控制吞吐量：

    XX:MaxGCPauseMillis 控制最大的垃圾收集停顿时间

    XX:GCRatio 直接设置吞吐量的大小。

直观上，只要最大的垃圾收集停顿时间越小，吞吐量是越高的，但是GC停顿时间的缩短是以牺牲吞吐量和新生代空间作为代价的。比如原来10秒收集一次，每次停顿100毫秒，现在变成5秒收集一次，每次停顿70毫秒。停顿时间下降的同时，吞吐量也下降了。

除此之外，Parallel Scavenge收集器还可以设置参数-XX:+UseAdaptiveSizePocily来动态调整停顿时间或者最大的吞吐量，这种方式称为GC自适应调节策略，这点是ParNew收集器所没有的。

### Serial Old收集器

Serial Old收集器是Serial收集器的老年代版本，也是一个单线程收集器，采用“标记-整理算法”进行回收。其运行过程与Serial收集器一样。

Serial Old收集器的主要意义也是在于给Client模式下的虚拟机使用。如果在Server模式下，那么它主要还有两大用途：一种用途是在JDK 1.5以及之前的版本中与Parallel Scavenge收集器搭配使用，另一种用途就是作为CMS收集器的后备预案，在并发收集发生Concurrent Mode Failure时使用。

### Parallel Old收集器

Parallel Old收集器是Parallel Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法进行垃圾回收。其通常与Parallel Scavenge收集器配合使用，“吞吐量优先”收集器是这个组合的特点，在注重吞吐量和CPU资源敏感的场合，都可以使用这个组合。
![jvm_parallelOld.jpeg](/images/jvm/jvm_parallelOld.jpeg)

### CMS收集器

CMS收集器（Concurrent Mark Sweep）的目标就是获取最短回收停顿时间。在注重服务器的响应速度，希望停顿时间最短，则CMS收集器是比较好的选择。

整个执行过程分为以下4个步骤：

    初始标记
    并发标记
    重新标记
    并发清除

初始标记和重新标记这两个步骤仍然需要暂停Java执行线程，初始标记只是标记GC Roots能够关联到的对象，并发标记就是执行GC Roots Tracing的过程，而重新标记就是为了修正并发标记期间因用户程序执行而导致标记发生变动使得标记错误的记录。其执行过程如下：
![jvm_cms.jpeg](/images/jvm/jvm_cms.jpeg)
由上图可知，整个过程中耗时最长的并发标记和并发清除过程收集器线程都可以与用户线程一起工作，因此，总体上CMS收集器的内存回收过程是与用户线程一起并发执行的。

CMS的优点很明显：并发收集、低停顿。由于进行垃圾收集的时间主要耗在并发标记与并发清除这两个过程，虽然初始标记和重新标记仍然需要暂停用户线程，但是从总体上看，这部分占用的时间相比其他两个步骤很小，所以可以认为是低停顿的。

尽管如此，CMS收集器的缺点也是很明显的：

    对CPU资源太敏感，这点可以这么理解，虽然在并发标记阶段用户线程没有暂停，但是由于收集器占用了一部分CPU资源，导致程序的响应速度变慢

    CMS收集器无法处理浮动垃圾。所谓的“浮动垃圾”，就是在并发标记阶段，由于用户程序在运行，那么自然就会有新的垃圾产生，这部分垃圾被标记过后，CMS无法在当次集中处理它们（为什么？原因在于CMS是以获取最短停顿时间为目标的，自然不可能在一次垃圾处理过程中花费太多时间），只好在下一次GC的时候处理。这部分未处理的垃圾就称为“浮动垃圾”

    由于CMS收集器是基于“标记-清除”算法的，前面说过这个算法会导致大量的空间碎片的产生，一旦空间碎片过多，大对象就没办法给其分配内存,那么即使内存还有剩余空间容纳这个大对象，但是却没有连续的足够大的空间放下这个对象，所以虚拟机就会触发一次Full GC（这个后面还会提到）这个问题的解决是通过控制参数-XX:+UseCMSCompactAtFullCollection，用于在CMS垃圾收集器顶不住要进行FullGC的时候开启空间碎片的合并整理过程。

### G1收集器

G1（Garbage-First）收集器是现今收集器技术的最新成果之一，之前一直处于实验阶段，直到jdk7u4之后，才正式作为商用的收集器。

与前几个收集器相比，G1收集器有以下特点：

    并行与并发
    分代收集（仍然保留了分代的概念）
    空间整合（整体上属于“标记-整理”算法，不会导致空间碎片）
    可预测的停顿（比CMS更先进的地方在于能让使用者明确指定一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒）

此外，G1收集器将Java堆划分为多个大小相等的Region（独立区域），新生代与老年代都是一部分Region的集合，G1的收集范围则是这一个个Region（化整为零）。

G1的工作过程如下：

    初始标记（Initial Marking）
    并发标记（Concurrent Marking）
    最终标记（Final Marking）
    筛选回收（Live Data Counting and Evacuation）

初始标记阶段仅仅只是标记一下GC Roots能够直接关联的对象，并且修改TAMS（Next Top at Mark Start）的值，让下一阶段的用户程序并发运行的时候，能在正确可用的Region中创建对象，这个阶段需要暂停线程。并发标记阶段从GC Roots进行可达性分析，找出存活的对象，这个阶段食欲用户线程并发执行的。最终标记阶段则是修正在并发标记阶段因为用户程序的并发执行而导致标记产生变动的那一部分记录，这部分记录被保存在Remembered Set Logs中，最终标记阶段再把Logs中的记录合并到Remembered Set中，这个阶段是并行执行的，仍然需要暂停用户线程。最后在筛选阶段首先对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间制定回收计划。整个执行过程如下：
![jvm_g1.jpeg](/images/jvm/jvm_g1.jpeg)

## 垃圾收集器常用参数总结
![jvm_param.jpeg](/images/jvm/jvm_param.jpeg)

client/serrver端不同的GC方式：
![jvm_client_server.jpeg](/images/jvm/jvm_client_server.jpeg)

Sun JDK HotSpot虚拟机GC组合方式：
![jvm_combination.jpeg](/images/jvm/jvm_combination.jpeg)

# 参考
1、周志明，深入理解Java虚拟机：JVM高级特性与最佳实践，机械工业出版社


---





