# 一、简介

## 1.特性

1、支持发布/订阅（Pub/Sub）和点对点（P2P）消息模型

2、在一个队列中可靠的先进先出（FIFO）和严格的顺序传递

3、支持拉（pull）和推（push）两种消息模式

4、单一队列百万消息的堆积能力

5、支持多种消息协议，如JMS、MQTT 等

6、分布式高可用的部署架构,满足至少一次消息传递语义

7、提供docker 镜像用于隔离测试和云集群部署

8、提供配置、指标和监控等功能丰富的Dashboard

## 2.相关角色

![RocketMQ 角色](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/01-0976636.png)

- 生产者（Producer）：负责产生消息，生产者向消息服务器发送由业务应用程序系统生成的消息，生产者本身既可以产生消息，如读取文本信息等。也可以对外提供接口，由外部应用来调用接口，再由生产者将收到的消息发送到MQ。

  > - 1、**获得 Topic-Broker 的映射关系**。
  >
  >   - Producer 启动时，也需要指定 Namesrv 的地址，从 Namesrv 集群中选一台建立长连接。如果该 Namesrv 宕机，会自动连其他 Namesrv ，直到有可用的 Namesrv 为止。
  >   - ==生产者每 30 秒从 Namesrv 获取 Topic 跟 Broker 的映射关系(或者说是路由规则)，更新到本地内存中。然后再跟 Topic 涉及的所有 Broker 建立长连接，每隔 30 秒发一次心跳。==
  >   - **在 Broker 端也会每 10 秒扫描一次当前注册的 Producer ，如果发现某个 Producer 超过 2 分钟都没有发心跳，则断开连接。**
  >
  > - 2、**生产者端的负载均衡**。
  >
  >   - **生产者发送时，会自动轮询当前所有可发送的broker，一条消息发送成功，下次换另外一个broker发送，以达到消息平均落到所有的broker上。**
  >
  >     > 这里需要注意一点：假如某个 Broker 宕机，意味生产者最长需要 30 秒才能感知到。在这期间会向宕机的 Broker 发送消息。当一条消息发送到某个 Broker 失败后，会自动再重发 2 次，假如还是发送失败，则抛出发送失败异常。
  >     >
  >     > **客户端里会自动轮询另外一个 Broker 重新发送，这个对于用户是透明的。**
  >
  > - 3、三种发送消息的方式：
  >
  >   - 同步方式
  >   - 异步方式
  >   - Oneway 方式：适合大数据场景，允许有一定消息丢失的场景。

- 生产者组(Producer Group)：简单来说就是多个发送同一类消息的生产者称之为一个生产者组。

