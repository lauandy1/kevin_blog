---
layout: post
title: mybatis缓存机制
author: andy
tags:  cache
categories:  mybatis
excerpt: mybatis一级缓存、二级缓存
---

* TOC
{:toc}

# 前言
MyBatis 提供了缓存机制。持久层缓存的目的，为了降低应用层对物理数据源（如：数据库）访问的频次，提高应用程序整体性能。缓存将复制数据物理数据源中的数据（或加工后的数据），程序运行时直接从缓存读写数据，在特定的时刻或事件发生时会同步缓存和物理数据源的数据。

# 概要
## 一级缓存
Mybatis提供了一级缓存的方案来优化在数据库会话间重复查询的问题。实现的方式是每一个SqlSession中都持有了自己的缓存，一种是SESSION级别，即在一个Mybatis会话中执行的所有语句，都会共享这一个缓存。一种是STATEMENT级别，可以理解为缓存只对当前执行的这一个statement有效。
### 一级缓存配置

	<setting name="localCacheScope" value="SESSION"/>

## 二级缓存
在上文中提到的一级缓存中，其最大的共享范围就是一个SqlSession内部，那么如何让多个SqlSession之间也可以共享缓存呢，答案是二级缓存。 当开启二级缓存后，会使用CachingExecutor装饰Executor，在进入后续执行前，先在CachingExecutor进行二级缓存的查询，具体的工作流程如下所示。

![mybaits-cache-process.png](/images/mybatis/mybaits-cache-process.png)

在二级缓存的使用中，一个namespace下的所有操作语句，都影响着同一个Cache，即二级缓存是被多个SqlSession共享着的，是一个全局的变量。 当开启缓存后，数据的查询执行的流程就是 二级缓存 -> 一级缓存 -> 数据库。

### 二级缓存配置

要正确的使用二级缓存，需完成如下配置的。


1 在Mybatis的配置文件中开启二级缓存。


    <setting name="cacheEnabled" value="true"/>


2 在Mybatis的映射XML中配置cache或者 cache-ref 。


    <cache/>


cache标签用于声明这个namespace使用二级缓存，并且可以自定义配置。


    type: cache使用的类型，默认是PerpetualCache，这在一级缓存中提到过。

    eviction: 定义回收的策略，常见的有FIFO，LRU。

    flushInterval: 配置一定时间自动刷新缓存，单位是毫秒

    size: 最多缓存对象的个数

    readOnly: 是否只读，若配置可读写，则需要对应的实体类能够序列化。

    blocking: 若缓存中找不到对应的key，是否会一直blocking，直到有对应的数据进入缓存。


    <cache-ref namespace="mapper.StudentMapper"/>


cache-ref代表引用别的命名空间的Cache配置，两个命名空间的操作使用的是同一个Cache。

# 总结
Mybatis的二级缓存相对于一级缓存来说，实现了SqlSession之间缓存数据的共享，同时粒度更加的细，能够到Mapper级别，通过Cache接口实现类不同的组合，对Cache的可控性也更强。

Mybatis在多表查询时，极大可能会出现脏数据，有设计上的缺陷，安全使用的条件比较苛刻。

在分布式环境下，由于默认的Mybatis Cache实现都是基于本地的，分布式环境下必然会出现读取到脏数据，需要使用集中式缓存将Mybatis的Cache接口实现，有一定的开发成本，不如直接用Redis，Memcache实现业务上的缓存就好了。

# 全文总结
本文介绍了Mybatis的基础概念，Mybatis一二级缓存的使用及源码分析，并对于一二级缓存进行了一定程度上的总结。 最终的结论是Mybatis的缓存机制设计的不是很完善，在使用上容易引起脏数据问题，个人建议不要使用Mybatis缓存，在业务层面上使用其他机制实现需要的缓存功能，让Mybatis老老实实做它的ORM框架就好了哈哈。

# 参考 
Mybatis 缓存特性的使用及源码分析（ImportNew）
