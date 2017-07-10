---
layout: post
title: Jvm原理及垃圾回收算法概览
author: andy
tags:  jvm gc java8 
categories:  jvm
excerpt: jvm架构、垃圾回收算法、Java8中的改进
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


---





