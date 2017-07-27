---
layout: post
title: Redis基础介绍
author: andy
tags:  redis nosql cache
categories:  redis
excerpt: redis基本原理、数据类型、持久化方式、部署方式、常用场景等
---

* TOC
{:toc}

# redis数据类型

# redis过期键删除策略

## 定时删除
在设置键的过期时间的同时，创建一个定时器( timer ). 让定时器在键的过期时间来临时，立即执行对键的删除操作。
> 对内存友好，对CPU不友好。通过定时器实现，对cpu造成压力。

## 惰性删除
放任键过期不管，但是每次从键空间中获取键时，都检查取得的键是否过期，如果过期的话，就删除该键;如果没有过期，就返回该键。
> 对CPU友好，对内存不友好。长期不使用的键会造成内存泄露。

## 定期删除
每隔一段时间，程序就对数据库进行一次检查，删除里面的过期键。至于要删除多少过期键，以及要检查多少个数据库， 则由算法决定。
在这三种策略中，第一种和第三种为主动删除策略， 而第二种则为被动删除策略。
>从上面对定时删除和惰性删除的讨论来看，这两种删除方式在单一使用时都有明显的缺陷:
定时删除占用太多CPU 时间，影响服务器的响应时间和吞吐量。惰性删除浪费太多内存，有内存泄漏的危险。
定期删除策略是前两种策略的一种整合和折中:

## Redis的过期键删除策略
> 我们讨论了定时删除、惰性删除和定期删除三种过期键删除策略， Redis服务器实际使用的是惰性删除和定期删除两种策略: 通过配合使用这两种删除策略，服务器可以很好地在合理使用CPU时间和避免浪费内存空间之间取得平衡。


# redis部署方式
## setinel

![redis-sentinel.jpeg](/images/redis/redis-sentinel.jpeg)




---




