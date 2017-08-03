---
layout: post
title: B树和B+树
author: andy
tags: structure algorithm
categories:  structure algorithm
excerpt: B树和B+树原理和应用
---

* TOC
{:toc}

# B树
B-树即是B树。
二叉查找树查询的复杂度是o(logN)，性能已经足够好。
从算法逻辑上来讲，二叉查找树查找速度和比较次数都是最小，但是，我们不得不考虑一个现实问题，磁盘IO。
数据库索引是存储在磁盘上的，当数据量比较大时，索引大小可能有几个G甚至更多。
当我们利用索引查询时，能把整个索引全部加载到内存吗？显然不可能。
能做的只有逐一加载每一个磁盘页，这里的磁盘页对应着索引树的节点。

![btree-disk.jpeg](/images/algorithm/btree/btree-disk.jpeg)



如果我们用二叉查找树作为索引结构，假设树的高度是4，查找的值是10，流程如下：

![2tree-0.jpeg](/images/algorithm/btree/2tree-0.jpeg)

第一次磁盘IO

![2tree-1.jpeg](/images/algorithm/btree/2tree-1.jpeg)

第二次磁盘IO

![2tree-2.jpeg](/images/algorithm/btree/2tree-2.jpeg)

第三次磁盘IO

![2tree-3.jpeg](/images/algorithm/btree/2tree-3.jpeg)

第四次磁盘IO

![2tree-4.jpeg](/images/algorithm/btree/2tree-4.jpeg)

可以看出，最坏情况下，磁盘IO次数等于索引树的高度。

既然如此，为了减少磁盘IO次数，我们需要把原本瘦高的树结构变的矮胖，这就是B树的特征之一。

## B树的特征
下面来具体介绍一下B-树（Balance Tree），一个m阶的B树具有如下几个特征：
	1.根结点至少有两个子女。

	2.每个中间节点都包含k-1个元素和k个孩子，其中 m/2 <= k <= m

	3.每一个叶子节点都包含k-1个元素，其中 m/2 <= k <= m

	4.所有的叶子结点都位于同一层。

	5.每个节点中的元素从小到大排列，节点当中k-1个元素正好是k个孩子包含的元素的值域分划。

以一个3阶B树为例，来看看B树的具体结构。树中元素和刚才的二叉查找树是一样的。

![btree-demo.jpeg](/images/algorithm/btree/btree-demo.jpeg)

这颗树中，重点看(2,6)节点。该节点有两个元素2和6，又有三个孩子1，（3，5），8。
其中，1小于元素2，（3，5）在元素2，6之间，8大于（3，5），正好符合刚列的几条特征。

下面演示一下B树的查询过程。

第一次磁盘IO

![btree-1.jpeg](/images/algorithm/btree/btree-1.jpeg)

在内存中定位（和9比较）

![btree-2.jpeg](/images/algorithm/btree/btree-2.jpeg)

第二次磁盘IO

![btree-3.jpeg](/images/algorithm/btree/btree-3.jpeg)

在内存中定位（和2，6比较）

![btree-4.jpeg](/images/algorithm/btree/btree-4.jpeg)

第三次磁盘IO

![btree-5.jpeg](/images/algorithm/btree/btree-5.jpeg)

在内存中定位（和3，5比较）

![btree-6.jpeg](/images/algorithm/btree/btree-6.jpeg)

通过整个流程我们可以看出，B树在查询中的比较次数其实不比二叉查找树少，尤其当单一节点中元素数量很多时。

可是相比磁盘IO的速度，内存中的比较耗时几乎可以忽略。所以，只要树的高度足够低，IO次数足够少，就可以提升查找性能。

相比之下，节点内部元素多一些也没有关系，仅仅是多了几次内存交互，只要不超过磁盘页的大小即可。这就是B树的优势之一。

查询的过程明白了，那么B树是如何做插入和删除的呢？

