---
layout: post
title: ArrayList
author: andy
tags:  collection
categories:  java
excerpt: ArrayList
---


* TOC
{:toc}

# ArrayList扩容原理
初始容量是10.
网上有的文章贴出的扩容代码是：

     int newCapacity = (oldCapacity * 3)/2 + 1;

估计是旧的源码了，我在1.7中看到如下，为啥是1.5倍就好解释了，还是为了效率，位运算比较快。
而且，ArrayList扩容不是很复杂，结合内存和效率的折中考虑，扩容为1.5倍。

     private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

# 为什么ArrayList的elementData是用transient修饰的？
看到ArrayList实现了Serializable接口，这意味着ArrayList是可以被序列化的，用transient修饰elementData意味着我不希望elementData数组被序列化。这是为什么？因为序列化ArrayList的时候，ArrayList里面的elementData未必是满的，比方说elementData有10的大小，但是我只用了其中的3个，那么是否有必要序列化整个elementData呢？显然没有这个必要，因此ArrayList中重写了writeObject方法：

每次序列化的时候调用这个方法，先调用defaultWriteObject()方法序列化ArrayList中的非transient元素，elementData不去序列化它，然后遍历elementData，只序列化那些有的元素，这样：


1、加快了序列化的速度


2、减小了序列化之后的文件大小