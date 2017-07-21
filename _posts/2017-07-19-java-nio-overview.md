---
layout: post
title: Java NIO基础介绍
author: andy
tags:  nio
categories:  nio
excerpt: nio介绍、select、poll、epoll
---

* TOC
{:toc}
# NIO概览
## 什么是NIO
 NIO是New I/O的简称，与旧式的基于流的I/O方法相对，从名字看，它表示新的一套Java I/O标 准。它是在Java 1.4中被纳入到JDK中的，并具有以下特性： 

* NIO是基于块（Block）的，它以块为基本单位处理数据 （硬盘上存储的单位也是按Block来存储，这样性能上比基于流的方式要好一些）
* 为所有的原始类型提供（Buffer）缓存支持 
* 增加通道（Channel）对象，作为新的原始 I/O 抽象
* 支持锁（我们在平时使用时经常能看到会出现一些.lock的文件，这说明有线程正在使用这把锁，当线程释放锁时，会把这个文件删除掉，
    这样其他线程才能继续拿到这把锁）和内存映射文件的文件访问接口 
* 提供了基于Selector的异步网络I/O 

![channel-buffer.png](/images/nio/channel-buffer.png)

所有的从通道中的读写操作，都要经过Buffer，而通道就是io的抽象，通道的另一端就是操纵的文件。 

## NIO几大组件
### Selector
多路复用选择器，基于“事件驱动”，其核心就是通过Selector来轮询注册在其上的Channel，当发现某个或多个Channel处于就绪状态后，从阻塞状态返回就绪的Channel的SelectionKey集合，进行I/O操作。

创建多路复用器并启动线程

	Selector selector=Selector.open();
	new Thread(new ReactorTask()).start();
 
创建Channel

	// 打开ServerSocketChannel，用于监听客户端的连接
	ServerSocketChannel ssc=ServerSocketChannel.open();
	//设置连接为非阻塞模式
	ssc.configureBlocking(false);
	//绑定监听端口
	ServerSocket ss=ssc.socket();
	ss.bind(new InetSocketAddress(InetAdderss.getByName("ip"),port));
	//将ServerSocketChannel注册到多路复用器Selector上，监听ACCEPT事件
	ssc.register(selector,SelectionKey.OP_ACCEPT);

等待客户端的连接

	while (true) {
            // selector.select是阻塞的，一直等到有客户端连接过来才返回，然后会检查发生的是哪一种事件，然后根据不同的事件做不同的操作
            selector.select();
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            Iterator<SelectionKey> it = selectionKeys.iterator();
            while (it.hasNext()) {
                SelectionKey key = it.next();
                if (key.isAcceptable()) {
                    // 处理新接入的请求消息
                    ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
                    SocketChannel sc = ssc.accept();
                    sc.configureBlocking(false);
                    // 注册读事件
                    sc.register(selector, SelectionKey.OP_READ);
                }
                if (key.isReadable()) {
                    // 处理读请求
                    SocketChannel sc = (SocketChannel) key.channel();
                    ByteBuffer readBuffer = ByteBuffer.allocate(1024);
                    int readBytes = sc.read(readBuffer);
                    if (readBytes > 0) {
                        readBuffer.flip();
                        byte[] bytes = new byte[readBuffer.remaining()];
                        readBuffer.get(bytes);
                        System.out.println(new String(bytes, "UTF-8"));
                    }
                }

            }
        }

### Channel
Channel是NIO对IO抽象的一个新概念，NIO在进行IO时需要创建一个Channel对象，是双向的，不象Standard IO分为输入流和输出流。

### Buffer

Buffer和Channel都是一起使用的，每次都是从一个Channel中读出一个Buffer或者把一个Buffer写入到一个Channel中。

				// 处理读请求
                    SocketChannel sc = (SocketChannel) key.channel();
                    ByteBuffer readBuffer = ByteBuffer.allocate(1024);
                    int readBytes = sc.read(readBuffer);
                    if (readBytes > 0) {
                        readBuffer.flip();
                        byte[] bytes = new byte[readBuffer.remaining()];
                        readBuffer.get(bytes);
                        System.out.println(new String(bytes, "UTF-8"));
                    }

Buffer有3个重要的属性

* position 正整数，指向Buffer中下一个要读取或写入的字节位置
* limit 正整数，指向Buffer中的某个位置，在IO时只读写下标小于limit的字节内容
* capacity 正整数，Buffer所能容纳的最大字节数

