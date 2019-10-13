# 一、简介

## 1.相关角色

![RocketMQ 角色](/Users/jack/Desktop/md/images/01-0976636.png)

- 生产者（Producer）：负责产生消息，生产者向消息服务器发送由业务应用程序系统生成的消息。

- 消费者（Consumer）：负责消费消息，消费者从消息服务器拉取信息并将其输入用户应用程序。

- 消息服务器（Broker）：**消息存储中心，主要作用是接收来自 Producer 的消息并存储， Consumer 从这里取得消息。**

- 名称服务器（NameServer）：用来保存 Broker 相关 Topic 等元信息并给 Producer ，提供 Consumer 查找 Broker 信息。

  > - Namesrv 用于存储 Topic、Broker 关系信息，功能简单，稳定性高。
  >
  >   - 多个 Namesrv 之间相互没有通信，单台 Namesrv 宕机不影响其它 Namesrv 与集群。
  >
  >     > 多个 Namesrv 之间的信息共享，**通过 Broker 主动向多个 Namesrv 都发起心跳**，Broker 需要跟所有 Namesrv 连接。
  >
  >   - 即使整个 Namesrv 集群宕机，已经正常工作的 Producer、Consumer、Broker 仍然能正常工作，但新起的 Producer、Consumer、Broker 就无法工作。
  >
  >     > 这点和 Dubbo 有些不同，不会缓存 Topic 等元信息到本地文件。
  >
  > - Namesrv 压力不会太大，平时主要开销是在维持心跳和提供 Topic-Broker 的关系数据。但有一点需要注意，**Broker 向 Namesr 发心跳时，会带上当前自己所负责的所有 Topic 信息，如果 Topic 个数太多（万级别），会导致一次心跳中，就 Topic 的数据就几十 M，网络情况差的话，网络传输失败，心跳失败，导致 Namesrv 误认为 Broker 心跳失败。**
  >
  >   > 当然，一般公司，很难达到过万级的 Topic ，因为一方面体量达不到，另一方面 RocketMQ 提供了 Tag 属性。
  >   >
  >   > 另外，内网环境网络相对是比较稳定的，传输几十 M 问题不大。同时，如果真的要优化，**Broker 可以把心跳包做压缩，再发送给 Namesrv 。不过，这样也会带来 CPU 的占用率的提升。**
  >
  > - ==将 Namesrv 地址列表提供给客户端( 生产者和消费者 )，==有四种方法：
  >
  >   - 编程方式，就像 `producer.setNamesrvAddr("ip:port")` 。
  >   - Java 启动参数设置，使用 `rocketmq.namesrv.addr` 。
  >   - 环境变量，使用 `NAMESRV_ADDR` ，可以通过修改操作系统的配置文件实现。
  >   - HTTP 端点，例如说：`http://namesrv.rocketmq.xxx.com` 地址，通过 DNS 解析获得 Namesrv 真正的地址。

## 2.RocketMQ 的整体流程

![整体流程](/Users/jack/Desktop/md/images/02-0976659.png)

- 1、启动 **Namesrv**，Namesrv起 来后监听端口，等待 Broker、Producer、Consumer 连上来，相当于一个路由控制中心。
- 2、**Broker** 启动，跟所有的 Namesrv 保持长连接，定时发送心跳包。

> 心跳包中，包含当前 Broker 信息(IP+端口等)以及存储所有 Topic 信息。注册成功后，Namesrv 集群中就有 Topic 跟 Broker 的映射关系。

- 3、收发消息前，先创建 Topic 。创建 Topic 时，需要指定该 Topic 要存储在 哪些 Broker上。也可以在发送消息时自动创建Topic。

- 4、**Producer** 发送消息。

> 启动时，先跟 Namesrv 集群中的其中一台建立长连接，并从Namesrv 中获取当前发送的 Topic 存在哪些 Broker 上，然后跟对应的 Broker 建立长连接，直接向 Broker 发消息。

- 5、**Consumer** 消费消息。

> Consumer 跟 Producer 类似。跟其中一台 Namesrv 建立长连接，获取当前订阅 Topic 存在哪些 Broker 上，然后直接跟 Broker 建立连接通道，开始消费消息。





















参照：http://svip.iocoder.cn/RocketMQ/Interview/