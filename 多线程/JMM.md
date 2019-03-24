# Java 内存模型

![jmm-](/Users/jack/Desktop/md/images/006tNc79gy1fow7lb5c5kj310y0z4q7w.jpg)

关于 Java 内存模型，涉及的内容会很多，所以建议胖友看如下的 [《深入Java内存模型.pdf》](https://files-cdn.cnblogs.com/files/skywang12345/%E6%B7%B1%E5%85%A5Java%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B.pdf) 这本小书。

然后，看完之后你肯定会忘记，就可以靠 [《《深入理解 Java 内存模型》读书笔记》](http://www.iocoder.cn/JUC/zhisheng/Java-Memory-Model?vip) 来补刀。

再另外，[《深入拆解 Java 虚拟机》](http://gk.link/a/100kc) 的 [「第五部分 高效并发」](http://svip.iocoder.cn/Java/Concurrent/Interview/#) 也推荐阅读。

## 1、什么是 Java 内存模型？

Java 虚拟机规范中试图定义一种 Java 内存模型（Java Memory Model，JMM）来屏蔽掉各层硬件和操作系统的内存访问差异，以实现让 Java 程序在各种平台下都能达到一致的内存访问效果。

Java 内存模型规定了所有的变量都存储在主内存（Main Memory）中。每条线程还有自己的工作内存（Working Memory），线程的工作内存中保存了被该线程使用到的变量的主内存副本拷贝，线程对变量的所有操作（读取、赋值等）都必须在主内存中进行，而不能直接读写主内存中的变量。不同的线程之间也无法直接访问对方工作内存中的变量，线程间的变量值的传递均需要通过主内存来完成，线程、主内存、工作内存三者的关系如下图：

![线程、主内存、工作内存](http://static.iocoder.cn/images/JDK/2020_02_07/01.png)

> 艿艿：当然，有个面试官会把 Java 内存模型，和 JVM 内存结构搞混淆。所以，在回答之前，可以先和面试官确认下说的是哪个。
>
> 关于 JVM 内存结构的面试题，我们在 [《精尽 Java【虚拟机】面试题》](http://svip.iocoder.cn/Java/VirtualMachine/Interview) 中在详细分享。

## 两个线程之间是如何通信的呢？

线程之间的通信方式，目前有共享内存和消息传递两种。

**1）共享内存**

在共享内存的并发模型里，线程之间共享程序的公共状态，线程之间通过写-读内存中的公共状态来隐式进行通信。典型的共享内存通信方式，就是通过共享对象进行通信。

![共享内存](http://static.iocoder.cn/images/JDK/2020_02_07/02.png)

例如上图线程 A 与 线程 B 之间如果要通信的话，那么就必须经历下面两个步骤：

1. 首先，线程 A 把本地内存 A 更新过得共享变量刷新到主内存中去。
2. 然后，线程 B 到主内存中去读取线程 A 之前更新过的共享变量。

**2）消息传递**

在消息传递的并发模型里，线程之间没有公共状态，线程之间必须通过明确的发送消息来显式进行通信。在 Java 中典型的消息传递方式，就是 `#wait()` 和 `#notify()` ，或者 BlockingQueue 。

![消息传递](http://static.iocoder.cn/images/JDK/2020_02_07/03.png)

## 为什么代码会重排序？

在执行程序时，为了提供性能，处理器和编译器常常会对指令进行重排序，但是不能随意重排序，不是你想怎么排序就怎么排序，它需要满足以下两个条件：

- 在单线程环境下不能改变程序运行的结果。
- 存在数据依赖关系的不允许重排序

**需要注意的是：重排序不会影响单线程环境的执行结果，但是会破坏多线程的执行语义**。

## 什么是内存模型的 happens-before 呢？

详细看 [《【死磕 Java 并发】—– Java 内存模型之 happens-before》](http://www.iocoder.cn/JUC/sike/happens-before/?vip) 文章。

## 什么是内存屏障？

内存屏障，又称内存栅栏，是一组处理器指令，用于实现对内存操作的顺序限制。

🦅 **内存屏障为何重要？**

对主存的一次访问一般花费硬件的数百次时钟周期。处理器通过缓存（caching）能够从数量级上降低内存延迟的成本这些缓存为了性能重新排列待定内存操作的顺序。也就是说，程序的读写操作不一定会按照它要求处理器的顺序执行。当数据是不可变的，同时/或者数据限制在线程范围内，这些优化是无害的。如果把这些优化与对称多处理（symmetric multi-processing）和共享可变状态（shared mutable state）结合，那么就是一场噩梦。

当基于共享可变状态的内存操作被重新排序时，程序可能行为不定。一个线程写入的数据可能被其他线程可见，原因是数据写入的顺序不一致。适当的放置内存屏障，通过强制处理器顺序执行待定的内存操作来避免这个问题。

# Java 并发容器

## 什么是并发容器的实现？

何为同步容器？可以简单地理解为通过 `synchronized`来实现同步的容器，如果有多个线程调用同步容器的方法，它们将会串行执行。

- 比如 Vector，Hashtable，以及 `Collections#synchronizedSet()`，`Collections#synchronizedList()` 等方法返回的容器。
- 可以通过查看 Vector，Hashtable 等这些同步容器的实现代码，可以看到这些容器实现线程安全的方式就是将它们的状态封装起来，并在需要同步的方法上加上关键字 `synchronized` 。

并发容器，使用了与同步容器完全不同的加锁策略来提供更高的并发性和伸缩性。

- 例如在 ConcurrentHashMap 中采用了一种粒度更细的加锁机制，可以称为分段锁。在这种锁机制下，允许任意数量的读线程并发地访问 map ，并且执行读操作的线程和写操作的线程也可以并发的访问 map ，同时允许一定数量的写操作线程并发地修改 map ，所以它可以在并发环境下实现更高的吞吐量。
- 再例如，CopyOnWriteArrayList 。

## SynchronizedMap 和 ConcurrentHashMap 有什么区别？

- SynchronizedMap
  - 一次锁住整张表来保证线程安全，所以每次只能有一个线程来访为 map 。
- ConcurrentHashMap
  - 使用分段锁来保证在多线程下的性能。ConcurrentHashMap 中则是一次锁住一个桶。ConcurrentHashMap 默认将 hash 表分为 16 个桶，诸如 get,put,remove 等常用操作只锁当前需要用到的桶。这样，原来只能一个线程进入，现在却能同时有 16 个写线程执行，并发性能的提升是显而易见的。【注意，这块是 JDK7 的实现。在 JDK8 中，具体的实现已经改变】
  - 另外 ConcurrentHashMap 使用了一种不同的迭代方式。在这种迭代方式中，当 iterator 被创建后集合再发生改变就不再是抛出 ConcurrentModificationException 异常，取而代之的是在改变时 `new` 新的数据从而不影响原有的数据，iterator 完成后再将头指针替换为新的数据 ，这样 iterator 线程可以使用原来老的数据，而写线程也可以并发的完成改变。

关于 ConcurrentHashMap 的源码解析，推荐胖友看看如下两篇文章：

- [《【死磕 Java 并发】—– J.U.C 之 Java并发容器：ConcurrentHashMap》](http://www.iocoder.cn/JUC/sike/ConcurrentHashMap/?vip)
- [《【死磕 Java 并发】—– J.U.C 之 ConcurrentHashMap 红黑树转换分析》](http://www.iocoder.cn/JUC/sike/ConcurrentHashMap-red-black-tree/?vip)

🦅 **Java 中 ConcurrentHashMap 的并发度是什么？**

在 JDK8 前，ConcurrentHashMap 把实际 map 划分成若干部分来实现它的可扩展性和线程安全。这种划分是使用并发度获得的，它是 ConcurrentHashMap 类构造函数的一个可选参数，默认值为 16 ，这样在多线程情况下就能避免争用。

在 JDK8 后，它摒弃了 Segment（锁段）的概念，而是启用了一种全新的方式实现，利用 CAS 算法。同时加入了更多的辅助变量来提高并发度，具体内容还是查看源码吧。

🦅 **ConcurrentHashMap 为何读不用加锁？**

在 JDK7 以及以前

- HashEntry 中的

   

  ```
  key
  ```

  、

  ```
  hash
  ```

  、

  ```
  next
  ```

   

  均为

   

  ```
  final
  ```

   

  型，只能表头插入、删除结点。

  - HashEntry 类的 `value` 域被声明为 `volatile` 型。
  - 不允许用 `null` 作为键和值，当读线程读到某个 HashEntry 的 `value` 域的值为 `null` 时，便知道产生了冲突——发生了重排序现象（put 方法设置新 `value` 对象的字节码指令重排序），需要加锁后重新读入这个 `value` 值。

- `volatile` 变量 `count` 协调读写线程之间的内存可见性，写操作后修改 `count` ，读操作先读 `count`，根据 happen-before 传递性原则写操作的修改读操作能够看到。

在 JDK8 开始

- Node 的 `val` 和 `next` 均为 `volatile` 型。
- `#tabAt(..,)` 和 `#casTabAt(...)` 对应的 Unsafe 操作实现了 `volatile` 语义。

## CopyOnWriteArrayList 可以用于什么应用场景？

CopyOnWriteArrayList(免锁容器)的好处之一是当多个迭代器同时遍历和修改这个列表时，不会抛出ConcurrentModificationException 异常。在 CopyOnWriteArrayList 中，写入将导致创建整个底层数组的副本，而源数组将保留在原地，使得复制的数组在被修改时，读取操作可以安全地执行。

- 由于写操作的时候，需要拷贝数组，会消耗内存，如果原数组的内容比较多的情况下，可能导致 ygc 或者 fgc 。
- 不能用于实时读的场景，像拷贝数组、新增元素都需要时间，所以调用一个 set 操作后，读取到数据可能还是旧的,虽然 CopyOnWriteArrayList 能做到最终一致性,但是还是没法满足实时性要求。

CopyOnWriteArrayList 透露的思想：

- 读写分离，读和写分开
- 最终一致性
- 使用另外开辟空间的思路，来解决并发冲突

CopyOnWriteArrayList 适用于读操作远远多于写操作的场景。例如，缓存。

关于 CopyOnWriteArrayList 的源码，可以看看 [《CopyOnWriteArrayList 实现原理及源码分析》](http://www.importnew.com/12773.html) 文章。

# Java 阻塞队列

## 什么是阻塞队列？有什么适用场景？

阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。这两个附加的操作是：

- 在队列为空时，获取元素的线程会等待队列变为非空。
- 当队列满时，存储元素的线程会等待队列可用。

阻塞队列常用于生产者和消费者的场景：

- 生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程
- 阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。

> 艿艿：如下的内容，和上面是相对重复的，或者是换一个说法，重新描述。

BlockingQueue 接口，是 Queue 的子接口，它的主要用途并不是作为容器，而是作为线程同步的的工具，因此他具有一个很明显的特性：

- 当生产者线程试图向 BlockingQueue 放入元素时，如果队列已满，则线程被阻塞。
- 当消费者线程试图从中取出一个元素时，如果队列为空，则该线程会被阻塞。
- 正是因为它所具有这个特性，所以在程序中多个线程交替向BlockingQueue中 放入元素，取出元素，它可以很好的控制线程之间的通信。

阻塞队列使用最经典的场景，就是 Socket 客户端数据的读取和解析：

- 读取数据的线程不断将数据放入队列。
- 然后，解析线程不断从队列取数据解析。

## Java 提供了哪些阻塞队列的实现？

JDK7 提供了 7 个阻塞队列。分别是：

> Java5 之前实现同步存取时，可以使用普通的一个集合，然后在使用线程的协作和线程同步可以实现生产者，消费者模式，主要的技术就是用好 wait、notify、notifyAll、`sychronized` 这些关键字。
>
> 而在 Java5 之后，可以使用阻塞队列来实现，此方式大大简少了代码量，使得多线程编程更加容易，安全方面也有保障。

- 【最常用】ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。

  > 此队列按照先进先出（FIFO）的原则对元素进行排序，但是默认情况下不保证线程公平的访问队列，即如果队列满了，那么被阻塞在外面的线程对队列访问的顺序是不能保证线程公平（即先阻塞，先插入）的。

- LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列。

  > 此队列按照先出先进的原则对元素进行排序

- PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。

- DelayQueue：支持延时获取元素的无界阻塞队列，即可以指定多久才能从队列中获取当前元素。

- SynchronousQueue：一个不存储元素的阻塞队列。

  > 每一个 put 必须等待一个 take 操作，否则不能继续添加元素。并且他支持公平访问队列。

- LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。

  > 相对于其他阻塞队列，多了 tryTransfer 和 transfer 方法。
  >
  > - transfer 方法：如果当前有消费者正在等待接收元素（take 或者待时间限制的 poll 方法），transfer 可以把生产者传入的元素立刻传给消费者。如果没有消费者等待接收元素，则将元素放在队列的 tail 节点，并等到该元素被消费者消费了才返回。
  > - tryTransfer 方法：用来试探生产者传入的元素能否直接传给消费者。如果没有消费者在等待，则返回 false 。和上述方法的区别是该方法无论消费者是否接收，方法立即返回。而 transfer 方法是必须等到消费者消费了才返回。

- LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

  > 优势在于多线程入队时，减少一半的竞争。

具体的源码解析，可以看看如下文章：

- [《【死磕 Java 并发】—– J.U.C 之阻塞队列：ArrayBlockingQueue》](http://www.iocoder.cn/JUC/sike/ArrayBlockingQueue?vip)
- [《【死磕 Java 并发】—– J.U.C 之阻塞队列：PriorityBlockingQueue》](http://www.iocoder.cn/JUC/sike/PriorityBlockingQueue?vip)
- [《【死磕 Java 并发】—– J.U.C 之阻塞队列：DelayQueue》](http://www.iocoder.cn/JUC/sike/DelayQueue?vip)
- [《【死磕 Java 并发】—– J.U.C 之阻塞队列：SynchronousQueue》](http://www.iocoder.cn/JUC/sike/SynchronousQueue?vip)
- [《【死磕 Java 并发】—– J.U.C 之阻塞队列：LinkedTransferQueue》](http://www.iocoder.cn/JUC/sike/LinkedTransferQueue?vip)
- [《【死磕 Java 并发】—– J.U.C 之阻塞队列：LinkedBlockingDeque》](http://www.iocoder.cn/JUC/sike/LinkedBlockingDeque?vip)
- [《【死磕 Java 并发】—– J.U.C 之阻塞队列：BlockingQueue 总结》](http://www.iocoder.cn/JUC/sike/BlockingQueue?vip)

🦅 **阻塞队列提供哪些重要方法？**

| 方法处理方式 | 抛出异常  | 返回特殊值 | 一直阻塞 | 超时退出             |
| ------------ | --------- | ---------- | -------- | -------------------- |
| 插入方法     | add(e)    | offer(e)   | put(e)   | offer(e, time, unit) |
| 移除方法     | remove()  | poll()     | take()   | poll(time, unit)     |
| 检查方法     | element() | peek()     | 不可用   | 不可用               |

🦅 **ArrayBlockingQueue 与 LinkedBlockingQueue 的区别？**

| Queue               | 阻塞与否 | 是否有界 | 线程安全保障    | 适用场景                       | 注意事项                                                     |
| ------------------- | -------- | -------- | --------------- | ------------------------------ | ------------------------------------------------------------ |
| ArrayBlockingQueue  | 阻塞     | 有界     | 一把全局锁      | 生产消费模型，平衡两边处理速度 | 用于存储队列元素的存储空间是预先分配的，使用过程中内存开销较小（无须动态申请存储空间） |
| LinkedBlockingQueue | 阻塞     | 可配置   | 存取采用 2 把锁 | 生产消费模型，平衡两边处理速度 | 无界的时候注意内存溢出问题，用于存储队列元素的存储空间是在其使用过程中动态分配的，因此它可能会增加 JVM 垃圾回收的负担。 |

感兴趣的胖友，可以看看如下两篇文章：

- [《ArrayBlockingQueue 与 LinkedBlockingQueue》](https://www.jianshu.com/p/5b85c1794351)
- [《从一个故障说说 Java 的三个 BlockingQueue》](http://hellojava.info/?p=464)

## 什么是双端队列？

在上面，我们看到的 LinkedBlockingQueue、ArrayBlockingQueue、PriorityBlockingQueue、SynchronousQueue 等，都是阻塞队列。

而 ArrayDeque、LinkedBlockingDeque 就是双端队列，类名以 Deque 结尾。

- 正如阻塞队列适用于生产者消费者模式，双端队列同样适用与另一种模式，即

  工作密取

  。在生产者-消费者设计中，所有消费者共享一个工作队列，而在工作密取中，每个消费者都有各自的双端队列。

  - 如果一个消费者完成了自己双端队列中的全部工作，那么他就可以从其他消费者的双端队列末尾秘密的获取工作。具有更好的可伸缩性，这是因为工作者线程不会在单个共享的任务队列上发生竞争。
  - 在大多数时候，他们都只是访问自己的双端队列，从而极大的减少了竞争。当工作者线程需要访问另一个队列时，它会从队列的尾部而不是头部获取工作，因此进一步降低了队列上的竞争。

- 适用于：网页爬虫等任务中

> 😈 实际场景下，双端队列，我们使用比较少。艿艿根本没用过。

## 延迟队列的实现方式，DelayQueue 和时间轮算法的异同？

JDK 的 Timer 和 DelayQueue 插入和删除操作的平均时间复杂度为 `O(nlog(n))` ，而基于时间轮可以将插入和删除操作的时间复杂度都降为 `O(1)` 。

- 关于 DelayQueue 的实现方式，在 [《【死磕 Java 并发】—– J.U.C 之阻塞队列：DelayQueue》》](http://www.iocoder.cn/JUC/sike/DelayQueue?vip) 。
- 关于实践论的实现方法，在 [《Kafka解惑之时间轮（TimingWheel）》](https://blog.csdn.net/u013256816/article/details/80697456) 。

## 简述 ConcurrentLinkedQueue 和 LinkedBlockingQueue 的用处和不同之处？

参考 [《LinkedBlockingQueue 和 ConcurrentLinkedQueue的用法及区别》](https://www.jianshu.com/p/1a49293294aa) 。

在 Java 多线程应用中，队列的使用率很高，多数生产消费模型的首选数据结构就是队列(先进先出)。

Java 提供的线程安全的 Queue 可以分为

- 阻塞队列，典型例子是 LinkedBlockingQueue 。

  > 适用阻塞队列的好处：多线程操作共同的队列时不需要额外的同步，另外就是队列会自动平衡负载，即那边（生产与消费两边）处理快了就会被阻塞掉，从而减少两边的处理速度差距。

- 非阻塞队列，典型例子是 ConcurrentLinkedQueue 。

  > 当许多线程共享访问一个公共集合时，`ConcurrentLinkedQueue` 是一个恰当的选择。

具体的选择，如下：

- LinkedBlockingQueue 多用于任务队列。
  - 单生产者，单消费者
  - 多生产者，单消费者
- ConcurrentLinkedQueue 多用于消息队列。
  - 单生产者，多消费者
  - 多生产者，多消费者

# Java 原子操作类

## 什么是原子操作？

原子操作（Atomic Operation），意为”不可被中断的一个或一系列操作”。

- 处理器使用基于对缓存加锁或总线加锁的方式，来实现多处理器之间的原子操作。
- 在 Java 中，可以通过锁和循环 CAS 的方式来实现原子操作。CAS操作 —— Compare & Set ，或是 Compare & Swap ，现在几乎所有的 CPU 指令都支持 CAS 的原子操作。

原子操作是指一个不受其他操作影响的操作任务单元。原子操作是在多线程环境下避免数据不一致必须的手段。

- `int++` 并不是一个原子操作，所以当一个线程读取它的值并加 1 时，另外一个线程有可能会读到之前的值，这就会引发错误。
- 为了解决这个问题，必须保证增加操作是原子的，在 JDK5 之前我们可以使用同步技术来做到这一点。到 JDK5 后，`java.util.concurrent.atomic` 包提供了 `int` 和 `long` 类型的原子包装类，它们可以自动的保证对于他们的操作是原子的并且不需要使用同步。

`java.util.concurrent` 这个包里面提供了一组原子类。其基本的特性就是在多线程环境下，当有多个线程同时执行这些类的实例包含的方法时，具有排他性，即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，才由 JVM 从等待队列中选择一个另一个线程进入，这只是一种逻辑上的理解。

- 原子类：AtomicBoolean，AtomicInteger，AtomicLong，AtomicReference 。
- 原子数组：AtomicIntegerArray，AtomicLongArray，AtomicReferenceArray 。
- 原子属性更新器：AtomicLongFieldUpdater，AtomicIntegerFieldUpdater，AtomicReferenceFieldUpdater 。
- 解决 ABA 问题的原子类：AtomicMarkableReference（通过引入一个`boolean` 来反映中间有没有变过），AtomicStampedReference（通过引入一个 `int` 来累加来反映中间有没有变过）。

关于 CAS 的内容，建议胖友在看看 [《【死磕 Java 并发】—- 深入分析 CAS》](http://www.iocoder.cn/JUC/sike/CAS/) 。

## CAS 操作有什么缺点？

1）**ABA 问题**

比如说一个线程 one 从内存位置 V 中取出 A ，这时候另一个线程 two 也从内存中取出 A ，并且 two 进行了一些操作变成了 B ，然后 two 又将 V 位置的数据变成 A ，这时候线程 one 进行 CAS 操作发现内存中仍然是 A ，然后 one 操作成功。尽管线程 one 的 CAS 操作成功，但可能存在潜藏的问题。

从 Java5 开始 JDK 的 `atomic`包里提供了一个类 AtomicStampedReference 来解决 ABA 问题。

2）**循环时间长开销大**

对于资源竞争严重（线程冲突严重）的情况，CAS 自旋的概率会比较大，从而浪费更多的 CPU 资源，效率低于 `synchronized`。

3）**只能保证一个共享变量的原子操作**

当对一个共享变量执行操作时，我们可以使用循环 CAS 的方式来保证原子操作，但是对多个共享变量操作时，循环 CAS 就无法保证操作的原子性，这个时候就可以用锁。

# Java 并发工具类

## Semaphore 是什么？

Semaphore ，是一种新的同步类，它是一个计数信号。从概念上讲，从概念上讲，信号量维护了一个许可集合。

- 如有必要，在许可可用前会阻塞每一个 `#acquire()` 方法，然后再获取该许可。
- 每个 `#release()` 方法，添加一个许可，从而可能释放一个正在阻塞的获取者。
- 但是，不使用实际的许可对象，Semaphore 只对可用许可的数量进行计数，并采取相应的行动。

信号量常常用于多线程的代码中，比如数据库连接池。

- 使用方式，可以看看 [《JAVA多线程 – 信号量(Semaphore)》](https://my.oschina.net/cloudcoder/blog/362974) 。
- 源码解析，可以看看 [《【死磕 Java 并发】—– J.U.C 之并发工具类：Semaphore》](http://www.iocoder.cn/JUC/sike/Semaphore/) 。

## 说说 CountDownLatch 原理

CountDownLatch ，字面意思是减小计数（CountDown）的门闩（Latch）。它要做的事情是，等待指定数量的计数被减少，意味着门闩被打开，然后进行执行。

CountDownLatch 默认的构造方法是 `CountDownLatch(int count)` ，其参数表示需要减少的计数，主线程调用 `#await()`方法告诉 CountDownLatch 阻塞等待指定数量的计数被减少，然后其它线程调用 CountDownLatch 的 `#countDown()` 方法，减小计数(不会阻塞)。等待计数被减少到零，主线程结束阻塞等待，继续往下执行。

- CountDownLatch 的使用示例，请看 [《Java 多线程 CountDownLatch 用法》](https://zk1878.iteye.com/blog/1002652) 。
- CountDownLatch 的源码解析，请看 [《【死磕 Java 并发】—– J.U.C 之并发工具类：CountDownLatch》](http://www.iocoder.cn/JUC/sike/CountDownLatch/?vip)

## 说说 CyclicBarrier 原理

CyclicBarrier ，字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。

CyclicBarrier 默认的构造方法是 `CyclicBarrier(int parties)` ，其参数表示屏障拦截的线程数量，每个线程调用 `#await()` 方法告诉 CyclicBarrier 我已经到达了屏障，然后当前线程被阻塞，直到 `parties` 个线程到达，结束阻塞。

- CyclicBarrier 的使用示例，请看 [《CyclicBarrier 的用法》](https://www.cnblogs.com/liuling/p/2013-8-21-01.html)
- CyclicBarrier 的源码解析，请看 [《【死磕 Java 并发】—- J.U.C 之并发工具类：CyclicBarrier》](http://www.iocoder.cn/JUC/sike/CyclicBarrier/?vip) 。

## 说说 Exchanger 原理

实际场景下，问了一圈朋友，Exchanger 基本没在业务中使用过。

- Exchanger 的使用示例，请看 [《【Java并发】线程同步工具Exchanger的使用》](https://blog.csdn.net/eson_15/article/details/51581842) 。
- Exchanger 的源码解析，请看 [《【死磕 Java 并发】—– J.U.C 之并发工具类：Exchanger》](http://www.iocoder.cn/JUC/sike/Exchanger/)

## CyclicBarrier 和 CountdownLatch 有什么区别？

CyclicBarrier 可以重复使用，而 CountdownLatch 不能重复使用。

- CountDownLatch 其实可以把它看作一个计数器，只不过这个计数器的操作是原子操作。
  - 你可以向 CountDownLatch 对象设置一个初始的数字作为计数值，任何调用这个对象上的 `#await()` 方法都会阻塞，直到这个计数器的计数值被其他的线程减为 0 为止。所以在当前计数到达零之前，await 方法会一直受阻塞。之后，会释放所有等待的线程，await 的所有后续调用都将立即返回。这种现象只出现一次——计数无法被重置。如果需要重置计数，请考虑使用 CyclicBarrier 。
  - CountDownLatch 的一个非常典型的应用场景是：有一个任务想要往下执行，但必须要等到其他的任务执行完毕后才可以继续往下执行。假如我们这个想要继续往下执行的任务调用一个 CountDownLatch 对象的 `#await()` 方法，其他的任务执行完自己的任务后调用同一个 CountDownLatch 对象上的 `#countDown()` 方法，这个调用 `#await()` 方法的任务将一直阻塞等待，直到这个 CountDownLatch 对象的计数值减到 0 为止。
- CyclicBarrier 一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 在释放等待线程后可以重用，所以称它为循环的 barrier 。

整理表格如下：

| CountDownLatch                                               | CyclicBarrier                                                |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 减计数方式                                                   | 加计数方式                                                   |
| 计算为 0 时释放所有等待的线程                                | 计数达到指定值时释放所有等待线程                             |
| 计数为 0 时，无法重置                                        | 计数达到指定值时，计数置为 0 重新开始                        |
| 调用 `#countDown()` 方法计数减一，调用 `#await()` 方法只进行阻塞，对计数没任何影响 | 调用 `#await()` 方法计数加 1 ，若加 1 后的值不等于构造方法的值，则线程阻塞 |
| 不可重复利用                                                 | 可重复利用                                                   |

# Java 线程池

![线程池](http://static.iocoder.cn/346a78f2c213423bcce456102006f4b3)

## 什么是 Executor 框架？

Executor 框架，是一个根据一组执行策略调用，调度，执行和控制的异步任务的框架。

无限制的创建线程，会引起应用程序内存溢出。所以创建一个线程池是个更好的的解决方案，因为可以限制线程的数量并且可以回收再利用这些线程。利用 Executor 框架，可以非常方便的创建一个线程池。

🦅 **为什么使用 Executor 框架？**

1. 每次执行任务创建线程 `new Thread()` 比较消耗性能，创建一个线程是比较耗时、耗资源的。
2. 调用 `new Thread()` 创建的线程缺乏管理，被称为野线程，而且可以无限制的创建，线程之间的相互竞争会导致过多占用系统资源而导致系统瘫痪，还有线程之间的频繁交替也会消耗很多系统资源。
3. 接使用 `new Thread()` 启动的线程不利于扩展，比如定时执行、定期执行、定时定期执行、线程中断等都不便实现。

🦅 **在 Java 中 Executor 和 Executors 的区别？**

- Executors 是 Executor 的工具类，不同方法按照我们的需求创建了不同的线程池，来满足业务的需求。
- Executor 接口对象，能执行我们的线程任务。
  - ExecutorService 接口，继承了 Executor 接口，并进行了扩展，提供了更多的方法我们能获得任务执行的状态并且可以获取任务的返回值。
    - 使用 ThreadPoolExecutor ，可以创建自定义线程池。
  - Future 表示异步计算的结果，他提供了检查计算是否完成的方法，以等待计算的完成，并可以使用 `#get()` 方法，获取计算的结果。

## 讲讲线程池的实现原理

- [《Java并发编程：线程池的使用》](http://www.cnblogs.com/dolphin0520/p/3932921.html)
- [《【死磕 Java 并发】—– J.U.C 之线程池：线程池的基础架构》](http://www.iocoder.cn/JUC/sike/ThreadPool-core/?vip)
- [《【死磕 Java 并发】—– J.U.C 之线程池：ThreadPoolExecutor》](http://www.iocoder.cn/JUC/sike/ThreadPoolExecutor/?vip)
- [《【死磕 Java 并发】—– J.U.C 之线程池：ScheduledThreadPoolExecutor》](http://www.iocoder.cn/JUC/sike/ScheduledThreadPoolExecutor/?vip)

## 创建线程池的几种方式？

Java 类库提供一个灵活的线程池以及一些有用的默认配置，我们可以通过Executors 的静态方法来创建线程池。

> Executors 创建的线程池，分成普通任务线程池，和定时任务线程池。

- 普通任务线程池

  - 1、

    ```
    #newFixedThreadPool(int nThreads)
    ```

     

    方法，创建一个固定长度的线程池。

    - 每当提交一个任务就创建一个线程，直到达到线程池的最大数量，这时线程规模将不再变化。
    - 当线程发生未预期的错误而结束时，线程池会补充一个新的线程。

  - 2、

    ```
    #newCachedThreadPool()
    ```

     

    方法，创建一个可缓存的线程池。

    - 如果线程池的规模超过了处理需求，将自动回收空闲线程。
    - 当需求增加时，则可以自动添加新线程。线程池的规模不存在任何限制。

  - 3、

    ```
    #newSingleThreadExecutor()
    ```

     

    方法，创建一个单线程的线程池。

    - 它创建单个工作线程来执行任务，如果这个线程异常结束，会创建一个新的来替代它。
    - 它的特点是，能确保依照任务在队列中的顺序来串行执行。

- 定时任务线程池

  - 4、`#newScheduledThreadPool(int corePoolSize)` 方法，创建了一个固定长度的线程池，而且以延迟或定时的方式来执行任务，类似 Timer 。
  - 5、`#newSingleThreadExecutor()` 方法，创建了一个固定长度为 1 的线程池，而且以延迟或定时的方式来执行任务，类似 Timer 。

🦅 **如何使用 ThreadPoolExecutor 创建线程池？**

Executors 提供了创建线程池的常用模板，实际场景下，我们可能需要自动以更灵活的线程池，此时就需要使用 ThreadPoolExecutor 类。

```
// ThreadPoolExecutor.java

public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

- `corePoolSize` 参数，核心线程数大小，当线程数 < corePoolSize ，会创建线程执行任务。

- ```
  maximumPoolSize
  ```

   

  参数，最大线程数， 当线程数 >= corePoolSize 的时候，会把任务放入

   

  ```
  workQueue
  ```

   

  队列中。

  - `keepAliveTime` 参数，保持存活时间，当线程数大于 `corePoolSize` 的空闲线程能保持的最大时间。
  - `unit` 参数，时间单位。

- ```
  workQueue
  ```

   

  参数，保存任务的阻塞队列。

  - `handler` 参数，超过阻塞队列的大小时，使用的拒绝策略。

- `threadFactory` 参数，创建线程的工厂。

🦅 **ThreadPoolExecutor 有哪些拒绝策略？**

ThreadPoolExecutor 默认有四个拒绝策略：

- `ThreadPoolExecutor.AbortPolicy()` ，直接抛出异常 RejectedExecutionException 。
- `ThreadPoolExecutor.CallerRunsPolicy()` ，直接调用 run 方法并且阻塞执行。
- `ThreadPoolExecutor.DiscardPolicy()` ，直接丢弃后来的任务。
- `ThreadPoolExecutor.DiscardOldestPolicy()` ，丢弃在队列中队首的任务。

如果我们有需要，可以自己实现 RejectedExecutionHandler 接口，实现自定义的拒绝逻辑。当然，绝大多数是不需要的。

🦅 ****

## 线程池的关闭方式有几种？

ThreadPoolExecutor 提供了两个方法，用于线程池的关闭，分别是：

- `#shutdown()` 方法，不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务。
- `#shutdownNow()` 方法，立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务。

实际场景下，一般会结合这两个方法，一起实现线程池的优雅关闭。示例代码如下：

```
void shutdownAndAwaitTermination(ExecutorService pool) {
  pool.shutdown(); // Disable new tasks from being submitted
  try {
    // Wait a while for existing tasks to terminate
    if (!pool.awaitTermination(60, TimeUnit.SECONDS)) {
      pool.shutdownNow(); // Cancel currently executing tasks
      // Wait a while for tasks to respond to being cancelled
      if (!pool.awaitTermination(60, TimeUnit.SECONDS))
          System.err.println("Pool did not terminate");
      }
    }
  } catch (InterruptedException ie) {
    // (Re-)Cancel if current thread also interrupted
    pool.shutdownNow();
    // Preserve interrupt status
    Thread.currentThread().interrupt();
  }
}
```

## Java 线程池大小为何会大多被设置成 CPU 核心数 +1 ？

详细的可以看看 [《如何合理地估算线程池大小？》](http://ifeve.com/how-to-calculate-threadpool-size/) 。如下是简单的总结和整理：

一般说来，大家认为线程池的大小经验值应该这样设置：（其中 N 为CPU的个数）

- 如果是 CPU 密集型应用，则线程池大小设置为 N+1

  > 因为 CPU 密集型任务使得 CPU 使用率很高，若开过多的线程数，只能增加上下文切换的次数，因此会带来额外的开销。

- 如果是 IO 密集型应用，则线程池大小设置为 2N+1

  > IO密 集型任务 CPU 使用率并不高，因此可以让 CPU 在等待 IO 的时候去处理别的任务，充分利用 CPU 时间。

- 如果是混合型应用，那么分别创建线程池

  > 可以将任务分成 IO 密集型和 CPU 密集型任务，然后分别用不同的线程池去处理。 只要分完之后两个任务的执行时间相差不大，那么就会比串行执行来的高效。
  >
  > 因为如果划分之后两个任务执行时间相差甚远，那么先执行完的任务就要等后执行完的任务，最终的时间仍然取决于后执行完的任务，而且还要加上任务拆分与合并的开销，得不偿失。

如果一台服务器上只部署这一个应用并且只有这一个线程池，那么这种估算或许合理，具体还需自行测试验证。

但是，IO 优化中，这样的估算公式可能更适合：最佳线程数目 = （（线程等待时间 + 线程 CPU 时间）/ 线程 CPU 时间 ）* CPU 数目
**因为很显然，线程等待时间所占比例越高，需要越多线程。线程CPU时间所占比例越高，需要越少线程。**

下面举个例子：比如平均每个线程 CPU 运行时间为 0.5s ，而线程等待时间（非 CPU 运行时间，比如 IO）为 1.5s ，CPU 核心数为 8 。
那么根据上面这个公式估算得到：`((0.5 + 1.5) / 0.5) * 8 = 32`。这个公式进一步转化为：最佳线程数目 = （线程等待时间与线程 CPU 时间之比 + 1）* CPU数目。

🦅 **线程池容量的动态调整？**

ThreadPoolExecutor 提供了动态调整线程池容量大小的方法：

- setCorePoolSize：设置核心池大小。
- setMaximumPoolSize：设置线程池最大能创建的线程数目大小。

当上述参数从小变大时，ThreadPoolExecutor 进行线程赋值，还可能立即创建新的线程来执行任务。

## 什么是 Callable、Future、FutureTask ？

1）**Callable**

Callable 接口，类似于 Runnable ，从名字就可以看出来了，但是Runnable 不会返回结果，并且无法抛出返回结果的异常，而 Callable 功能更强大一些，被线程执行后，可以返回值，这个返回值可以被 Future 拿到，也就是说，Future 可以拿到异步执行任务的返回值。

> 简单来说，可以认为是带有回调的 Runnable 。

2）**Future**

Future 接口，表示异步任务，是还没有完成的任务给出的未来结果。所以说 Callable 用于产生结果，Future 用于获取结果。

3）**FutureTask**

在 Java 并发程序中，FutureTask 表示一个可以取消的异步运算。

- 它有启动和取消运算、查询运算是否完成和取回运算结果等方法。只有当运算完成的时候结果才能取回，如果运算尚未完成 get 方法将会阻塞。
- 一个 FutureTask 对象，可以对调用了 Callable 和 Runnable 的对象进行包装，由于 FutureTask 也是继承了 Runnable 接口，所以它可以提交给 Executor 来执行。

## 线程池执行任务的过程？

刚创建时，里面没有线程调用 execute() 方法，添加任务时：

1. 如果正在运行的线程数量小于核心参数

    

   ```
   corePoolSize
   ```

    

   ，继续创建线程运行这个任务

   - 否则，如果正在运行的线程数量大于或等于 `corePoolSize` ，将任务加入到阻塞队列中。
   - 否则，如果队列已满，同时正在运行的线程数量小于核心参数 `maximumPoolSize` ，继续创建线程运行这个任务。
   - 否则，如果队列已满，同时正在运行的线程数量大于或等于 `maximumPoolSize` ，根据设置的拒绝策略处理。

2. 完成一个任务，继续取下一个任务处理。

   - 没有任务继续处理，线程被中断或者线程池被关闭时，线程退出执行，如果线程池被关闭，线程结束。
   - 否则，判断线程池正在运行的线程数量是否大于核心线程数，如果是，线程结束，否则线程阻塞。因此线程池任务全部执行完成后，继续留存的线程池大小为 `corePoolSize` 。

🦅 **线程池中 submit 和 execute 方法有什么区别？**

两个方法都可以向线程池提交任务。

- `#execute(...)` 方法，返回类型是 `void` ，它定义在 Executor 接口中。
- `#submit(...)` 方法，可以返回持有计算结果的 Future 对象，它定义在 ExecutorService 接口中，它扩展了 Executor 接口，其它线程池类像 ThreadPoolExecutor 和 ScheduledThreadPoolExecutor 都有这些方法。

🦅 **如果你提交任务时，线程池队列已满，这时会发生什么？**

> 艿艿：重点在于线程池的队列是有界还是无界的。

- 如果你使用的 LinkedBlockingQueue，也就是无界队列的话，没关系，继续添加任务到阻塞队列中等待执行，因为 LinkedBlockingQueue 可以近乎认为是一个无穷大的队列，可以无限存放任务。
- 如果你使用的是有界队列比方说 ArrayBlockingQueue 的话，任务首先会被添加到 ArrayBlockingQueue 中，ArrayBlockingQueue满了，则会使用拒绝策略 RejectedExecutionHandler 处理满了的任务，默认是 AbortPolicy 。

## Fork/Join 框架是什么？

> 艿艿：这是，可能了解的人不多，我也是。大体知道就好。

Oracle 的官方给出的定义是：Fork/Join 框架是一个实现了 ExecutorService接口 的多线程处理器。它可以把一个大的任务划分为若干个小的任务并发执行，充分利用可用的资源，进而提高应用的执行效率。

我们再通过 Fork 和 Join 这两个单词来理解下 Fork/Join 框架。

- Fork 就是把一个大任务切分为若干子任务并行的执行，Join 就是合并这些子任务的执行结果，最后得到这个大任务的结果。
- 比如计算 `1+2+...＋10000` ，可以分割成 10 个子任务，每个子任务分别对 1000 个数进行求和，最终汇总这 10 个子任务的结果。

感兴趣的胖友，可以看看如下文章：

- [《JDK 7 中的 Fork/Join 模式》](https://www.ibm.com/developerworks/cn/java/j-lo-forkjoin/index.html)
- [《聊聊并发（八） —— Fork/Join 框架介绍》](https://www.infoq.cn/article/fork-join-introduction)

## 如何让一段程序并发的执行，并最终汇总结果？

- 1、CountDownLatch：允许一个或者多个线程等待前面的一个或多个线程完成，构造一个 CountDownLatch 时指定需要 countDown 的点的数量，每完成一点就 countDown 一下。当所有点都完成，CountDownLatch 的 `#await()` 就解除阻塞。

- 2、CyclicBarrier：可循环使用的 Barrier ，它的作用是让一组线程到达一个 Barrier 后阻塞，直到所有线程都到达 Barrier 后才能继续执行。

  > CountDownLatch 的计数值只能使用一次，CyclicBarrier 可以通过使用 reset 重置，还可以指定到达栅栏后优先执行的任务。

- 3、Fork/Join 框架，fork 把大任务分解成多个小任务，然后汇总多个小任务的结果得到最终结果。使用一个双端队列，当线程空闲时从双端队列的另一端领取任务。