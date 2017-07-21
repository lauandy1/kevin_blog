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

> 一个队列，多个消费者消费，默认情况下会Round-robin分发。
发送5个消息，msg1-msg5，consumer端C1和C2接收到消息分别为：msg1、msg3、msg5和msg2、msg4。

> 默认情况下，消费者处理完消息后需要进行ack。broker会一直等待ack，不会有超时机制。

    * 如果consumer忘记发送ack，那么在consumer退出时消息会requeue到队列中。
    也就是说如果consumer保持连接，消息就不会requeue。
    但是，客户端忘记ack，broker由于不能释放未ack的消息，内存会持续消耗。

参考资料
[https://www.rabbitmq.com/tutorials/tutorial-two-python.html](https://www.rabbitmq.com/tutorials/tutorial-two-python.html)







---





