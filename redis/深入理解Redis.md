# 一、Redis 的线程模型

## 1.原理

​	Redis 内部使用文件事件处理器 `file event handler`，这个**文件事件处理器是单线程的，所以 Redis 才叫做单线程的模型**。它采用 **IO 多路复用机制同时监听多个 socket**，根据 socket 上的事件来选择对应的事件处理器进行处理。

文件事件处理器的结构包含 4 个部分：

- 多个 socket
- IO 多路复用程序
- 文件事件分派器
- 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）

> 多个 socket 可能会并发产生不同的操作，每个操作对应不同的文件事件，但是 IO 多路复用程序会监听多个 socket，会将 socket 产生的事件放入队列中排队，事件分派器每次从队列中取出一个事件，把该事件交给对应的事件处理器进行处理。

来看客户端与 Redis 的一次通信过程：

![Redis-single-thread-model](/Users/jack/Desktop/md/images/01-0249933.png)

- 客户端 socket01 向 Redis 的 server socket 请求建立连接，此时 server socket 会产生一个 `AE_READABLE` 事件，**IO 多路复用程序监听到 server socket 产生的事件后，将该事件压入队列中。文件事件分派器从队列中获取该事件，交给`连接应答处理器`。**==连接应答处理器会创建一个能与客户端通信的 socket01，并将该 socket01 的 `AE_READABLE` 事件与命令请求处理器关联。==
- 假设此时客户端发送了一个 `set key value` 请求，此时 Redis 中的 **socket01** 会产生 `AE_READABLE` 事件，IO 多路复用程序将事件压入队列，此时事件分派器从队列中获取到该事件，由于前面 socket01 的 `AE_READABLE` 事件已经与命令请求处理器关联，因此**事件分派器将事件交给命令请求处理器来处理**。命令请求处理器读取 socket01 的 `key value` 并在自己内存中完成 `key value` 的设置。操作完成后，它会将 socket01 的 `AE_WRITABLE` 事件与令回复处理器关联。
- 如果此时客户端准备好接收返回结果了，那么 Redis 中的 socket01 会产生一个 `AE_WRITABLE` 事件，同样压入队列中，事件分派器找到相关联的命令回复处理器，**由命令回复处理器对 socket01 输入本次操作的一个结果，比如 `ok`，之后解除 socket01 的 `AE_WRITABLE` 事件与命令回复处理器的关联。**

## 2.效率高的原因

1、纯内存操作。

> Redis 为了达到最快的读写速度，将数据都读到内存中，并通过异步的方式将数据写入磁盘。所以 Redis 具有快速和数据持久化的特征。
>
> 如果不将数据放在内存中，磁盘 I/O 速度为严重影响 Redis 的性能。

**2、核心是基于非阻塞的 IO 多路复用机制。**

3、单线程反而避免了多线程的频繁上下文切换问题。

> Redis 利用队列技术，将并发访问变为串行访问，消除了传统数据库串行控制的开销

4、Redis 全程使用 hash 结构，读取速度快，还有一些特殊的数据结构，对数据存储进行了优化，如压缩表，对短数据进行压缩存储，再如，跳表，使用有序的数据结构加快读取的速度。

​	可以在同一个服务器部署多个 Redis 的实例，并把他们当作不同的服务器来使用，在某些时候，无论如何一个服务器是不够的， 所以，如果想使用多个 CPU ，可以考虑一下分区。

# 二、Redis持久化

Redis 提供了两种方式，实现数据的持久化到硬盘。

- 1、【全量】RDB 持久化，是指在指定的时间间隔内将内存中的**数据集快照**写入磁盘。实际操作过程是，fork 一个子进程，先将数据集写入临时文件，写入成功后，再替换之前的文件，用二进制压缩存储。
- 2、【增量】AOF持久化，以日志的形式记录服务器所处理的每一个**写、删除操作**，查询操作不会记录，以文本的方式记录，可以打开文件看到详细的操作记录。

## **RDB 优缺点**

① 优点

- **灵活设置备份频率和周期。**你可能打算每个小时归档一次最近 24 小时的数据，同时还要每天归档一次最近 30 天的数据。通过这样的备份策略，一旦系统出现灾难性故障，我们可以非常容易的进行恢复。

- **非常适合冷备份，对于灾难恢复而言，RDB 是非常不错的选择。**因为我们可以非常轻松的将一个单独的文件压缩后再转移到其它存储介质上。推荐，可以将这种完整的数据文件发送到一些远程的安全存储上去，比如说 Amazon 的 S3 云服务上去，在国内可以是阿里云的 OSS 分布式存储上。

- **性能最大化。**对于 Redis 的服务进程而言，在开始持久化时，它唯一需要做的只是 fork 出子进程，之后再由子进程完成这些持久化的工作，这样就可以极大的避免服务进程执行 IO 操作了。也就是说，RDB 对 Redis 对外提供的读写服务，影响非常小，可以让 Redis 保持高性能。

  > 一个进程，包括代码、数据和分配给进程的资源。fork（）函数通过系统调用创建一个与原来进程几乎完全相同的进程，也就是两个进程可以做完全相同的事，但如果初始参数或者传入的变量不同，两个进程也可以做不同的事。
  > 一个进程调用fork（）函数后，系统先给新的进程分配资源，例如存储数据和代码的空间。然后把原来的进程的所有值都复制到新的新进程中，只有少数值与原来的进程的值不同。相当于克隆了一个自己。

- **恢复更快。相比于 AOF 机制，RDB 的恢复速度更更快，更适合恢复数据，特别是在数据集非常大的情况。**

② 缺点

- 如果你想保证数据的高可用性，即最大限度的避免数据丢失，那么 RDB 将不是一个很好的选择。因为系统一旦在定时持久化之前出现宕机现象，此前没有来得及写入磁盘的数据都将丢失。

  > 所以，RDB 实际场景下，需要和 AOF 一起使用。

- 由于 RDB 是通过 fork 子进程来协助完成数据持久化工作的，因此，如果当数据集较大时，可能会导致整个服务器停止服务几百毫秒，甚至是 1 秒钟。

  > 所以，RDB 建议在业务低估，例如在半夜执行。

## **AOF 优缺点**

① 优点

- 该机制可以带来更高的数据安全性，即数据持久性。**Redis 中提供了 3 种同步策略，即每秒同步、每修改(执行一个命令)同步和不同步。**
  - 事实上，每秒同步也是异步完成的，其效率也是非常高的，所差的是一旦系统出现宕机现象，那么这一秒钟之内修改的数据将会丢失。
  - 每修改同步，我们可以将其视为同步持久化，即每次发生的数据变化都会被立即记录到磁盘中。可以预见，这种方式在效率上是最低的。
  - 不同步，则每次发生的数据变化不会同时记录到磁盘。
- 由于该机制对日志文件的写入操作采用的是append模式，因此在写入过程中即使出现宕机现象，也不会破坏日志文件中已经存在的内容。
  - 因为以 append-only 模式写入，所以没有任何磁盘寻址的开销，写入性能非常高。
  - 另外，如果我们本次操作只是写入了一半数据就出现了系统崩溃问题，不用担心，在 Redis 下一次启动之前，我们可以通过 Redis-check-aof 工具来帮助我们解决数据一致性的问题。
- ==如果日志过大，Redis可以自动启用 **rewrite** 机制。==即使出现后台重写操作，也不会影响客户端的读写。因为在 rewrite log 的时候，会对其中的指令进行压缩，创建出一份需要恢复数据的最小日志出来。再创建新日志文件的时候，老的日志文件还是照常写入。当新的 merge 后的日志文件 ready 的时候，再交换新老日志文件即可。
- AOF 包含一个格式清晰、易于理解的日志文件用于记录所有的**修改操作**。事实上，我们也可以通过该文件完成数据的重建。

② 缺点

- **对于相同数量的数据集而言，AOF 文件通常要大于 RDB 文件。RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。**
- 根据同步策略的不同，AOF 在运行效率上往往会慢于 RDB 。总之，每秒同步策略的效率是比较高的，同步禁用策略的效率和 RDB 一样高效。
- 以前 AOF 发生过 bug ，就是通过 AOF 记录的日志，进行数据恢复的时候，没有恢复一模一样的数据出来。所以说，类似 AOF 这种较为复杂的基于命令日志/merge/回放的方式，比基于 RDB 每次持久化一份完整的数据快照文件的方式，更加脆弱一些，容易有 bug 。不过 AOF 就是为了避免 rewrite 过程导致的 bug ，因此每次 rewrite 并不是基于旧的指令日志进行 merge 的，而是基于当时内存中的数据进行指令的重新构建，这样健壮性会好很多。

**如何选择**

- 不要仅仅使用 RDB，因为那样会导致你丢失很多数据

- 也不要仅仅使用 AOF，因为那样有两个问题，第一，你通过 AOF 做冷备，没有 RDB 做冷备，来的恢复速度更快; 第二，RDB 每次简单粗暴生成数据快照，更加健壮，可以避免 AOF 这种复杂的备份和恢复机制的 bug 。

- Redis 支持同时开启开启两种持久化方式，我们可以综合使用 AOF 和 RDB 两种持久化机制，用 AOF 来保证数据不丢失，作为数据恢复的第一选择; 用 RDB 来做不同程度的冷备，在 AOF 文件都丢失或损坏不可用的时候，还可以使用 RDB 来进行快速的数据恢复。

  - 如果同时使用 RDB 和 AOF 两种持久化机制，那么在 Redis 重启的时候，会使用 **AOF** 来重新构建数据，因为 AOF 中的**数据更加完整**。

    > 一般来说， 如果想达到足以媲美 PostgreSQL 的数据安全性， 你应该同时使用两种持久化功能。如果你非常关心你的数据， 但仍然可以承受数分钟以内的数据丢失，那么你可以只使用 RDB 持久化。
    >
    > 有很多用户都只使用 AOF 持久化，但并不推荐这种方式：因为定时生成 RDB 快照（snapshot）非常便于进行数据库备份， 并且 RDB 恢复数据集的速度也要比AOF恢复的速度要快，除此之外，使用 RDB 还可以避免之前提到的 AOF 程序的 bug。