- 消费者（Consumer）：负责消费消息，消费者从消息服务器拉取信息并将其输入用户应用程序。

  > - 1、获得 Topic-Broker 的映射关系。
  >
  >   - Consumer 启动时需要指定 Namesrv 地址，与其中一个 Namesrv 建立长连接。消费者每隔 30 秒从 Namesrv 获取所有Topic 的最新队列情况，这意味着某个 Broker 如果宕机，客户端最多要 30 秒才能感知。连接建立后，从 Namesrv 中获取当前消费 Topic 所涉及的 Broker，直连 Broker 。
  >   - **Consumer 跟 Broker 是长连接，会每隔 30 秒发心跳信息到Broker 。Broker 端每 10 秒检查一次当前存活的 Consumer ，若发现某个 Consumer 2 分钟内没有心跳，就断开与该 Consumer 的连接，并且向该消费组的其他实例发送通知，触发该消费者集群的负载均衡。**
  >
  > - 2、**消费者端的负载均衡**。根据消费者的消费模式不同，负载均衡方式也不同。
  >
  >   > ==消费者有两种消费模式：集群消费和广播消费。==
  >   >
  >   > - 集群消费：一个 Topic 可以由同一个消费这分组( Consumer Group )下所有消费者分担消费。
  >   >
  >   >   > 消费者的一种消费模式。一个 Consumer Group 中的各个 Consumer 实例分摊去消费消息，即一条消息只会投递到一个 Consumer Group 下面的一个实例。
  >   >   >
  >   >   > - 实际上，**每个 Consumer 是平均分摊 Message Queue 的去做拉取消费。例如某个 Topic 有 3 个队列，其中一个 Consumer Group 有 3 个实例（可能是 3 个进程，或者 3 台机器），那么每个实例只消费其中的 1 个队列。**
  >   >   > - 而由 Producer 发送消息的时候是轮询所有的队列，所以消息会平均散落在不同的队列上，可以认为队列上的消息是平均的。那么实例也就平均地消费消息了。
  >   >   > - ==这种模式下，消费进度的存储会持久化到 Broker==。
  >   >   > - 当新建一个 Consumer Group 时，默认情况下，该分组的消费者会从 min offset 开始重新消费消息。
  >   >   >
  >   >   > 具体例子：假如 TopicA 有 6 个队列，某个消费者分组起了 2 个消费者实例，那么每个消费者负责消费 3 个队列。如果再增加一个消费者分组相同消费者实例，即当前共有 3 个消费者同时消费 6 个队列，那每个消费者负责 2 个队列的消费。
  >   >
  >   > - 广播消费：每个消费者消费 Topic 下的所有队列。
  >   >
  >   >   > 消费者的一种消费模式。**消息将对一 个Consumer Group 下的各个 Consumer 实例都投递一遍。即使这些 Consumer 属于同一个Consumer Group ，消息也会被 Consumer Group 中的每个 Consumer 都消费一次。**
  >   >   >
  >   >   > - 实际上，是一个消费组下的每个消费者实例都获取到了 Topic 下面的每个 Message Queue 去拉取消费。所以消息会投递到每个消费者实例。
  >   >   > - ==这种模式下，消费进度会存储持久化到实例本地。==

- 消费者组(Consumer Group)：消费同一类消息的多个consumer 实例组成一个消费者组。

- 主题(topic)：Topic 是一种消息的逻辑分类，比如说你有订单类的消息，也有库存类的消息，那么就需要进行分类，一个是订单 Topic 存放订单相关的消息，一个是库存 Topic 存储库存相关的消息。

- 信息(Message)：Message 是消息的载体。一个 Message 必须指定 topic，相当于寄信的地址。Message 还有一个可选的 tag 设置，以便消费端可以基于 tag 进行过滤消息。也可以添加额外的键值对，例如你需要一个业务 key 来查找 broker 上的消息，方便在开发过程中诊断问题。

- 标签(tag)：标签可以被认为是对Topic 进一步细化。一般在相同业务模块中通过引入标签来标记不同用途的消息。