0 <= position <= limit <= capacity

![nio-buffer-1.png](/images/nio/nio-buffer-1.png)

初始状态：
从Channel中读入5个字到ByteBuffer 

![nio-buffer-2.png](/images/nio/nio-buffer-2.png)

flip()，准备写入或输出

	public final Buffer flip() {
        limit = position;
        position = 0;
        mark = -1;
        return this;
    }

![nio-buffer-3.png](/images/nio/nio-buffer-3.png)

输出内容后，position就移动到跟limit相同的位置上 

![nio-buffer-4.png](/images/nio/nio-buffer-4.png)

ByteBuffer如果要重复利用，需要清理，position和limit回到初始状态时的位置，然后可以接着用这个Buffer来读写数据，不需要再New 新的Buffer

	public final Buffer clear() {
        position = 0;
        limit = capacity;
        mark = -1;
        return this;
    }

![nio-buffer-5.png](/images/nio/nio-buffer-5.png)

比较好的基于NIO的开源框架（Netty）

优点：

    api简单，开发门槛低
    功能强大，内置了多种编码、解码功能
    与其它业界主流的NIO框架对比，netty的综合性能最优
    社区活跃，使用广泛，经历过很多商业应用项目的考验
    定制能力强，可以对框架进行灵活的扩展




### Jdk中buffer

![nio-buffer.png](/images/nio/nio-buffer.png)

Java中Buffer的实现。基本的数据类型都有它对应的Buffer。

	package test;

	import java.io.File;
	import java.io.FileInputStream;
	import java.nio.ByteBuffer;
	import java.nio.channels.FileChannel;

	public class Test {
		public static void main(String[] args) throws Exception {
			FileInputStream fin = new FileInputStream(new File(
					"d:\\temp_buffer.tmp"));
			FileChannel fc = fin.getChannel();
			ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
			fc.read(byteBuffer);
			fc.close();
			byteBuffer.flip();//读写转换
		}
	}

总结下使用的步骤是：

1. 得到Channel

2. 申请Buffer

3. 建立Channel和Buffer的读/写关系

4. 关闭


# Linux IO模式及 select、poll、epoll详解
[https://segmentfault.com/a/1190000003063859](https://segmentfault.com/a/1190000003063859)
## 用户空间和内核空间
现在操作系统都是采用虚拟存储器，那么对32位操作系统而言，它的寻址空间（虚拟存储空间）为4G（2的32次方）。操作系统的核心是内核，独立于普通的应用程序，可以访问受保护的内存空间，也有访问底层硬件设备的所有权限。为了保证用户进程不能直接操作内核（kernel），保证内核的安全，操心系统将虚拟空间划分为两部分，一部分为内核空间，一部分为用户空间。针对linux操作系统而言，将最高的1G字节（从虚拟地址0xC0000000到0xFFFFFFFF），供内核使用，称为内核空间，而将较低的3G字节（从虚拟地址0x00000000到0xBFFFFFFF），供各个进程使用，称为用户空间。

## 文件描述符fd
文件描述符（File descriptor）是计算机科学中的一个术语，是一个用于表述指向文件的引用的抽象化概念。

文件描述符在形式上是一个非负整数。实际上，它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。在程序设计中，一些涉及底层的程序编写往往会围绕着文件描述符展开。但是文件描述符这一概念往往只适用于UNIX、Linux这样的操作系统。

## IO模式
刚才说了，对于一次IO访问（以read举例），数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间。所以说，当一个read操作发生时，它会经历两个阶段：
1. 等待数据准备 (Waiting for the data to be ready)
2. 将数据从内核拷贝到进程中 (Copying the data from the kernel to the process)

正式因为这两个阶段，linux系统产生了下面五种网络模式的方案。
- 阻塞 I/O（blocking IO）
- 非阻塞 I/O（nonblocking IO）
- I/O 多路复用（ IO multiplexing）
- 信号驱动 I/O（ signal driven IO）
- 异步 I/O（asynchronous IO）

注：由于signal driven IO在实际中并不常用，所以我这只提及剩下的四种IO Model。


# 参考资料
[https://my.oschina.net/hosee/blog/615269](https://my.oschina.net/hosee/blog/615269)

[https://github.com/lauandy1/technology-talk/blob/master/basic-knowledge/NIO.md](https://github.com/lauandy1/technology-talk/blob/master/basic-knowledge/NIO.md)






---





