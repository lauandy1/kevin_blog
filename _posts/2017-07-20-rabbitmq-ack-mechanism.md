---
layout: post
title: RabbitMQ消息确认机制
author: andy
tags:  rabbitmq
categories:  rabbitmq
excerpt: 消息发布确认、消息消费确认
---

* TOC
{:toc}

# 消费者确认和发布者确认

> 消息发布后不能保证成功到达broker，并且成功接收和被消费者消费。因此需要一套机制来保证。
在AMQP 0-9-1中定义了消费者消息确认机制，broker给发布者发送确认是协议的扩展。

## 投递确认（Deliver Acknowledgements(Consumer)）

### 投递标识符：投递标签（Delivery Tags）

> 当一个消费者订阅了消息，rabbitmq会使用basic.deliver方法投递消息。这个方法携带了一个delivery tag用于标识在一个channel中的投递的唯一性。
因此，delivery tag的范围在channel中。
Delivery tag正整数单向递增，client方法确认投递会携带这个tag作为参数。

### 消费者确认模式
> * basic.ack 积极（positive）确认，表示消费者成功处理了消息
* basic.nack 消极（negative）确认，表示消费者没有成功处理消息，仍然删除消息
* basic.reject 相比back多了一个限制，表示消费者没有成功处理消息，仍然删除消息

> 积极确认指示rabbitmq记录消息成功投递，消极确认也是同样的效果。积极确认表示一个消息被成功处理，而消极确认表示消息未成功处理但仍然要删除。

    * 如果consumer忘记发送ack，那么在consumer退出时消息会requeue到队列中。
    也就是说如果consumer保持连接，消息就不会requeue。
    但是，客户端忘记ack，broker由于不能释放未ack的消息，内存会持续消耗。

### 一次确认多个投递
> 批量确认，为了减少网络负载。通过设置ack method的multiple属性为true，basic.reject没有这个属性，因此引入basic.nack作为协议扩展。
假如有delivery tags 5,6,7,8在channel Ch上未确认，当一个确认帧（acknowledgement frame）到达携带了delivery_tag为8并且multiple为true,
那么5,6,7,8都会被确认。

### Channel Prefetch Setting (QoS)
> consumer为了避免消息缓冲区越界，通过设置basic.qos方法的prefetch count参数，这个参数指定了channel上最大未确认的消息数。
达到这个上限后，mq不会继续向channel上投递消息。
由于消息投递和consumer消息确认都是异步的，因此prefetch value如果重新设置了，并且此时channel上还有正在投递的消息，
会临时出现channel上未确认的消息超过prefetch的上限。

### client端错误：重复确认和未知的tags
> 如果client确认同一个delivery tag超过一次，rabbitmq会产生channel error 例如 PRECONDITION_FAILED - unknown delivery tag 100.
还有client确认了一个未知的delivery tag，也会报这个错误。

## 发布者确认
> 通过使用事物可以确保消息不丢失，使channel为transactional，publish message，and commit.
事物比较重量级，降低了吞吐量。
为了补救这一点，引入了发布者确认机制，它模拟了consumer确认机制。
client使用confirm.select方法，依赖是否设置了no-wait，broker会回复一个confirm.select-ok。
一旦channel使用了confirm.select方法，就表示启用了confirm模式，这种模式下不能启用事物。
当channel在confirm模式下，broker和client会计数消息，第一个confirm.select从1开始。
broker处理完消息后发送basic.ack进行消息确认。delivery-tag属性包含了确认消息的序列号。
broker也会设置multiple属性在basic.ack，表示当前消息序列号之前的所有消息都被处理了。

### negative确认
> 异常情况broker不能成功处理消息，会发送basic.nack代替basic.ack。这种情况下，和basic.ack的效果是相同的，
都会忽略requeue属性。broker返回nack表示不能处理消息并且拒绝负责他们。这种情况下，client可以选择re-publish消息。
当channel在confirm模式下，所有发布的消息会被确认或者nack。不保证一个消息多久可以被确认。但是不会有同一个消息同时被确认和nack（这俩肯定是互斥的）。
basic.nack只会在Erlang进程内部错误发生时发送。
当一个消息requeue时，它会放在queue中原有的位置上。由于多个并发投递和消费，当多个消费者共享一个queue时，会出现requeue到接近队列头部的位置。

### 什么时候消息会被确认
> 对于不能路由的消息，broker会发布一个确认，一旦exchange验证一个消息不能路由到任何queue上。
如果消息强制发布，在返回basic.ack之前会先返回一个basic.return。对于nack也是这样。
对于可以路由的消息，当消息被所有队列接收后，basic.ack会发送。对于持久化消息发到持久化队列，这
意味着要持久化到磁盘。对于镜像队列，意味着所有镜像队列都接收到了这个消息。

### 持久化消息下Ack延迟
> 对于持久化的消息，在持久化到磁盘后才发送basic.ack。为了最大化减少fsync(2)调用，rabbitmq会间隔几百毫秒后批量持久化消息到磁盘。
这意味着在一个固定负载下，basic.ack的延迟会达到几百毫秒。
为了提高吞吐量，强烈建议应用异步处理确认或者批量发布消息，然后等待确认。

### 发布确认的顺序考虑
> 大多时候rabbitmq会确认消息按照他们发送的顺序。然而，发布确认异步并且能够确认一个或者一组消息。
应用不应依赖消息确认的顺序。

### 发布确认和保证投递
> broker会丢失持久化消息，如果在消息持久到磁盘之前broker崩溃。这种情况下，会导致broker以下奇怪的行为。
例如考虑这样的场景：
一个client发布一个持久化消息到持久化队列，
另个一client消费消息，但是没有发送ack确认。
broker死掉后重启。
client会重连开始消费消息。
这种情况，client会合理的认为消息会被重复投递。事实上不是这样，重启会导致broker丢失消息。
为了保证持久化，client应当使用确认。如果发布者channel在确认模式，发布者不会受到丢失消息（还未成功持久化到磁盘）的ack。

# 参考资料
[https://www.rabbitmq.com/confirms.html](https://www.rabbitmq.com/confirms.html)







---





