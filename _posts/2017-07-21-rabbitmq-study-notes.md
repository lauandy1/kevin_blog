---
layout: post
title: RabbitMQ学习笔记
author: andy
tags:  rabbitmq
categories:  rabbitmq
excerpt: RabbitMQ学习笔记
---

* TOC
{:toc}

# 工作队列

![rabbitmq-1.png](/images/rabbitmq/rabbitmq-1.png)

## Round-robin分发

> 一个队列，多个消费者消费，默认情况下会Round-robin分发。
发送5个消息，msg1-msg5，consumer端C1和C2接收到消息分别为：msg1、msg3、msg5和msg2、msg4。

## ack机制

> 默认情况下，消费者处理完消息后需要进行ack。broker会一直等待ack，不会有超时机制。

    * 如果consumer忘记发送ack，那么在consumer退出时消息会requeue到队列中。
    也就是说如果consumer保持连接，消息就不会requeue。
    但是，客户端忘记ack，broker由于不能释放未ack的消息，内存会持续消耗。

## 消息持久化

> 消息持久化机制不能完全保证消息不丢失。在rabbitmq接收到消息后，在持久化到磁盘之前，有一个短暂的时间窗口，
在这段时间内rabbitmq挂掉，消息会丢失。必须使用Publish confirm机制保证消息不丢失。

## 公平分发

> rabbitmq默认情况下，不管消费者忙不忙，都会把消息平均的分配到多个消费者，假如两个消费者，奇数消息都给了A，
偶数消息都给了B，奇数消息处理比较费时，而偶数消息可以快速处理。这样A会一直忙，B则会比较轻松。
为公平起见，可以设置basic.qos方法的prefetch_count=1，这样consumer只有在处理完一条消息后才会接收下一条进行处理。
如果没处理完，rabbitmq会把消息派发给其他的空闲的consumer。

# 发布/订阅

> 一个消息可以传递给多个消费者消费。

## Exchanges

> 生产者发送消息首先会到exchange，由exchange将消息分发给队列。
exchange四种类型：direct，topic，headers，fanout。
fanout类型的exchange会把消息广播给所有的队列。
之前的例子中没有用到exchange，是因为有一个名字为“”的默认的exchange。

参考资料
[https://www.rabbitmq.com/tutorials/tutorial-two-python.html](https://www.rabbitmq.com/tutorials/tutorial-two-python.html)







---