B树插入新节点的过程比较复杂，而且分成很多情况。我们只举一个最典型的例子，假如要插入的元素值是4.

自顶向下查找4的位置，发现4应该插入到节点元素3，5之间。

![btree-insert.jpeg](/images/algorithm/btree/btree-insert.jpeg)

节点3，5已经是两元素节点，无法再增加。父亲节点 2， 6 也是两元素节点，也无法再增加。根节点9是单元素节点，可以升级为两元素节点。于是拆分节点3，5与节点2，6，让根节点9升级为两元素节点4，9。节点6独立为根节点的第二个孩子。

![btree-insert1.jpeg](/images/algorithm/btree/btree-insert1.jpeg)

就为了插入一个元素，引起B树那么多节点发生连锁改变。
确实有些麻烦，但也正是如此，让B树能够始终维持多路平衡。这也是B树一大优势，自平衡。

下面看看删除过程，删除元素11。

自顶向下查找元素11的节点位置。

![btree-delete.jpeg](/images/algorithm/btree/btree-delete.jpeg)

删除11后，节点12只有一个孩子，不符合B树规范。因此找出12,13,15三个节点的中位数13，取代节点12，而节点12自身下移成为第一个孩子。（这个过程称为左旋）

![btree-delete1.jpeg](/images/algorithm/btree/btree-delete1.jpeg)

![btree-delete2.jpeg](/images/algorithm/btree/btree-delete2.jpeg)

以上就是B树插入和删除，也是学习B树最绕的部分。

## B树的应用

B树主要应用于文件系统以及部分数据库索引，比如著名的非关系型数据库Mongodb。

而大部分关系型数据库，比如mysql，则使用B+树作为索引。

## 参考 
漫画：什么是 B- 树？ 算法爱好者

# B+树

# B+树特征
一个m阶的B+树具有如下几个特征：

	1.有k个子树的中间节点包含有k个元素（B树中是k-1个元素），每个元素不保存数据，只用来索引，所有数据都保存在叶子节点。

	2.所有的叶子结点中包含了全部元素的信息，及指向含这些元素记录的指针，且叶子结点本身依关键字的大小自小而大顺序链接。

	3.所有的中间节点元素都同时存在于子节点，在子节点元素中是最大（或最小）元素。

B+树例子

![bplustree-demo.jpeg](/images/algorithm/bplustree/bplustree-demo.jpeg)

这是什么怪树，不但节点之间含有重复元素，而且叶子节点还用指针连在一起。

这些正是B+树的几个特点。首先，每个父节点的元素都出现在子节点中，是子几点的最大（或最小）元素。

![bplustree-demo2.jpeg](/images/algorithm/bplustree/bplustree-demo2.jpeg)

在上面这棵树中，根节点元素8是子节点2，5，8中最大元素，也是叶子节点6，8的最大元素。

根节点元素15是子节点11，15的最大元素，也是叶子节点13，15的最大元素。

需要注意的是，根节点的最大元素，也就等同于整个B+树的最大元素，以后无论插入删除多少元素，始终要保持最大元素在根节点中。

至于叶子节点，由于父节点元素都出现在子节点，因此所有叶子节点包含了全量元素信息。

并且，每个叶子节点都带有指向下一个节点的指针，形成一个有序链表。

![bplustree-demo3.jpeg](/images/algorithm/bplustree/bplustree-demo3.jpeg)

## 卫星数据
B+树还有一个特点，这个特点是在索引之外，确是至关重要的特点，那就是卫星数据的位置。

所谓卫星数据，指的是索引元素所指向的数据记录，比如数据库中的某一行。在B树中，无论中间节点还是叶子节点都带有卫星数据。

B树中的卫星数据：

![btree-satelite.jpeg](/images/algorithm/bplustree/btree-satelite.jpeg)

而在B+树中，只有叶子节点带有卫星数据，其余中间节点仅仅是索引，没有任何数据关联。

B+树中的卫星数据：