在 Redis4.0 版本开始，允许你使用 RDB-AOF 混合持久化方式，详细可见 [《Redis4.0 之 RDB-AOF 混合持久化》](https://yq.aliyun.com/articles/193034) 。也因此，RDB 和 AOF 同时使用，是希望达到安全的持久化的推荐方式。

> - bgsave 做镜像全量持久化，AOF 做增量持久化。因为 bgsave 会耗费较长时间，不够实时，在停机的时候会导致大量丢失数据，所以需要 AOF 来配合使用。在 Redis 实例重启时，会使用 bgsave 持久化文件重新构建内存，再使用 AOF 重放近期的操作指令来实现完整恢复重启之前的状态。
> - 对方追问那如果突然机器掉电会怎样？取决于 AOF 日志 sync 属性的配置，如果不要求性能，在每条写指令时都 sync 一下磁盘，就不会丢失数据。但是在高性能的要求下每次都 sync 是不现实的，一般都使用定时 sync ，比如 1 秒 1 次，这个时候最多就会丢失 1 秒的数据。
> - 对方追问 bgsave 的原理是什么？你给出两个词汇就可以了，fork 和 cow 。fork 是指 Redis 通过创建子进程来进行 bgsave 操作。cow 指的是 copy on write ，子进程创建后，父子进程共享数据段，父进程继续提供读写服务，写脏的页面数据会逐渐和子进程分离开来。这里 bgsave 操作后，会产生 RDB 快照文件。

为什么不建议在主 Redis 节点开启 RDB 功能呢？因为会带来一定时间的阻塞，特别是数据量大的时候。

> - 子进程/fork相关的阻塞：在bgsave的时候，Redis主进程会fork一个子进程，利用操作系统的写时复制技术，这个子进程在拷贝父进程的时候理论上是很快的，因为并不需要全拷贝，比如主进程虽然占了10g内存，但子进程拷贝他可能只要200毫秒，也就阻塞了200毫秒(此耗时基本跟主进程占用的内存是成正比的)，这个具体的时间可以通过统计项info stats 里的last_fork_usec查看。
> - cpu/单线程相关的阻塞：**Redis主进程是单线程跑在单核cpu上的，如果显示绑定了CPU，则子进程会与主进程共享一个CPU，而子进程进行持久化的时候是非常占CPU（强势90%），因此这种情况也可能导致提供服务的主进程发生阻塞（因此如果需要持久化功能，不建议绑定CPU）；**
> - 内存相关的阻塞：**虽然利用写时复制技术可以大大降低进程拷贝的内存消耗，但这也导致了父进程在处理写请求时需要维护修改的内存页，因此这部分内存过大的话（修改页数多或每页占空间大）也会导致父进程的写操作阻塞。**（而不巧的是，Linux中TransparentHugePage会将复制内存页面单位有4K变成2M，这对于Redis来说是比较不友好的，也是建议优化的，具体可度之）
> - 磁盘相关的阻塞：极端情况下，假设整个机器的内存已经所剩无几，触发了内存交换（SWAP），则整个Redis的效率将会非常低下（显然这不仅仅针对save/bgsave），因此，关注系统的io情况，也是定位阻塞问题的一种方法。

# 三、Redis的“过期”和“淘汰”策略

## 1.“过期”策略

Redis 的过期策略，就是指当 Redis 中缓存的 key 过期了，Redis 如何处理。

Redis 提供了 3 种数据过期策略：

- 被动删除：当读/写一个已经过期的 key 时，会触发惰性删除策略，直接删除掉这个过期 key 。

  > 只有key被操作时(如GET)，REDIS才会被动检查该key是否过期，如果过期则删除之并且返回NIL。
  >
  > 1、这种删除策略对CPU是友好的，删除操作只有在不得不的情况下才会进行，不会其他的expire key上浪费无谓的CPU时间。
  >
  > 2、但是这种策略对内存不友好，一个key已经过期，但是在它被操作之前不会被删除，仍然占据内存空间。如果有大量的过期键存在但是又很少被访问到，那会造成大量的内存空间浪费。expireIfNeeded(redisDb *db, robj *key)函数位于src/db.c。
  >
  > 但仅是这样是不够的，**因为可能存在一些key永远不会被再次访问到，这些设置了过期时间的key也是需要在过期后被删除的，我们甚至可以将这种情况看作是一种内存泄露----无用的垃圾数据占用了大量的内存，而服务器却不会自己去释放它们，这对于运行状态非常依赖于内存的Redis服务器来说，肯定不是一个好消息**

- 主动删除：由于惰性删除策略无法保证冷数据被及时删掉，所以 Redis 会定期主动淘汰一批已过期的 key 。

  > 先说一下时间事件，对于持续运行的服务器来说， 服务器需要定期对自身的资源和状态进行必要的检查和整理， 从而让服务器维持在一个健康稳定的状态， 这类操作被统称为常规操作（cron job）。
  >
  > 在 Redis 中， 常规操作由 `redis.c/serverCron` 实现， 它主要执行以下操作：
  >
  > - 更新服务器的各类统计信息，比如时间、内存占用、数据库占用情况等。
  > - 清理数据库中的过期键值对。
  > - 对不合理的数据库进行大小调整。
  > - 关闭和清理连接失效的客户端。
  > - 尝试进行 AOF 或 RDB 持久化操作。
  > - 如果服务器是主节点的话，对附属节点进行定期同步。
  > - 如果处于集群模式的话，对集群进行定期同步和连接测试。
  >
  > Redis 将 `serverCron` 作为时间事件来运行， 从而确保它每隔一段时间就会自动运行一次， 又因为 `serverCron` 需要在 Redis 服务器运行期间一直定期运行， 所以它是一个循环时间事件： `serverCron` 会一直定期执行，直到服务器关闭为止。
  >
  > 在 Redis 2.6 版本中， 程序规定 `serverCron` 每秒运行 `10` 次， 平均每 `100` 毫秒运行一次。 从 Redis 2.8 开始， 用户可以通过修改 `hz`选项来调整 `serverCron` 的每秒执行次数， 具体信息请参考 `redis.conf` 文件中关于 `hz` 选项的说明。

- maxmemory：当前已用内存超过 maxmemory 限定时，触发主动清理策略，即 [「数据“淘汰”策略」](http://svip.iocoder.cn/Redis/Interview/#) 。

  > 当前已用内存超过maxmemory限定时，触发**主动清理**策略：
  >
  > - volatile-lru：只对设置了过期时间的key进行LRU（默认值）
  > - allkeys-lru ： 删除lru算法的key
  > - volatile-random：随机删除即将过期key
  > - allkeys-random：随机删除
  > - volatile-ttl ： 删除即将过期的
  > - noeviction ： 永不过期，返回错误当mem_used内存已经超过maxmemory的设定，对于所有的读写请求，都会触发redis.c/freeMemoryIfNeeded(void)函数以清理超出的内存。注意这个清理过程是阻塞的，直到清理出足够的内存空间。所以如果在达到maxmemory并且调用方还在不断写入的情况下，可能会反复触发主动清理策略，导致请求会有一定的延迟。 
  >
  > 当mem_used内存已经超过maxmemory的设定，对于所有的读写请求，都会触发redis.c/freeMemoryIfNeeded(void)函数以清理超出的内存。注意这个清理过程是阻塞的，直到清理出足够的内存空间。所以如果在达到maxmemory并且调用方还在不断写入的情况下，可能会反复触发主动清理策略，导致请求会有一定的延迟。
  >
  > 清理时会根据用户配置的maxmemory-policy来做适当的清理（一般是LRU或TTL），这里的LRU或TTL策略并不是针对redis的所有key，而是以配置文件中的maxmemory-samples个key作为样本池进行抽样清理。
  >
  > maxmemory-samples在redis-3.0.0中的默认配置为5，如果增加，会提高LRU或TTL的精准度，redis作者测试的结果是当这个配置为10时已经非常接近全量LRU的精准度了，并且增加maxmemory-samples会导致在主动清理时消耗更多的CPU时间，建议：
  >
  > - 尽量不要触发maxmemory，最好在mem_used内存占用达到maxmemory的一定比例后，需要考虑调大hz以加快淘汰，或者进行集群扩容。
  > - 如果能够控制住内存，则可以不用修改maxmemory-samples配置；如果Redis本身就作为LRU cache服务（这种服务一般长时间处于maxmemory状态，由Redis自动做LRU淘汰），可以适当调大maxmemory-samples。

在 Redis 中，同时使用了上述 3 种策略，即它们**非互斥**的。

参照： [《关于 Redis 数据过期策略》](https://www.cnblogs.com/chenpingzhao/p/5022467.html)

## 2.“淘汰”策略

Redis 内存数据集大小上升到一定大小的时候，就会进行数据淘汰策略。

### 数据淘汰策略：

> **volatile-lru策略和volatile-random策略适合我们将一个Redis实例既应用于缓存和又应用于持久化存储的时候，然而我们也可以通过使用两个Redis实例来达到相同的效果，值得一提的是将key设置过期时间实际上会消耗更多的内存，因此我们建议使用allkeys-lru策略从而更有效率的使用内存。**

1. volatile-lru

   > 从已设置过期时间的数据集中挑选**最近最少使用的数据淘汰**。redis并不是保证取得所有数据集中最近最少使用的键值对，而只是随机挑选的几个键值对中的， 当内存达到限制的时候无法写入非过期时间的数据集。
   >
   > 

2. volatile-ttl

   > 从已设置过期时间的数据集中挑选**将要过期的数据淘汰**。redis 并不是保证取得所有数据集中最近将要过期的键值对，而只是随机挑选的几个键值对中的， 当内存达到限制的时候无法写入非过期时间的数据集。
   >
   > 这种策略使得我们可以向Redis提示哪些key更适合被eviction。

3. volatile-random

   > 从已设置过期时间的数据集中**任意选择数据淘汰**。当内存达到限制的时候无法写入非过期时间的数据集。

4. allkeys-lru

   > **从数据集中挑选最近最少使用的数据淘汰。**当内存达到限制的时候，对所有数据集挑选最近最少使用的数据淘汰，可写入新的数据集。
   >
   > ==如果我们的应用对缓存的访问符合幂律分布，也就是存在相对热点数据，或者我们不太清楚我们应用的缓存访问分布状况，我们可以选择allkeys-lru策略。==可保证热点数据不要被淘汰

5. allkeys-random

   > **从数据集中任意选择数据淘汰**，当内存达到限制的时候，对所有数据集挑选随机淘汰，可写入新的数据集。
   >
   > **如果我们的应用对于缓存key的访问概率相等，则可以使用这个策略。**

6. no-enviction

   > 当内存达到限制的时候，不淘汰任何数据，不可写入任何数据集，所有引起申请内存的命令会报错。

### 配置：

通过配置redis.conf中的maxmemory这个值来开启内存淘汰功能：# maxmemory ；值得注意的是，maxmemory为0的时候表示我们对Redis的内存使用没有限制。根据应用场景，选择淘汰策略：# maxmemory-policy noeviction。

### 内存淘汰的过程：

> 首先，客户端发起了需要申请更多内存的命令（如set）。
>
> 然后，Redis检查内存使用情况，如果已使用的内存大于maxmemory则开始根据用户配置的不同淘汰策略来淘汰内存（key），从而换取一定的内存。
>
> 最后，如果上面都没问题，则这个命令执行成功。

### 动态改配置命令

此外，redis支持动态改配置，无需重启。

设置最大内存

```
config set maxmemory 100000
```

设置淘汰策略

```
config set maxmemory-policy noeviction
```

参照： [《Redis实战（二） 内存淘汰机制》](http://blog.720ui.com/2016/redis_action_02_maxmemory_policy/)

# 四、Redis数据结构

- 字符串 String
- 字典Hash
- 列表List
- 集合Set
- 有序集合 SortedSet

- HyperLogLog
- Geo
- Pub / Sub

- BloomFilter
- RedisSearch
- Redis-ML
- JSON

另外，在 Redis 5.0 增加了 Stream 功能，一个新的强大的支持多播的可持久化的消息队列，提供类似 Kafka 的功能。

一个 Redis 实例，最多能存放多少的 keys ，List、Set、Sorted Set 他们最多能存放多少元素。

理论上，Redis 可以处理多达 2^32 的 keys ，并且在实际中进行了测试，每个实例至少存放了 2 亿 5 千万的 keys。

任何 list、set、和 sorted set 都可以放 2^32 个元素。

# 五、Redis使用场景

## 缓存

对于热点数据，缓存以后可能读取数十万次，因此，对于热点数据，缓存的价值非常大。例如，分类栏目更新频率不高，但是绝大多数的页面都需要访问这个数据，因此读取频率相当高，可以考虑基于 Redis 实现缓存。

## 会话缓存

此外，还可以考虑使用 Redis 进行会话缓存。例如，将 web session 存放在 Redis 中。

## 时效性

例如验证码只有60秒有效期，超过时间无法使用，或者基于 Oauth2 的 Token 只能在 5 分钟内使用一次，超过时间也无法使用。

## 访问频率

出于减轻服务器的压力或防止恶意的洪水攻击的考虑，需要控制访问频率，例如限制 IP 在一段时间的最大访问量。

## 计数器

数据统计的需求非常普遍，通过原子递增保持计数。例如，应用数、资源数、点赞数、收藏数、分享数等。

## 社交列表

社交属性相关的列表信息，例如，用户点赞列表、用户分享列表、用户收藏列表、用户关注列表、用户粉丝列表等，使用 Hash 类型数据结构是个不错的选择。

## 记录用户判定信息

记录用户判定信息的需求也非常普遍，可以知道一个用户是否进行了某个操作。例如，用户是否点赞、用户是否收藏、用户是否分享等。

## 交集、并集和差集

在某些场景中，例如社交场景，通过交集、并集和差集运算，可以非常方便地实现共同好友，共同关注，共同偏好等社交关系。

## 热门列表与排行榜

按照得分进行排序，例如，展示最热、点击率最高、活跃度最高等条件的排名列表。

## 最新动态

按照时间顺序排列的最新动态，也是一个很好的应用，可以使用 Sorted Set 类型的分数权重存储 Unix 时间戳进行排序。

## 消息队列

Redis 能作为一个很好的消息队列来使用，依赖 List 类型利用 LPUSH 命令将数据添加到链表头部，通过 BRPOP 命令将元素从链表尾部取出。同时，市面上成熟的消息队列产品有很多，例如 RabbitMQ。因此，更加建议使用 RabbitMQ 作为消息中间件。

## 秒杀

- 提前预热数据，放入Redis
- 商品列表放入Redis List
- 商品的详情数据 Redis hash保存，设置过期时间
- 商品的库存数据Redis sorted set保存
- 用户的地址信息Redis set保存
- 订单产生扣库存通过Redis制造分布式锁，库存同步扣除
- 订单产生后发货的数据，产生Redis list，通过消息队列处理
- 秒杀结束后，再把Redis数据和数据库进行同步
- 可能还会引入http缓存，或者将消息对接用MQ代替等方案

> **1、显示最新的项目列表**
>  下面这个语句常用来显示最新项目，随着数据多了，查询毫无疑问会越来越慢。
>
> ```
> SELECT * FROM foo WHERE … ORDER BY time DESC LIMIT 10
> ```
>
> 在Web应用中，“列出最新的回复”之类的查询非常普遍，这通常会带来可扩展性问题。
> 比如说，我们的一个Web应用想要列出用户贴出的最新20条评论。在最新的评论边上我们有一个“显示全部”的链接，点击后就可以获得更多的评论。
>  我们假设数据库中的每条评论都有一个唯一的递增的ID字段，可以使用分页来制作主页和评论页，使用Redis的模板，每次新评论发表时，我们会将它的ID添加到一个Redis列表：`LPUSH latest.comments`
>  我们将列表裁剪为指定长度，因此Redis只需要保存最新的5000条评论： `LTRIM latest.comments 0 5000`
>  每次我们需要获取最新评论的项目范围时，我们调用一个函数来完成(使用伪代码)：
>
> ```lua
> FUNCTION get_latest_comments(start, num_items):
> id_list = redis.lrange(“latest.comments”,start,start+num_items – 1)
> IF id_list.length < num_items
> id_list = SQL_DB(“SELECT … ORDER BY time LIMIT …”)
> END
> RETURN id_list
> END
> ```
>
> 这里我们做的很简单。在Redis中我们的最新ID使用了常驻缓存，这是一直更新的。但是我们做了限制不能超过5000个ID，因此我们的获取ID函数会一直询问Redis。只有在start/count参数超出了这个范围的时候，才需要去访问数据库。
>  我们的系统不会像传统方式那样“刷新”缓存，Redis实例中的信息永远是一致的。SQL数据库(或是硬盘上的其他类型数据库)只是在用户需要获取“很远”的数据时才会被触发，而主页或第一个评论页是不会麻烦到硬盘上的数据库了。
>  **2、删除与过滤**
>  我们可以使用LREM来删除评论。如果删除操作非常少，另一个选择是直接跳过评论条目的入口，报告说该评论已经不存在。
>  有些时候你想要给不同的列表附加上不同的过滤器。如果过滤器的数量受到限制，你可以简单的为每个不同的过滤器使用不同的Redis列表。毕竟每个列表只有5000条项目，但Redis却能够使用非常少的内存来处理几百万条项目。
>  **3、排行榜相关**
>  ==另一个很普遍的需求是各种数据库的数据并非存储在内存中，因此在按得分排序以及实时更新这些几乎每秒钟都需要更新的功能上数据库的性能不够理想。==
>  典型的比如那些在线游戏的排行榜，比如一个Facebook的游戏，根据得分你通常想要：
>  – 列出前100名高分选手
>  – 列出某用户当前的全球排名
>  这些操作对于Redis来说小菜一碟，即使你有几百万个用户，每分钟都会有几百万个新的得分。
>  模式是这样的，每次获得新得分时，我们用这样的代码：`ZADD leaderboard`
>  ****你可能用userID来取代username，这取决于你是怎么设计的。**
> **得到前100名高分用户很简单： `ZREVRANGE leaderboard 0 99`。**
>  **用户的全球排名也相似，只需要： `ZRANK leaderboard`。**
> **4、按照用户投票和时间排序**
>  排行榜的一种常见变体模式就像Reddit或Hacker News用的那样，新闻按照类似下面的公式根据得分来排序：
>  `score = points / time^alpha`
>  因此用户的投票会相应的把新闻挖出来，但时间会按照一定的指数将新闻埋下去。下面是我们的模式，当然算法由你决定。
>  模式是这样的，开始时先观察那些可能是最新的项目，例如首页上的1000条新闻都是候选者，因此我们先忽视掉其他的，这实现起来很简单。
>  每次新的新闻贴上来后，我们将ID添加到列表中，使用 `LPUSH + LTRIM`，确保只取出最新的1000条项目。
>  有一项后台任务获取这个列表，并且持续的计算这1000条新闻中每条新闻的最终得分。计算结果由ZADD命令按照新的顺序填充生成列表，老新闻则被清除。这里的关键思路是排序工作是由后台任务来完成的。
>  **5、处理过期项目**
>  另一种常用的项目排序是按照时间排序。我们使用unix时间作为得分即可。
>  模式如下：
>  – 每次有新项目添加到我们的非Redis数据库时，我们把它加入到排序集合中。这时我们用的是时间属性，current_time和time_to_live。
>  – 另一项后台任务使用ZRANGE…SCORES查询排序集合，取出最新的10个项目。如果发现unix时间已经过期，则在数据库中删除条目。
>  **6、计数**
>  Redis是一个很好的计数器，这要感谢INCRBY和其他相似命令。
>  我相信你曾许多次想要给数据库加上新的计数器，用来获取统计或显示新信息，但是最后却由于写入敏感而不得不放弃它们。
>  好了，现在使用Redis就不需要再担心了。有了原子递增(atomic increment)，你可以放心的加上各种计数，用GETSET重置，或者是让它们过期。
>  例如这样操作：
>
> ```
> INCR user: EXPIRE
> user: 60
> ```
>
> 你可以计算出最近用户在页面间停顿不超过60秒的页面浏览量，当计数达到比如20时，就可以显示出某些条幅提示，或是其它你想显示的东西。
>  **7、特定时间内的特定项目**
>  另一项对于其他数据库很难，但Redis做起来却轻而易举的事就是统计在某段特点时间里有多少特定用户访问了某个特定资源。比如我想要知道某些特定的注册用户或IP地址，他们到底有多少访问了某篇文章。
>  每次我获得一次新的页面浏览时我只需要这样做：
>  `SADD page:day1:`
>  当然你可能想用unix时间替换day1，比如time()-(time()%3600*24)等等。
>  想知道特定用户的数量吗?只需要使`用SCARD page:day1:`。
>  需要测试某个特定用户是否访问了这个页面?SISMEMBER page:day1: 。
>  **8、实时分析正在发生的情况，用于数据统计与防止垃圾邮件等**
>  我们只做了几个例子，但如果你研究Redis的命令集，并且组合一下，就能获得大量的实时分析方法，有效而且非常省力。使用Redis原语命令，更容易实施垃圾邮件过滤系统或其他实时跟踪系统。
>
> **9、Pub/Sub**
>  Redis的Pub/Sub非常非常简单，运行稳定并且快速。支持模式匹配，能够实时订阅与取消频道。
>  **10、队列**
>  你应该已经注意到像list push和list pop这样的Redis命令能够很方便的执行队列操作了，但能做的可不止这些：比如Redis还有list pop的变体命令，能够在列表为空时阻塞队列。
>  现代的互联网应用大量地使用了消息队列(Messaging)。消息队列不仅被用于系统内部组件之间的通信，同时也被用于系统跟其它服务之间的交互。消息队列的使用可以增加系统的可扩展性、灵活性和用户体验。非基于消息队列的系统，其运行速度取决于系统中最慢的组件的速度(注：短板效应)。而基于消息队列可以将系统中各组件解除耦合，这样系统就不再受最慢组件的束缚，各组件可以异步运行从而得以更快的速度完成各自的工作。
>  此外，当服务器处在高并发操作的时候，比如频繁地写入日志文件。可以利用消息队列实现异步处理。从而实现高性能的并发操作。

参照：[《聊聊 Redis 使用场景》](http://blog.720ui.com/2017/redis_core_use/)

- [《Redis 常见的应用场景解析》](https://zhuanlan.zhihu.com/p/29665317)

# 六、Redis实现分布式锁

## **方案一：set 指令**

==先拿 **setnx** 来争抢锁，抢到之后，再用 expire 给锁加一个过期时间防止锁忘记了释放。==

> set 指令有非常复杂的参数，可以同时把 setnx 和 expire 合成一条指令来用的，防止在 setnx 之后执行 expire 之前进程意外 crash 或者要重启维护了。

可以使用 **set** 指令，实现分布式锁。指令如下：

```bash
SET key value [EX seconds] [PX milliseconds] [NX|XX]
```

- 可以使用 `SET key value EX seconds NX` 命令，尝试获得锁。

> 一般通过`setnx+lua`实现，或者`set key value px milliseconds nx`。后一种方式的核心实现命令如下：
>
> ```lua
> - 获取锁（unique_value可以是UUID等）
> SET resource_name unique_value NX PX 30000
> 
> - 释放锁（lua脚本中，一定要比较value，防止误解锁）
> if redis.call("get",KEYS[1]) == ARGV[1] then
>     return redis.call("del",KEYS[1])
> else
>     return 0
> end
> ```
>
> 这种实现方式有3大要点：
>
> 1. set命令要用`set key value px milliseconds nx`；
> 2. value要具有唯一性；
> 3. 释放锁时要验证value值，不能误解锁；
>
> **事实上这类琐最大的缺点就是它加锁时只作用在一个Redis节点上，即使Redis通过sentinel保证高可用，如果这个master节点由于某些原因发生了主从切换，那么就会出现锁丢失的情况：**
>
> 1. 在Redis的master节点上拿到了锁；
> 2. 但是这个加锁的key还没有同步到slave节点；
> 3. master故障，发生故障转移，slave节点升级为master节点；
> 4. 导致锁丢失。

### 具体的实现：

​	分布式锁一般有三种实现方式：1. 数据库乐观锁；2. 基于Redis的分布式锁；3. 基于ZooKeeper的分布式锁。

​	**首先，为了确保分布式锁可用，我们至少要确保锁的实现同时满足以下四个条件：**

1. **互斥性。**在任意时刻，只有一个客户端能持有锁。
2. **不会发生死锁。**即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。
3. **具有容错性。**只要大部分的Redis节点正常运行，客户端就可以加锁和解锁。
4. **解铃还须系铃人。**加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了。

#### 	代码实现：

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

```java
public class RedisTool {
    private static final String LOCK_SUCCESS = "OK";
    private static final String SET_IF_NOT_EXIST = "NX";
    private static final String SET_WITH_EXPIRE_TIME = "PX";
    /** 尝试获取分布式锁 
    @param jedis Redis客户端 
    @param lockKey 锁 
    @param requestId 请求标识 
    @param expireTime 超期时间 
    @return 是否获取成功 
    */
    public static boolean tryGetDistributedLock(Jedis jedis, String lockKey, String requestId, int expireTime) {

        String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);

        if (LOCK_SUCCESS.equals(result)) {
            return true;
        }
        return false;
    }
}
```

​	上面的加锁就一行代码：`jedis.set(String key, String value, String nxxx, String expx, int time)`，这个set()方法一共有五个形参：

- 第一个为key，使用key来当锁，因为key是唯一的。
- 第二个为value，传的是requestId，除了key作为锁，因为要保证可靠性，即分布式锁要满足第四个条件**解铃还须系铃人**，通过给value赋值为requestId，**这样就知道这把锁是哪个请求加的了**，在解锁的时候就可以有依据。==requestId可以使用`UUID.randomUUID().toString()`方法生成。==
- 第三个为nxxx，**这个参数是NX，意思是SET IF NOT EXIST，即当key不存在时，我们进行set操作；若key已经存在，则不做任何操作；**
- 第四个为expx，**这个参数传的是PX，意思是我们要给这个key加一个过期的设置，具体时间由第五个参数决定。**
- **第五个为time，与第四个参数相呼应，代表key的过期时间。**

​	==总的来说，执行上面的set()方法就只会导致两种结果：1. 当前没有锁（key不存在），那么就进行加锁操作，并对锁设置个有效期，同时value表示加锁的客户端。2. 已有锁存在，不做任何操作。==

​	上面的加锁代码满足**可靠性**里描述的三个条件：首先，set()加入了NX参数，可以保证如果已有key存在，则函数不会调用成功，也就是只有一个客户端能持有锁，满足互斥性。其次，**由于对锁设置了过期时间，即使锁的持有者后续发生崩溃而没有解锁，锁也会因为到了过期时间而自动解锁（即key被删除），不会发生死锁。**最后，因为将value赋值为requestId，代表加锁的客户端请求标识，**那么在客户端在解锁的时候就可以进行校验是否是同一个客户端。**由于我们只考虑Redis单机部署的场景，所以容错性我们暂不考虑。

#### 错误示例1

比较常见的错误示例就是使用`jedis.setnx()`和`jedis.expire()`组合实现加锁，代码如下：

```java
public static void wrongGetLock1(Jedis jedis, String lockKey, String requestId, int expireTime) {

    Long result = jedis.setnx(lockKey, requestId);
    if (result == 1) {
        // 若在这里程序突然崩溃，则无法设置过期时间，将发生死锁
        jedis.expire(lockKey, expireTime);
    }
}
```

​	==setnx()方法作用就是SET IF NOT EXIST，expire()方法就是给锁加一个过期时间。==乍一看好像和前面的set()方法结果一样，然而**由于这是两条Redis命令，不具有原子性**，如果程序在执行完setnx()之后突然崩溃，导致锁没有设置过期时间。那么将会发生死锁。网上之所以有人这样实现，是因为低版本的jedis并不支持多参数的set()方法。

#### 错误示例2

​	这一种错误示例就比较难以发现问题，而且实现也比较复杂。实现思路：使用`jedis.setnx()`命令实现加锁，其中key是锁，value是锁的过期时间。执行过程：1. 通过setnx()方法尝试加锁，如果当前锁不存在，返回加锁成功。2. 如果锁已经存在则获取锁的过期时间，和当前时间比较，如果锁已经过期，则设置新的过期时间，返回加锁成功。

```java
public static boolean wrongGetLock2(Jedis jedis, String lockKey, int expireTime) {
    long expires = System.currentTimeMillis() + expireTime;
    String expiresStr = String.valueOf(expires);
    // 如果当前锁不存在，返回加锁成功
    if (jedis.setnx(lockKey, expiresStr) == 1) {
        return true;
    }
    // 如果锁存在，获取锁的过期时间
    String currentValueStr = jedis.get(lockKey);
    if (currentValueStr != null && Long.parseLong(currentValueStr) < System.currentTimeMillis()) {
        // 锁已过期，获取上一个锁的过期时间，并设置现在锁的过期时间
        String oldValueStr = jedis.getSet(lockKey, expiresStr);
        if (oldValueStr != null && oldValueStr.equals(currentValueStr)) {
            // 考虑多线程并发的情况，只有一个线程的设置值和当前值相同，它才有权利加锁
            return true;
        }
    }
    // 其他情况，一律返回加锁失败
    return false;
}
```

​	那么这段代码问题在于：

1. **由于是客户端自己生成过期时间，所以需要强制要求分布式下每个客户端的时间必须同步。** 
2. 当锁过期的时候，如果多个客户端同时执行`jedis.getSet()`方法，那么虽然最终只有一个客户端可以加锁，但是这个客户端的锁的过期时间可能被其他客户端覆盖。
3. 锁不具备拥有者标识，即任何客户端都可以解锁。

### 正确代码

```java
public class RedisTool {
    private static final Long RELEASE_SUCCESS = 1L;
    /** 
    释放分布式锁
    @param jedis Redis客户端 
    @param lockKey 锁 
    @param requestId 请求标识 
    @return 是否释放成功 
    */
    public static boolean releaseDistributedLock(Jedis jedis, String lockKey, String requestId) {
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));
        if (RELEASE_SUCCESS.equals(result)) {
            return true;
        }
        return false;
    }
}
```

​	可以看到，解锁只需要两行代码就搞定了！第一行代码，是一个简单的Lua脚本代码。第二行代码**，将Lua代码传到`jedis.eval()`方法里，并使参数KEYS[1]赋值为lockKey，ARGV[1]赋值为requestId。eval()方法是将Lua代码交给Redis服务端执行。**

​	这段Lua代码的功能：首先获取锁对应的value值，检查是否与requestId相等，如果相等则删除锁（解锁）。使用Lua语言是为了确保上述操作是原子性的，eval()方法可以确保原子性，**在eval命令执行Lua代码的时候，Lua代码将被当成一个命令去执行，并且直到eval命令执行完成，Redis才会执行其他命令。**

#### 错误示例1

​	最常见的解锁代码就是直接使用`jedis.del()`方法删除锁，这种不先判断锁的拥有者而直接解锁的方式，会导致任何客户端都可以随时进行解锁，即使这把锁不是它的。

```java
public static void wrongReleaseLock1(Jedis jedis, String lockKey) {
    jedis.del(lockKey);
}
```

#### 错误示例2

​	这种解锁代码乍一看也是没问题，甚至我之前也差点这样实现，与正确代码差不多，唯一区别的是分成两条命令去执行，代码如下：

```java
public static void wrongReleaseLock2(Jedis jedis, String lockKey, String requestId) {
    // 判断加锁与解锁是不是同一个客户端
    if (requestId.equals(jedis.get(lockKey))) {
        // 若在此时，这把锁突然不是这个客户端的，则会误解锁
        jedis.del(lockKey);
    }
}
```

​	如代码注释，问题在于如果调用`jedis.del()`方法的时候，这把锁已经不属于当前客户端的时候会解除他人加的锁。比如客户端A加锁，一段时间之后客户端A解锁，在执行`jedis.del()`之前，锁突然过期了，此时客户端B尝试加锁成功，然后客户端A再执行del()方法，则将客户端B的锁给解除了。

​	此外，如果你的项目中Redis是多机部署的，那么可以尝试使用`Redisson`实现分布式锁，这是Redis官方提供的Java组件。

参照 [《Redis 分布式锁的正确实现方式（Java版）》](https://wudashan.cn/2017/10/23/Redis-Distributed-Lock-Implement/) 文章。

## **方案二：Redlock**

​	set 指令的方案，适合用于在单机 Redis 节点的场景下，在多 Redis 节点的场景下，会存在分布式锁丢失的问题。所以，Redis 作者 Antirez 基于分布式环境下提出了一种更高级的分布式锁的实现方式：Redlock 。

### 	Redlock实现

​	antirez提出的redlock算法大概是这样的：在Redis的分布式环境中，我们假设有N个Redis master。这些节点**完全互相独立，不存在主从复制或者其他集群协调机制**。我们确保将在N个实例上使用与在Redis单实例下相同方法获取和释放锁。现在我们假设有5个Redis master节点，同时我们需要在5台服务器上面运行这些Redis实例，这样保证他们不会同时都宕掉。

为了取到锁，客户端应该执行以下操作:

- 获取当前Unix时间，以毫秒为单位。
- 依次尝试从5个实例，使用相同的key和**具有唯一性的value**（例如UUID）获取锁。当向Redis请求获取锁时，客户端应该设置一个网络连接和响应超时时间，这个超时时间应该小于锁的失效时间。例如你的锁自动失效时间为10秒，则超时时间应该在5-50毫秒之间。这样可以避免服务器端Redis已经挂掉的情况下，客户端还在死死地等待响应结果。如果服务器端没有在规定时间内响应，客户端应该尽快尝试去另外一个Redis实例请求获取锁。
- 客户端使用当前时间减去开始获取锁时间（步骤1记录的时间）就得到获取锁使用的时间。**当且仅当从大多数**（N/2+1，这里是3个节点）**的Redis节点都取到锁，并且使用的时间小于锁失效时间时，锁才算获取成功**。
- 如果取到了锁，key的真正有效时间等于有效时间减去获取锁所使用的时间（步骤3计算的结果）。
- 如果因为某些原因，获取锁失败（没有在至少N/2+1个Redis实例取到锁或者取锁时间已经超过了有效时间），客户端应该在**所有的Redis实例上进行解锁**（即便某些Redis实例根本就没有加锁成功，防止某些节点获取到锁但是客户端没有得到响应而导致接下来的一段时间不能被重新获取锁）。

### Redlock源码

​	redisson已经有对redlock算法封装，接下来对其用法进行简单介绍，并对核心源码进行分析（**假设5个redis实例**）。

```xml
<!-- https://mvnrepository.com/artifact/org.redisson/redisson -->
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.3.2</version>
</dependency>
```

#### 用法

​	redission封装的redlock算法实现的分布式锁用法，跟重入锁（ReentrantLock）有点类似：

```java
Config config = new Config();
config.useSentinelServers().addSentinelAddress("127.0.0.1:6369","127.0.0.1:6379", "127.0.0.1:6389")
        .setMasterName("masterName")
        .setPassword("password").setDatabase(0);
RedissonClient redissonClient = Redisson.create(config);
// 还可以getFairLock(), getReadWriteLock()
RLock redLock = redissonClient.getLock("REDLOCK_KEY");
boolean isLock;
try {
    isLock = redLock.tryLock();
    // 500ms拿不到锁, 就认为获取锁失败。10000ms即10s是锁失效时间。
    isLock = redLock.tryLock(500, 10000, TimeUnit.MILLISECONDS);
    if (isLock) {
        //TODO if get lock success, do something;
    }
} catch (Exception e) {
} finally {
    // 无论如何, 最后都要解锁
    redLock.unlock();
}
```

#### 唯一ID

​	实现分布式锁的一个非常重要的点就是set的value要具有唯一性，redisson的value是通过**UUID+threadId**保证value的唯一性。入口在redissonClient.getLock("REDLOCK_KEY")，源码在Redisson.java和RedissonLock.java中：

```java
protected final UUID id = UUID.randomUUID();
String getLockName(long threadId) {
    return id + ":" + threadId;
}
```

#### 获取锁

​	获取锁的代码为redLock.tryLock()或者redLock.tryLock(500, 10000, TimeUnit.MILLISECONDS)，两者的最终核心源码都是下面这段代码，只不过前者获取锁的默认租约时间（leaseTime）是LOCK_EXPIRATION_INTERVAL_SECONDS，即30s：

```java
<T> RFuture<T> tryLockInnerAsync(long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
    internalLockLeaseTime = unit.toMillis(leaseTime);
    // 获取锁时向5个redis实例发送的命令
    return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, command,
    // 首先分布式锁的KEY不能存在，如果确实不存在，那么执行hset命令（hset REDLOCK_KEY uuid+threadId 1），并通过pexpire设置失效时间（也是锁的租约时间）
              "if (redis.call('exists', KEYS[1]) == 0) then " +
                  "redis.call('hset', KEYS[1], ARGV[2], 1); " +
                  "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                  "return nil; " +
              "end; " +
// 如果分布式锁的KEY已经存在，并且value也匹配，表示是当前线程持有的锁，那么重入次数加1，并且设置失效时间
              "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                  "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                  "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                  "return nil; " +
              "end; " +
              // 获取分布式锁的KEY的失效时间毫秒数
              "return redis.call('pttl', KEYS[1]);",
              // 这三个参数分别对应KEYS[1]，ARGV[1]和ARGV[2]
                Collections.<Object>singletonList(getName()), internalLockLeaseTime, getLockName(threadId));
}
```

获取锁的命令中，

- **KEYS[1]**就是Collections.singletonList(getName())，表示分布式锁的key，即REDLOCK_KEY；
- **ARGV[1]**就是internalLockLeaseTime，即锁的租约时间，默认30s；
- **ARGV[2]**就是getLockName(threadId)，是获取锁时set的唯一值，即UUID+threadId：

------

#### 释放锁

​	释放锁的代码为redLock.unlock()，核心源码如下：

```java
protected RFuture<Boolean> unlockInnerAsync(long threadId) {
    // 向5个redis实例都执行如下命令
    return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
            // 如果分布式锁KEY不存在，那么向channel发布一条消息
            "if (redis.call('exists', KEYS[1]) == 0) then " +
                "redis.call('publish', KEYS[2], ARGV[1]); " +
                "return 1; " +
            "end;" +
            // 如果分布式锁存在，但是value不匹配，表示锁已经被占用，那么直接返回
            "if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then " +
                "return nil;" +
            "end; " +
            // 如果就是当前线程占有分布式锁，那么将重入次数减1
            "local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); " +
            // 重入次数减1后的值如果大于0，表示分布式锁有重入过，那么只设置失效时间，还不能删除
            "if (counter > 0) then " +
                "redis.call('pexpire', KEYS[1], ARGV[2]); " +
                "return 0; " +
            "else " +
                // 重入次数减1后的值如果为0，表示分布式锁只获取过1次，那么删除这个KEY，并发布解锁消息
                "redis.call('del', KEYS[1]); " +
                "redis.call('publish', KEYS[2], ARGV[1]); " +
                "return 1; "+
            "end; " +
            "return nil;",
            // 这5个参数分别对应KEYS[1]，KEYS[2]，ARGV[1]，ARGV[2]和ARGV[3]
            Arrays.<Object>asList(getName(), getChannelName()), LockPubSub.unlockMessage, internalLockLeaseTime, getLockName(threadId));

}
```

#### 具体实现：

##### 单机Redis

```java
Config config1 = new Config();
config1.useSingleServer().setAddress("redis://172.29.1.180:5378")
        .setPassword("a123456").setDatabase(0);
RedissonClient redissonClient1 = Redisson.create(config1);

Config config2 = new Config();
config2.useSingleServer().setAddress("redis://172.29.1.180:5379")
        .setPassword("a123456").setDatabase(0);
RedissonClient redissonClient2 = Redisson.create(config2);

Config config3 = new Config();
config3.useSingleServer().setAddress("redis://172.29.1.180:5380")
        .setPassword("a123456").setDatabase(0);
RedissonClient redissonClient3 = Redisson.create(config3);

String resourceName = "REDLOCK";
RLock lock1 = redissonClient1.getLock(resourceName);
RLock lock2 = redissonClient2.getLock(resourceName);
RLock lock3 = redissonClient3.getLock(resourceName);

RedissonRedLock redLock = new RedissonRedLock(lock1, lock2, lock3);
boolean isLock;
try {
    isLock = redLock.tryLock(500, 30000, TimeUnit.MILLISECONDS);
    System.out.println("isLock = "+isLock);
    if (isLock) {
        //TODO if get lock success, do something;
        Thread.sleep(30000);
    }
} catch (Exception e) {
} finally {
    // 无论如何, 最后都要解锁
    System.out.println("");
    redLock.unlock();
}
```

最核心的变化就是`RedissonRedLock redLock = new RedissonRedLock(lock1, lock2, lock3);`，这里用到三个节点。

参照两篇博客：

- [《Redlock：Redis分布式锁最牛逼的实现》](https://mp.weixin.qq.com/s/JLEzNqQsx-Lec03eAsXFOQ)
- [《Redisson 实现 Redis 分布式锁的 N 种姿势》](https://www.jianshu.com/p/f302aa345ca8)

## **对比 Zookeeper 分布式锁**

- 从可靠性上来说，Zookeeper 分布式锁好于 Redis 分布式锁。
- 从性能上来说，Redis 分布式锁好于 Zookeeper 分布式锁。

所以，没有绝对的好坏，可以根据自己的业务来具体选择。

# 七、Redis实现消息队列

​	一般使用 list 结构作为队列，rpush 生产消息，lpop 消费消息。当 lpop 没有消息的时候，要适当 sleep 一会再重试。

- 除了sleep，list 还有个指令叫 blpop ，在没有消息的时候，它会阻塞住直到消息到来。
- 使用 pub / sub 主题订阅者模式，可以实现 生产一次消费多次 ，即1:N 的消息队列，
- pub / sub的缺点是：在消费者下线的情况下，生产的消息会丢失，得使用专业的消息队列如 rabbitmq 等。
- **可以通过使用 sortedset ，拿时间戳作为 score ，消息内容作为 key 调用 zadd 来生产消息，消费者用 zrangebyscore 指令获取 N 秒之前的数据轮询进行处理，实现延时队列。**

​	当然，实际上 Redis 真的真的真的不推荐作为消息队列使用，它最多只是消息队列的存储层，上层的逻辑，还需要做大量的封装和支持。

另外，在 Redis 5.0 增加了 Stream 功能，一个新的强大的支持多播的可持久化的消息队列，提供类似 Kafka 的功能。

# 八、Redis集群

Redis 集群方案如下：

- 1、Redis Sentinel
- 2、Redis Cluster
- 3、Twemproxy
- 4、Codis
- 5、客户端分片

下面主要看前四种的具体实现：

### 1.Replication（主从复制）

​	Redis的replication机制允许slave从master那里通过网络传输拷贝到完整的数据备份，从而达到主从机制。为了实现主从复制，我们准备三个redis服务，依次命名为master，slave1，slave2。

**配置主服务器**

修改主服务器的配置文件redis.conf的端口信息	port 6300

**配置从服务器**

​	replication相关的配置比较简单，只需要把下面一行加到slave的配置文件中。你只需要把ip地址和端口号改一下。

```
slaveof 192.168.1.1 6379
```

修改从服务器1的配置文件redis.conf的端口信息和从服务器配置。

```
port 6301slaveof 127.0.0.1 6300
```

再修改从服务器2的配置文件redis.conf的端口信息和从服务器配置。

```
port 6302slaveof 127.0.0.1 6300
```

​	从redis2.6版本开始，slave支持只读模式，而且是默认的。可以通过配置项slave-read-only来进行配置。
此外，**如果master通过requirepass配置项设置了密码，slave每次同步操作都需要验证密码，可以通过在slave的配置文件中添加以下配置项**

```
masterauth <password>
```

### 2.Sentinel（哨兵）

​	主从机制，上面的方案中主服务器可能存在单点故障，万一主服务器宕机，这是个麻烦事情，所以Redis提供了Redis-Sentinel，**以此来实现主从切换的功能，类似与zookeeper。**

​	Redis-Sentinel是Redis官方推荐的高可用性(HA)解决方案，当用Redis做master-slave的高可用方案时，假如master宕机了，Redis本身(包括它的很多客户端)都没有实现自动进行主备切换，而Redis-Sentinel本身也是一个独立运行的进程，它能监控多个master-slave集群，发现master宕机后能进行自动切换。

它的主要功能有以下几点

- 监控（Monitoring）：不断地检查redis的主服务器和从服务器是否运作正常。
- 提醒（Notification）：如果发现某个redis服务器运行出现状况，可以通过 API 向管理员或者其他应用程序发送通知。
- 自动故障迁移（Automatic failover）：能够进行自动切换。当一个主服务器不能正常工作时，会将失效主服务器的其中一个从服务器升级为新的主服务器，并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。

Redis Sentinel 兼容 Redis 2.4.16 或以上版本， 推荐使用 Redis 2.8.0 或以上的版本。

#### 配置Sentinel

​	必须指定一个sentinel的配置文件sentinel.conf，如果不指定将无法启动sentinel。首先，我们先创建一个配置文件sentinel.conf

```
port 26379sentinel monitor mymaster 127.0.0.1 6300 2
```

官方典型的配置如下

```conf
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout mymaster 180000
sentinel parallel-syncs mymaster 1
sentinel monitor resque 192.168.1.3 6380 4
sentinel down-after-milliseconds resque 10000
sentinel failover-timeout resque 180000
sentinel parallel-syncs resque 5
```

​	配置文件只需要配置master的信息就好啦，不用配置slave的信息，因为slave能够被自动检测到(master节点会有关于slave的消息)。

​	**需要注意的是，配置文件在sentinel运行期间是会被动态修改的，例如当发生主备切换时候，配置文件中的master会被修改为另外一个slave。这样，之后sentinel如果重启时，就可以根据这个配置来恢复其之前所监控的redis集群的状态。**

接下来我们将一行一行地解释上面的配置项：

> **上面的配置中，第一行指示 Sentinel 去监视一个名为 mymaster 的主服务器， 这个主服务器的 IP 地址为 127.0.0.1 ， 端口号为 6300， 而将这个主服务器判断为失效至少需要 2 个 Sentinel 同意，只要同意 Sentinel 的数量不达标，自动故障迁移就不会执行。**
>
> ​	不过要注意， 无论设置要多少个 Sentinel ，只有统一才能判断一个服务器失效， 一个 Sentinel 都需要获得系统中多数（majority） Sentinel 的支持， 才能发起一次自动故障迁移， 并预留一个给定的配置纪元 （configuration Epoch ，一个配置纪元就是一个新主服务器配置的版本号）。换句话说， 在只有少数（minority） Sentinel 进程正常运作的情况下， Sentinel 是不能执行自动故障迁移的。sentinel集群中各个sentinel也有互相通信，通过gossip协议。
>
> 第二行开始：
>
> ```
> sentinel <option_name> <master_name> <option_value>
> ```
>
> - down-after-milliseconds 选项指定了 Sentinel 认为服务器已经断线所需的毫秒数。
>
> - parallel-syncs 选项指定了在执行故障转移时， 最多可以有多少个从服务器同时对新的主服务器进行同步， 这个数字越小， 完成故障转移所需的时间就越长。

#### 启动 Sentinel

对于 redis-sentinel 程序， 你可以用以下命令来启动 Sentinel 系统

```
redis-sentinel sentinel.conf
```

对于 redis-server 程序， 你可以用以下命令来启动一个运行在 Sentinel 模式下的 Redis 服务器

```
redis-server sentinel.conf --sentinel
```

​	以上两种方式，都必须指定一个sentinel的配置文件sentinel.conf， 如果不指定将无法启动sentinel。sentinel默认监听26379端口，所以运行前必须确定该端口没有被别的进程占用。

### 3.Twemproxy

​	Twemproxy是由Twitter开源的Redis代理， Redis客户端把请求发送到Twemproxy，Twemproxy根据路由规则发送到正确的Redis实例，最后Twemproxy把结果汇集返回给客户端。

​	Twemproxy通过引入一个代理层，将多个Redis实例进行统一管理，使Redis客户端只需要在Twemproxy上进行操作，而不需要关心后面有多少个Redis实例，从而实现了Redis集群。Twemproxy本身也是单点，需要用Keepalived做高可用方案。

​	但是，Twemproxy存在诸多不方便之处，最主要的是，Twemproxy无法平滑地增加Redis实例，业务量突增，需增加Redis服务器；业务量萎缩，需要减少Redis服务器。但对Twemproxy而言，基本上都很难操作。其次，没有友好的监控管理后台界面，不利于运维监控。

### 4.Codis

​	Codis解决了Twemproxy的这两大痛点，由豌豆荚于2014年11月开源，基于Go和C开发、现已广泛用于豌豆荚的各种Redis业务场景。

Codis 3.x 由以下组件组成：

- Codis Server：基于 redis-2.8.21 分支开发。增加了额外的数据结构，以支持 slot 有关的操作以及数据迁移指令。具体的修改可以参考文档 redis 的修改。
- Codis Proxy：客户端连接的 Redis 代理服务, 实现了 Redis 协议。 除部分命令不支持以外(不支持的命令列表)，表现的和原生的 Redis 没有区别（就像 Twemproxy）。对于同一个业务集群而言，可以同时部署多个 codis-proxy 实例；不同 codis-proxy 之间由 codis-dashboard 保证状态同步。
- Codis Dashboard：集群管理工具，支持 codis-proxy、codis-server 的添加、删除，以及据迁移等操作。在集群状态发生改变时，codis-dashboard 维护集群下所有 codis-proxy 的状态的一致性。对于同一个业务集群而言，同一个时刻 codis-dashboard 只能有 0个或者1个；所有对集群的修改都必须通过 codis-dashboard 完成。
- Codis Admin：集群管理的命令行工具。可用于控制 codis-proxy、codis-dashboard 状态以及访问外部存储。
- Codis FE：集群管理界面。多个集群实例共享可以共享同一个前端展示页面；通过配置文件管理后端 codis-dashboard 列表，配置文件可自动更新。
- Codis HA：为集群提供高可用。依赖 codis-dashboard 实例，自动抓取集群各个组件的状态；会根据当前集群状态自动生成主从切换策略，并在需要时通过 codis-dashboard 完成主从切换。
- Storage：为集群状态提供外部存储。提供 Namespace 概念，不同集群的会按照不同 product name 进行组织；目前仅提供了 Zookeeper 和 Etcd 两种实现，但是提供了抽象的 interface 可自行扩展。

Codis引入了Group的概念，每个Group包括1个Redis Master及一个或多个Redis Slave，这是和Twemproxy的区别之一，实现了Redis集群的高可用。当1个Redis Master挂掉时，Codis不会自动把一个Slave提升为Master，这涉及数据的一致性问题，Redis本身的数据同步是采用主从异步复制，当数据在Maste写入成功时，Slave是否已读入这个数据是没法保证的，需要管理员在管理界面上手动把Slave提升为Master。

Codis使用，可以参考官方文档<https://github.com/CodisLabs/codis/blob/release3.0/doc/tutorial_zh.md>

### 5.Redis 3.0集群

​	Redis 3.0集群采用了P2P的模式，完全去中心化。支持多节点数据集自动分片，提供一定程度的分区可用性，部分节点挂掉或者无法连接其他节点后，服务可以正常运行。Redis 3.0集群采用Hash Slot方案，而不是一致性哈希。Redis把所有的Key分成了16384个slot，每个Redis实例负责其中一部分slot。集群中的所有信息（节点、端口、slot等），都通过节点之间定期的数据交换而更新。

​	Redis客户端在任意一个Redis实例发出请求，如果所需数据不在该实例中，通过重定向命令引导客户端访问所需的实例。

Redis 3.0集群，目前支持的cluster特性

- 节点自动发现
- slave->master 选举,集群容错
- Hot resharding:在线分片
- 集群管理:cluster xxx
- 基于配置(nodes-port.conf)的集群管理
- ASK 转向/MOVED 转向机制

关于前四种参照 [《Redis 实战（四）集群机制》](http://blog.720ui.com/2016/redis_action_04_cluster/) 。

与Redis分区类似，参照： [《Redis 分区》](http://www.runoob.com/redis/redis-partitioning.html) 文章。

> Redis 分区方案，主要分成两种类型：
>
> - **客户端分区**，就是在客户端就已经决定数据会被存储到哪个 Redis 节点或者从哪个 Redis 节点读取。大多数客户端已经实现了客户端分区。
>   - 案例：Redis Cluster 和客户端分区。
> - **代理分区**，意味着客户端将请求发送给代理，然后代理决定去哪个节点写数据或者读数据。代理根据分区规则决定请求哪些 Redis 实例，然后根据 Redis 的响应结果返回给客户端。
>   - 案例：Twemproxy 和 Codis 。
>
> 查询路由(Query routing)的意思，是客户端随机地请求任意一个 Redis 实例，然后由 Redis 将请求转发给正确的 Redis 节点。Redis Cluster 实现了一种混合形式的查询路由，但并不是直接将请求从一个Redis 节点转发到另一个 Redis 节点，而是在客户端的帮助下直接 redirect 到正确的 Redis 节点。

关于最后一种，客户端分片，在 Redis Cluster 出现之前使用较多，目前已经使用比较少了。实现方式如下：

> ​	在业务代码层实现，起几个毫无关联的 Redis 实例，**在代码层，对 Key 进行 hash 计算，然后去对应的 Redis 实例操作数据。**
>
> ​	这种方式对 hash 层代码要求比较高，考虑部分包括，节点失效后的替代算法方案，数据震荡后的自动脚本恢复，实例的监控，等等。

目前一般在选型上来说：

- 体量较小时，选择 Redis Sentinel ，单主 Redis 足以支撑业务。
- 体量较大时，选择 Redis Cluster ，通过分片，使用更多内存。

 **Redis 集群扩容**

- 如果 Redis 被当做**缓存**使用，使用一致性哈希实现动态扩容缩容。
- 如果 Redis 被当做一个**持久化存**储使用，必须使用固定的 keys-to-nodes 映射关系，节点的数量一旦确定不能变化。否则的话(即Redis 节点需要动态变化的情况），必须使用可以在运行时进行数据再平衡的一套系统，而当前只有 Redis Cluster、Codis 可以做到这样。

# 九、Redis 主从同步

​	==Redis 的主从同步(replication)机制，允许 Slave 从 Master 那里，通过网络传输拷贝到完整的数据备份，从而达到主从机制。==

- 主数据库可以进行读写操作，当发生写操作的时候自动将数据同步到从数据库，而从数据库一般是只读的，并接收主数据库同步过来的数据。
- 一个主数据库可以有多个从数据库，而一个从数据库只能有一个主数据库。
- 第一次同步时，主节点做一次 bgsave 操作，并同时将后续修改操作记录到内存 buffer ，待完成后将 RDB 文件全量同步到复制节点，复制节点接受完成后将 RDB 镜像加载到内存。加载完成后，再通知主节点将期间修改的操作记录同步到复制节点进行重放就完成了同步过程。

**好处**

通过 Redis 的复制功，能可以很好的实现数据库的读写分离，提高服务器的负载能力。主数据库主要进行写操作，而从数据库负责读操作。

​	为了使在部分节点失败或者大部分节点无法通信的情况下集群仍然可用，所以集群使用了**主从复制**模型，每个节点都会有 N-1 个复制节点。

所以，Redis Cluster 可以说是 Redis Sentinel 带分片的加强版。也可以说：

- Redis Sentinel 着眼于高可用，在 master 宕机时会自动将 slave 提升为 master ，继续提供服务。
- Redis Cluster 着眼于扩展性，在单个 Redis 内存不足时，使用Cluster 进行分片存储。

Redis 主从同步，是很多 Redis 集群方案的基础，例如 Redis Sentinel、Redis Cluster 等等。

## redis replication 的核心机制

- redis 采用**异步方式**复制数据到 slave 节点，不过 redis2.8 开始，slave node 会周期性地确认自己每次复制的数据量；
- 一个 master node 是可以配置多个 slave node 的；
- slave node 也可以连接其他的 slave node；
- slave node 做复制的时候，不会 block master node 的正常工作；
- slave node 在做复制的时候，也不会 block 对自己的查询操作，它会用旧的数据集来提供服务；但是复制完成的时候，需要删除旧数据集，加载新数据集，这个时候就会暂停对外服务了；
- slave node 主要用来进行横向扩容，做读写分离，扩容的 slave node 可以提高读的吞吐量。

注意，如果采用了主从架构，那么建议必须**开启** master node 的[持久化](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/redis-persistence.md)，**不建议用 slave node 作为 master node 的数据热备，因为那样的话，如果你关掉 master 的持久化，可能在 master 宕机重启的时候数据是空的，然后可能一经过复制， slave node 的数据也丢了。**

​	另外，master 的各种备份方案，也需要做。万一本地的所有文件丢失了，从备份中挑选一份 rdb 去恢复 master，这样才能**确保启动的时候，是有数据的**，即使采用了后续讲解的[高可用机制](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/redis-sentinel.md)，slave node 可以自动接管 master node，但也可能 sentinel 还没检测到 master failure，master node 就自动重启了，还是可能导致上面所有的 slave node 数据被清空。

## redis 主从复制的核心原理

==当启动一个 slave node 的时候，它会发送一个 `PSYNC` 命令给 master node。==

​	如果这是 slave node 初次连接到 master node，那么会触发一次 `full resynchronization` 全量复制。==此时 master 会启动一个后台线程，开始生成一份 `RDB` 快照文件，同时还会将从客户端 client 新收到的所有写命令缓存在内存中。`RDB` 文件生成完毕后， master 会将这个 `RDB` 发送给 slave，slave 会先**写入本地磁盘，然后再从本地磁盘加载到内存**中，接着 master 会将内存中缓存的写命令发送到 slave，slave 也会同步这些数据。slave node 如果跟 master node 有网络故障，断开了连接，会自动重连，连接之后 master node 仅会复制给 slave 部分缺少的数据。==

[![redis-master-slave-replication](/Users/jack/Desktop/md/images/redis-master-slave-replication.png)](https://github.com/doocs/advanced-java/blob/master/images/redis-master-slave-replication.png)

### 主从复制的断点续传

​	从 redis2.8 开始，就支持主从复制的断点续传，**如果主从复制过程中，网络连接断掉了，那么可以接着上次复制的地方，继续复制下去，而不是从头开始复制一份。**

​	master node 会在内存中维护一个 backlog，master 和 slave 都会保存一个 replica offset 还有一个 master run id，offset 就是保存在 backlog 中的。如果 master 和 slave 网络连接断掉了，slave 会让 master 从上次 replica offset 开始继续复制，如果没有找到对应的 offset，那么就会执行一次 `resynchronization`。

> 如果根据 host+ip 定位 master node，是不靠谱的，如果 master node 重启或者数据出现了变化，那么 slave node 应该根据不同的 run id 区分。

### 无磁盘化复制

​	master 在内存中直接创建 `RDB`，然后发送给 slave，不会在自己本地落地磁盘了。只需要在配置文件中开启 `repl-diskless-sync yes` 即可。

```conf
repl-diskless-sync yes
# 等待 5s 后再开始复制，因为要等更多 slave 重新连接过来
repl-diskless-sync-delay 5
```

### 过期 key 处理

​	**slave 不会过期 key，只会等待 master 过期 key。如果 master 过期了一个 key，或者通过 LRU 淘汰了一个 key，那么会模拟一条 del 命令发送给 slave。**

## 复制的完整流程

​	slave node 启动时，会在自己本地保存 master node 的信息，包括 master node 的`host`和`ip`，但是复制流程没开始。

​	slave node 内部有个定时任务，每秒检查是否有新的 master node 要连接和复制，如果发现，就跟 master node 建立 socket 网络连接。然后 slave node 发送 `ping` 命令给 master node。如果 master 设置了 requirepass，那么 slave node 必须发送 masterauth 的口令过去进行认证。master node **第一次执行全量复制**，将所有数据发给 slave node。而在后续，master node 持续将写命令，异步复制给 slave node。

[![redis-master-slave-replication-detail](/Users/jack/Desktop/md/images/redis-master-slave-replication-detail.png)](https://github.com/doocs/advanced-java/blob/master/images/redis-master-slave-replication-detail.png)

### 全量复制

- master 执行 bgsave ，在本地生成一份 RDB 快照文件。
- master node 将 RDB 快照文件发送给 slave node，如果 rdb 复制时间超过 60秒（repl-timeout），那么 slave node 就会认为复制失败，可以适当调大这个参数(对于千兆网卡的机器，一般每秒传输 100MB，6G 文件，很可能超过 60s)
- master node 在生成 RDB 时，会将所有新的写命令缓存在内存中，在 slave node 保存了 rdb 之后，再将新的写命令复制给 slave node。
- 如果在复制期间，内存缓冲区持续消耗超过 64MB，或者一次性超过 256MB，那么停止复制，复制失败。

```
client-output-buffer-limit slave 256MB 64MB 60
```

- slave node 接收到 RDB 之后，清空自己的旧数据，然后重新加载 RDB 到自己的内存中，同时**基于旧的数据版本**对外提供服务。
- 如果 slave node 开启了 AOF，那么会立即执行 BGREWRITEAOF，重写 AOF。

### 增量复制

- 如果全量复制过程中，master-slave 网络连接断掉，那么 slave 重新连接 master 时，会触发增量复制。
- master 直接从自己的 backlog 中获取部分丢失的数据，发送给 slave node，默认 backlog 就是 1MB。
- master 就是根据 slave 发送的 psync 中的 offset 来从 backlog 中获取数据的。

### heartbeat

主从节点互相都会发送 heartbeat 信息。

master 默认每隔 10秒 发送一次 heartbeat，slave node 每隔 1秒 发送一个 heartbeat。

### 异步复制

master 每次接收到写命令之后，先在内部写入数据，然后异步发送给 slave node。

参照： [《Redis 主从架构》](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/redis-master-slave.md) 。

高可用可以看：

 [《Redis 哨兵集群实现高可用》](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/redis-sentinel.md) 。

- [《Redis 集群教程》](http://redis.cn/topics/cluster-tutorial.html) 完整版
- [《Redis 集群模式的工作原理能说一下么？》](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/redis-cluster.md) 精简版

## Redis 哈希槽

​	Redis Cluster 没有使用一致性 hash ，而是引入了哈希槽的概念。

Redis 集群有 16384 个哈希槽，每个 key 通过 CRC16 校验后对 16384 取模来决定放置哪个槽，集群的每个节点负责一部分 hash 槽。

因为最大是 16384 个哈希槽，所以考虑 Redis 集群中的每个节点都能分配到一个哈希槽，所以最多支持 16384 个 Redis 节点。

# 十、Redis的部署

- Redis Cluster，10 台机器，5 台机器部署了 redis 主实例，另外 5 台机器部署了 redis 的从实例，每个主实例挂了一个从实例，5 个节点对外提供读写服务，每个节点的读写高峰 qps 可能可以达到每秒 5 万，5 台机器最多是 25 万读写请求每秒。
- 机器配置：32G 内存 + 8 核 CPU + 1T 磁盘，但是分配给 Redis 进程的是 10g 内存，一般线上生产环境，Redis 的内存尽量不要超过 10g，超过 10g 可能会有问题。那么，5 台机器对外提供读写，一共有 50g 内存。
- 因为每个主实例都挂了一个从实例，所以是高可用的，任何一个主实例宕机，都会自动故障迁移，Redis 从实例会自动变成主实例继续提供读写服务。
- 你往内存里写的是什么数据？每条数据的大小是多少？商品数据，每条数据是 10kb 。100 条数据是 1mb ，10 万条数据是 1g 。常驻内存的是 200 万条商品数据，占用内存是 20g，仅仅不到总内存的 50%。目前高峰期每秒就是 3500 左右的请求量。
- 其实大型的公司，会有基础架构的 team 负责缓存集群的运维。

## Redis 的健康指标

推荐阅读 [《Redis 几个重要的健康指标》](https://mp.weixin.qq.com/s/D_khsApGkRckEoV75pYpDA)

- 存活情况
- 连接数
- 阻塞客户端数量
- 使用内存峰值
- 内存碎片率
- 缓存命中率
- OPS
- 持久化
- 失效KEY
- 慢日志

**提高 Redis 命中率**：推荐阅读 [《如何提高缓存命中率（Redis）》](http://www.cnblogs.com/shamo89/p/8383915.html) 。

## 优化 Redis 的内存占用

推荐阅读 [《Redis 的内存优化》](https://www.jianshu.com/p/8677603d3865)

- redisObject 对象
- 缩减键值对象
- 共享对象池
- 字符串优化
- 编码优化
- 控制 key 的数量

> **假如 Redis 里面有 1 亿个 key，其中有 10w 个 key 是以某个固定的已知的前缀开头的，如果将它们全部找出来？**
>
> 使用 keys 指令可以扫出指定模式的 key 列表。
>
> - 对方接着追问：如果这个 Redis 正在给线上的业务提供服务，那使用keys指令会有什么问题？
> - 这个时候你要回答 Redis 关键的一个特性：Redis 的单线程的。keys 指令会导致线程阻塞一段时间，线上服务会停顿，直到指令执行完毕，服务才能恢复。这个时候可以使用 scan 指令，scan 指令可以无阻塞的提取出指定模式的 key 列表，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整体所花费的时间会比直接用 keys 指令长。

## Redis 常见的性能问题

1、Master 最好不要做任何持久化工作，如 RDB 内存快照和 AOF 日志文件。

- Master 写内存快照，save 命令调度 rdbSave 函数，会阻塞主线程的工作，当快照比较大时对性能影响是非常大的，会间断性暂停服务，所以 Master 最好不要写内存快照。
- Master AOF 持久化，如果不重写 AOF 文件，这个持久化方式对性能的影响是最小的，但是 AOF 文件会不断增大，AOF 文件过大会影响 Master 重启的恢复速度。
- 所以，Master 最好不要做任何持久化工作，包括内存快照和 AOF 日志文件，特别是不要启用内存快照做持久化。如果数据比较关键，某个 Slave 开启AOF备份数据，策略为每秒同步一次。

2、Master 调用 BGREWRITEAOF 重写 AOF 文件，AOF 在重写的时候会占大量的 CPU 和内存资源，导致服务 load 过高，出现短暂服务暂停现象。

3、尽量避免在压力很大的主库上增加从库。

4、主从复制不要用图状结构，用单向链表结构更为稳定，即：Master <- Slave1 <- Slave2 <- Slave3...。

- 这样的结构，也方便解决单点故障问题，实现 Slave 对 Master 的替换。如果 Master挂了，可以立刻启用 Slave1 做 Master ，其他不变。

5、Redis 主从复制的性能问题，为了主从复制的速度和连接的稳定性，Slave 和 Master 最好在同一个局域网内。



参照：

- [芋道源码](http://svip.iocoder.cn/Redis/Interview/)

- JeffreyLcm [《Redis 面试题》](https://segmentfault.com/a/1190000014507534)
- 烙印99 [《史上最全 Redis 面试题及答案》](https://www.imooc.com/article/36399)
- yanglbme [《Redis 和 Memcached 有什么区别？Redis 的线程模型是什么？为什么单线程的 Redis 比多线程的 Memcached 效率要高得多？》](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/redis-single-thread-model.md)
- 老钱 [《天下无难试之 Redis 面试题刁难大全》](https://zhuanlan.zhihu.com/p/32540678)
- yanglbme [《Redis 的持久化有哪几种方式？不同的持久化机制都有什么优缺点？持久化机制具体底层是如何实现的？》](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/redis-persistence.md)