- 消息服务器（Broker）：==**消息存储中心，主要作用是接收来自 Producer 的消息并存储， Consumer 从这里取得消息。**==通过提供轻量级的Topic 和 Queue 机制来处理消息存储,同时支持推（push）和拉（pull）模式以及主从结构的容错机制。

  > - 1、 **高并发读写服务**。Broker的高并发读写主要是依靠以下两点：
  >
  >   - 消息顺序写，所有 Topic 数据同时只会写一个文件，一个文件满1G ，再写新文件，真正的顺序写盘，使得发消息 TPS 大幅提高。
  >   - 消息随机读，RocketMQ 尽可能让读命中系统 Pagecache ，因为操作系统访问 Pagecache 时，即使只访问 1K 的消息，系统也会提前预读出更多的数据，在下次读时就可能命中 Pagecache ，减少 IO 操作。
  >
  > - 2、 **负载均衡与动态伸缩**。
  >
  >   - 负载均衡：==Broker 上存 Topic 信息，Topic 由多个队列组成，队列会平均分散在多个 Broker 上，而 Producer 的发送机制保证消息尽量平均分布到所有队列中，最终效果就是所有消息都平均落在每个 Broker 上。==
  >
  >   - 动态伸缩能力（非顺序消息）：Broker 的伸缩性体现在两个维度：Topic、Broker。
  >
  >     - Topic 维度：假如一个 Topic 的消息量特别大，但集群水位压力还是很低，就可以扩大该 Topic 的队列数， Topic 的队列数跟发送、消费速度成正比。
  >
  >       > Topic 的队列数一旦扩大，就无法很方便的缩小。因为，生产者和消费者都是基于相同的队列数来处理。
  >       > 如果真的想要缩小，只能新建一个 Topic ，然后使用它。不过，Topic 的队列数，也不存在什么影响的。
  >
  >     - Broker 维度：如果集群水位很高了，需要扩容，直接加机器部署 Broker 就可以。Broker 启动后向 Namesrv 注册，Producer、Consumer 通过 Namesrv 发现新Broker，立即跟该 Broker 直连，收发消息。
  >
  >       > 新增的 Broker 想要下线，想要下线也比较麻烦，暂时没特别好的方案。大体的前提是，消费者消费完该 Broker 的消息，生产者不往这个 Broker 发送消息。
  >
  > - 3、 **高可用 & 高可靠**。
  >
  >   - 高可用：集群部署时一般都为主备，备机实时从主机同步消息，如果其中一个主机宕机，备机提供消费服务，但不提供写服务。
  >
  >   - 高可靠：所有发往 Broker 的消息，有同步刷盘和异步刷盘机制。
  >
  >     - 同步刷盘时，消息写入物理文件才会返回成功。
  >
  >     - 异步刷盘时，只有机器宕机，才会产生消息丢失，Broker 挂掉可能会发生，但是机器宕机崩溃是很少发生的，除非突然断电。
  >
  >       > 如果 Broker 挂掉，未同步到硬盘的消息，还在 Pagecache 中呆着。
  >
  > - 4、 **Broker 与 Namesrv 的心跳机制**。
  >
  >   - ==单个 Broker 跟所有 Namesrv 保持心跳请求，心跳间隔为30秒，心跳请求中包括当前 Broker 所有的 Topic 信息==。
  >   - Namesrv 会反查 Broker 的心跳信息，如果某个 Broker 在 2 分钟之内都没有心跳，则认为该 Broker 下线，调整 Topic 跟 Broker 的对应关系。但此时 Namesrv 不会主动通知Producer、Consumer 有 Broker 宕机。也就说，只能等 Producer、Consumer 下次定时拉取 Topic 信息的时候，才会发现有 Broker 宕机。
  >
  >  Broker 是 RocketMQ 中最复杂的角色，主要包括如下五个模块：
  >
  > ![img](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/006tKfTcgy1fo4vbpoxtej30r30dsdgs.jpg)
  >
  > - 远程处理模块：是 Broker 的入口，处理来自客户的请求。
  > - Client Manager ：管理客户端（生产者/消费者），并维护消费者的主题订阅。
  > - Store Service ：提供简单的 API 来存储或查询物理磁盘中的消息。
  > - HA 服务：提供主节点和从节点之间的数据同步功能。
  > - 索引服务：通过指定键为消息建立索引，并提供快速的消息查询。

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

## 3.RocketMQ 的整体流程

![整体流程](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/02-0976659.png)

- 1、启动 **Namesrv**，Namesrv起 来后监听端口，等待 Broker、Producer、Consumer 连上来，相当于一个路由控制中心。
- 2、**Broker** 启动，跟所有的 Namesrv 保持长连接，定时发送心跳包。

> 心跳包中，包含当前 Broker 信息(IP+端口等)以及存储所有 Topic 信息。注册成功后，Namesrv 集群中就有 Topic 跟 Broker 的映射关系。

- 3、收发消息前，先创建 Topic 。创建 Topic 时，需要指定该 Topic 要存储在 哪些 Broker上。也可以在发送消息时自动创建Topic。

- 4、**Producer** 发送消息。

> 启动时，先跟 Namesrv 集群中的其中一台建立长连接，并从Namesrv 中获取当前发送的 Topic 存在哪些 Broker 上，然后跟对应的 Broker 建立长连接，直接向 Broker 发消息。

- 5、**Consumer** 消费消息。

> Consumer 跟 Producer 类似。跟其中一台 Namesrv 建立长连接，获取当前订阅 Topic 存在哪些 Broker 上，然后直接跟 Broker 建立连接通道，开始消费消息。





















参照：http://svip.iocoder.cn/RocketMQ/Interview/