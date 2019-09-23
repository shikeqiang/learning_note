# 一、简介

​	RabbitMQ是一个开源的消息代理和队列服务器，用来通过普通协议在完全不同的应用之间传递数据，RabbitMQ是使用Erlang语言来编写的，并且RabbitMQ是基于AMQP协议的。

## 特点：

1. RabbitMQ底层使用Erlang语言编写，传递效率高，延迟低
2. 开源、性能优秀、稳定性较高
3. 与SpringAMQP完美的整合、API丰富
4. 集群模式丰富、表达式配置、HA模式、镜像队列模式
5. 保证数据不丢失的情况下，做到高可用
6. AMQP全称：Advanced Message Queuing Protocol(Spring为RabbitMQ设计的一套框架)
7. AMQP翻译:高级消息队列协议

## RabbitMQ整体架构

![img_0918](/Users/jack/Desktop/md/images/img_0918.png)

​	先从客户端生产者发送消息到exchange上，然后再路由到对应的queue中，消费者只要监听对应的queue即可接收到消息。

## AMQP核心概念

- Server：又称Broker，接收客户端的连接，实现AMQP实体服务

- Connection：连接，应用程序与Broker的网络连接

- Channel：网络信道，**几乎所有的操作都在Channel中进行，包括定义Queue、定义Exchange、绑定Queue与Exchange、发布消息等。**Channel是进行消息读写的通道。客户端可以建立多个Channel，每个Channel代表一个会话任务。

- Message：消息，服务器和应用程序之间传送的数据，由Properties和Body组成。

  > Properties可以对消息进行修饰，比如消息的优先级、延迟等高级特性；Body就是消息体内容。

- Virtual host：**虚拟地址，用于进行逻辑隔离，最上层的消息路由**。一个Virtual host可以有若干个Exchange和Queue，==同一个Virtual host里面不能有相同名称的Exchange和Queue。==

- Exchange：交换机，接收消息，根据路由键转发消息到绑定的队列。

> RabbitMQ中有三种常用的交换机类型:
>
> - direct: 如果路由键匹配，消息就投递到对应的队列
> - fanout：投递消息给所有绑定在当前交换机上面的队列
> - topic：允许实现有趣的消息通信场景，使得5不同源头的消息能够达到同一个队列。topic队列名称有两个特殊的关键字。

- Binding：Exchange和Queue之间的虚拟连接，binding中可以包含routing key。

- Routing key：一个路由规则，虚拟机可用它来确定如何路由一个特定消息

  > \* 可以替换一个单词
  > \# 可以替换多个单词

- Queue：也称为Message Queue，消息队列，保存消息并将它们转发给消费者，多个消费者可以订阅同一个Queue，这时Queue中的消息会被平均分摊给多个消费者进行处理，而不是每个消费者都收到所有的消息并处理。

- Prefetch count：如果有多个消费者同时订阅同一个Queue中的消息，Queue中的消息会被平摊给多个消费者。这时如果每个消息的处理时间不同，就有可能会导致某些消费者一直在忙，而另外一些消费者很快就处理完手头工作并一直空闲的情况。我们可以通过设置prefetchCount来限制Queue每次发送给每个消费者的消息数，比如我们设置prefetchCount=1，则Queue每次给每个消费者发送一条消息；消费者处理完这条消息后Queue会再给该消费者发送一条消息。

## 消息流转图

![image](/Users/jack/Desktop/md/images/RabbitMQ1.jpg)





































参照：https://github.com/suxiongwei/springboot-rabbitmq

[RabbitMQ消息中间件极速入门与实战](https://www.imooc.com/learn/1042)