![bplustree-satelite.jpeg](/images/algorithm/bplustree/bplustree-satelite.jpeg)

需要补充的是，在数据库的聚集索引（Clustered Index）中，叶子节点直接包含卫星数据。在非聚集索引（NonClustered Index）中，叶子节点带有指向卫星数据的指针。

## B+树的好处

B+树的好处主要体现在查询性能上。下面通过单行查询和范围查询来做分析。

在单元素查询时候，B+树会自顶向下逐层查找节点，最终找到匹配的叶子节点。比如我们要找的是元素3.

第一次磁盘IO：

![bplus-1.jpeg](/images/algorithm/bplustree/bplus-1.jpeg)

第二次磁盘IO：

![bplus-2.jpeg](/images/algorithm/bplustree/bplus-2.jpeg)

第三次磁盘IO：

![bplus-3.jpeg](/images/algorithm/bplustree/bplus-3.jpeg)

查询流程看起来跟B树差不多。

不，有两点不同。首先，B+树的中间节点没有卫星数据，所以同样大小的磁盘页可以容纳更多的节点元素。

这就意味着，数据量相同的情况下，B+树的结构比B树更加矮胖，因此查询时IO次数更少。

其次，B+树的查询必须最终查找到叶子节点，而B树只要找到匹配的元素即可，无论元素处于中间节点还是叶子节点。
因此，B树的查找性能不稳定，最好情况是只查根节点，最坏情况是查到叶子节点。而B+树每一次查找是稳定的。

下面我们看看范围查询。B树如何做范围查询？只能靠繁琐的中序遍历。比如我们要查询3到11的元素。

B树的查找过程：

自顶向下，查找到范围的下限（3）：

![bfind-1.jpeg](/images/algorithm/bplustree/bfind-1.jpeg)

中序遍历到元素6：

![bfind-2.jpeg](/images/algorithm/bplustree/bfind-2.jpeg)

中序遍历到元素8：

![bfind-3.jpeg](/images/algorithm/bplustree/bfind-3.jpeg)

中序遍历到元素9：

![bfind-4.jpeg](/images/algorithm/bplustree/bfind-4.jpeg)

中序遍历到元素11，遍历结束：

![bfind-5.jpeg](/images/algorithm/bplustree/bfind-5.jpeg)

B树的范围查询确实很繁琐，反观B+树的范围查询，简单的多，只要在链表上遍历即可。

B+树的范围查找过程

自顶向下，查找到范围的下限（3）：

![bplusfind-1.jpeg](/images/algorithm/bplustree/bplusfind-1.jpeg)

通过链表指针，遍历到元素6, 8：

![bplusfind-2.jpeg](/images/algorithm/bplustree/bplusfind-2.jpeg)

通过链表指针，遍历到元素9, 11，遍历结束：

![bplusfind-3.jpeg](/images/algorithm/bplustree/bplusfind-3.jpeg)

果然比B树的中序遍历要简单的多。

综合起来，B+树比B树的优势有三个：

	1.IO次数更少。
	2.查询性能稳定。
	3.范围查询简单。

至于B+树的插入和删除，过程与B树大同小异，这里不描述了。

最好，总结一下B+树的特征和优势：

## B+树的特征：

	1.有k个子树的中间节点包含有k个元素（B树中是k-1个元素），每个元素不保存数据，只用来索引，所有数据都保存在叶子节点。

	2.所有的叶子结点中包含了全部元素的信息，及指向含这些元素记录的指针，且叶子结点本身依关键字的大小自小而大顺序链接。

	3.所有的中间节点元素都同时存在于子节点，在子节点元素中是最大（或最小）元素。


## B+树的优势：

	1.单一节点存储更多的元素，使得查询的IO次数更少。

	2.所有查询都要查找到叶子节点，查询性能稳定。

	3.所有叶子节点形成有序链表，便于范围查询。

## 参考
漫画：什么是 B+ 树？ 算法爱好者












