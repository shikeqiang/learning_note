- `synchronized`
  - ![synchronized-1](/Users/jack/Desktop/md/images/b9bc9653929e5da43d8edad6e6a0d293-20190306111143172.jpeg)
  - ![synchronized-2导图](http://static.iocoder.cn/a79be5f48c26abb905348e43a1732d55)
- `volatile`
  - ![volatile](/Users/jack/Desktop/md/images/506052a856416414e18c7ed79d43cc5c.jpeg)

# 一、`synchronized` 的原理是什么?

`synchronized`是 Java 内置的关键字，它提供了一种独占的加锁方式。

- `synchronized`的获取和释放锁由JVM实现，用户不需要显示的释放锁，非常方便。
- 然而，synchronized也有一定的局限性。
  - 当线程尝试获取锁的时候，如果获取不到锁会一直阻塞。
  - 如果获取锁的线程进入休眠或者阻塞，除非当前线程异常，否则其他线程尝试获取锁必须一直等待。

关于原理，直接阅读 [《【死磕 Java 并发】—– 深入分析 synchronized 的实现原理》](http://www.iocoder.cn/JUC/sike/synchronized/?vip) 文章

## 1. 实现原理

> `synchronized` 可以保证方法或者代码块在运行时，同一时刻只有一个方法可以进入到临界区，同时它还可以保证共享变量的内存可见性。

Java 中每一个对象都可以作为锁，这是 `synchronized` 实现同步的基础：

1. 普通同步方法，锁是当前实例对象
2. 静态同步方法，锁是当前类的 class 对象
3. 同步方法块，锁是括号里面的对象

**同步代码块**：==`monitorenter` 指令插入到同步代码块的开始位置，`monitorexit` 指令插入到同步代码块的结束位置，JVM 需要保证每一个 `monitorenter` 都有一个 `monitorexit` 与之相对应。==任何对象都有一个 Monitor 与之相关联，当且一个 Monitor 被持有之后，他将处于锁定状态。线程执行到 `monitorenter` 指令时，将会尝试获取对象所对应的 Monitor 所有权，即尝试获取对象的锁。

## 2. Java 对象头、Monitor

### 2.1 Java对象头

`synchronized` 用的锁是存在Java对象头里的。**那么什么是 Java 对象头呢**？Hotspot 虚拟机的对象头主要包括两部分数据：Mark Word（标记字段）、Klass Pointer（类型指针）。其中：

- **Klass Point 是是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。**

Mark Word 用于存储对象自身的运行时数据，如哈希码（HashCode）、GC 分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等等。Java 对象头一般占有两个机器码（在 32 位虚拟机中，1 个机器码等于 4 字节，也就是 32 bits）。但是如果对象是数组类型，则需要三个机器码，因为 JVM 虚拟机可以通过 Java 对象的元数据信息确定 Java 对象的大小，无法从数组的元数据来确认数组的大小，所以用一块来记录数组长度。

下图是 Java 对象头的存储结构（32位虚拟机）：

![存储结构](/Users/jack/Desktop/md/images/201812081002.png)

对象头信息是与对象自身定义的数据无关的额外存储成本，但是考虑到虚拟机的空间效率，Mark Word 被设计成一个**非固定**的数据结构以便在极小的空间内存存储尽量多的数据，它会根据对象的状态复用自己的存储空间，也就是说，Mark Word 会随着程序的运行发生变化，变化状态如下：

- 32 位虚拟机：

  ![32 位虚拟机](/Users/jack/Desktop/md/images/201812081003.png)

  - 每一行，是一种情况。

- 64 位虚拟机：

  ![img](/Users/jack/Desktop/md/images/1.jpeg)

  - 对于 32 位无锁状态，有 25 bits 没有使用。

### 2.2 Monitor

> FROM [《Java 8 并发篇 - 冷静分析 Synchronized（下）》](https://juejin.im/post/5abc9de851882555770c8c72)
>
> - **互斥**： 一个 Monitor 锁在同一时刻只能被一个线程占用，其他线程无法占用。
> - **信号机制( signal )**： 占用 Monitor 锁失败的线程会暂时放弃竞争并等待某个谓词成真（条件变量），但该条件成立后，当前线程会通过释放锁通知正在等待这个条件变量的其他线程，让其可以重新竞争锁。

​	监视器和锁在 Java 虚拟机中是一块使用的。监视器监视一块同步代码块，确保一次只有一个线程执行同步代码块。每一个监视器都和一个对象引用相关联。**线程在获取锁之前不允许执行同步代码**。

> FROM 《Java并发编程的艺术》的 [「2.2 synchronized 的实现原理与引用」](http://www.iocoder.cn/JUC/sike/synchronized/?vip#) 章节。
>
> Monitor Record 是线程**私有**的数据结构，每一个线程都有一个可用 Monitor Record 列表，同时还有一个全局的可用列表。
> 每一个被锁住的对象都会和一个 Monitor Record 关联（对象头的 MarkWord 中的 LockWord 指向 Monitor 的起始地址），Monitor Record 中有一个 Owner 字段，存放拥有该锁的线程的唯一标识，表示该锁被这个线程占用。其结构如下：

> ![Monitor Record](/Users/jack/Desktop/md/images/201812081004.png)

> - **Owner**：1）初始时为 NULL 表示当前没有任何线程拥有该 Monitor Record；2）当线程成功拥有该锁后保存线程唯一标识；3）当锁被释放时又设置为 NULL 。
> - **EntryQ**：关联一个系统互斥锁（ semaphore ），阻塞所有试图锁住 Monitor Record失败的线程 。
> - **RcThis**：表示 blocked 或 waiting 在该 Monitor Record 上的所有线程的个数。
> - **Nest**：用来实现重入锁的计数。
> - **HashCode**：保存从对象头拷贝过来的 HashCode 值（可能还包含 GC age ）。
> - **Candidate**：用来避免不必要的阻塞或等待线程唤醒。因为每一次只有一个线程能够成功拥有锁，如果每次前一个释放锁的线程唤醒所有正在阻塞或等待的线程，会引起不必要的上下文切换（从阻塞到就绪然后因为竞争锁失败又被阻塞）从而导致性能严重下降。Candidate 只有两种可能的值 ：1）0 表示没有需要唤醒的线程；2）1 表示要唤醒一个继任线程来竞争锁。

## 3. 锁优化

> FROM [《JVM 内部细节之一：synchronized 关键字及实现细节（轻量级锁Lightweight Locking）》](https://www.cnblogs.com/javaminer/p/3889023.html)
>
> 简单来说，在 JVM 中 `monitorenter` 和 `monitorexit` 字节码依赖于底层的操作系统的Mutex Lock 来实现的，但是由于使用 Mutex Lock 需要将当前线程挂起并从用户态切换到内核态来执行，这种切换的代价是非常昂贵的。然而，在现实中的大部分情况下，同步方法是运行在单线程环境（**无锁竞争环境**），如果每次都调用 Mutex Lock 那么将严重的影响程序的性能。

因此，JDK 1.6 对锁的实现引入了大量的优化，如**自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁**等技术来减少锁操作的开销。

### 3.1 自旋锁

**由来**

线程的阻塞和唤醒，**需要 CPU 从用户态转为核心态**。频繁的阻塞和唤醒对 CPU 来说是一件负担很重的工作，势必会给系统的并发性能带来很大的压力。同时，我们发现在许多应用上面，**对象锁的锁状态只会持续很短一段时间**。为了这一段很短的时间，频繁地阻塞和唤醒线程是非常不值得的。所以引入自旋锁。

**定义**

所谓自旋锁，就是让该线程等待一段时间，不会被立即挂起，看持有锁的线程是否会很快释放锁。

怎么等待呢？**执行一段无意义的循环即可**（自旋）。

​	自旋等待不能替代阻塞，先不说对处理器数量的要求（多核，貌似现在没有单核的处理器了），虽然它可以避免线程切换带来的开销，但是它占用了处理器的时间。**如果持有锁的线程很快就释放了锁，那么自旋的效率就非常好，反之，自旋的线程就会白白消耗掉处理的资源，它不会做任何有意义的工作，典型的占着茅坑不拉屎，这样反而会带来性能上的浪费。**

所以说，自旋等待的时间（自旋的**次数**）必须要有一个限度，如果自旋超过了定义的时间仍然没有获取到锁，则应该被挂起。

​	自旋锁在 JDK 1.4.2 中引入，默认关闭，但是可以使用 `-XX:+UseSpinning` 开开启。在 JDK1.6 中默认开启。同时自旋的默认次数为 10 次，可以通过参数 `-XX:PreBlockSpin` 来调整。

​	如果通过参数 `-XX:PreBlockSpin` 来调整自旋锁的自旋次数，会带来诸多不便。假如我将参数调整为 10 ，但是系统很多线程都是等你刚刚退出的时候，就释放了锁（假如你多自旋一两次就可以获取锁），你是不是很尴尬。于是 JDK 1.6 引入自适应的自旋锁，让虚拟机会变得越来越聪明。

#### 3.1.1 适应自旋锁

JDK 1.6 引入了更加聪明的自旋锁，即自适应自旋锁。

所谓自适应就意味着自旋的次数**不再是固定**的，**它是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定**。(根据上一次自旋次数来决定下一次自旋的次数，如果上次成功了，那么下一次很可能也会成功，所以次数可以多点。)

- 线程如果自旋成功了，那么下次自旋的次数会更加多，因为虚拟机认为既然上次成功了，那么此次自旋也很有可能会再次成功，那么它就会允许自旋等待持续的次数更多。
- 反之，如果对于某个锁，很少有自旋能够成功的，那么在以后要或者这个锁的时候自旋的次数会减少甚至省略掉自旋过程，以免浪费处理器资源。

有了自适应自旋锁，随着程序运行和性能监控信息的不断完善，虚拟机对程序锁的状况预测会越来越准确，虚拟机会变得越来越聪明。

### 3.2 锁消除

**由来**

​	为了保证数据的完整性，我们在进行操作时需要对这部分操作进行同步控制。但是，在有些情况下，**JVM检测到不可能存在共享数据竞争，这是JVM会对这些同步锁进行锁消除。**如果不存在竞争，为什么还需要加锁呢？所以锁消除可以节省毫无意义的请求锁的时间。

**定义**

​	锁消除的依据是**逃逸分析的数据支持**。变量是否逃逸，对于虚拟机来说需要使用数据流分析来确定，但是对于我们程序员来说这还不清楚么？我们会在明明知道不存在数据竞争的代码块前加上同步吗？但是有时候程序并不是我们所想的那样？我们虽然没有显示使用锁，但是我们在使用一些 JDK 的内置 API 时，如 StringBuffer、Vector、HashTable 等，这个时候会存在**隐性的加锁操作**。比如 StringBuffer 的 `append(..)`方法，Vector 的 `add(...)` 方法：

```java
public void vectorTest(){
    Vector<String> vector = new Vector<String>();
    for (int i = 0 ; i < 10 ; i++){
    	vector.add(i + "");
    }
    System.out.println(vector);
}
```

在运行这段代码时，JVM 可以明显检测到变量 `vector` 没有逃逸出方法 `#vectorTest()` 之外，所以 JVM 可以大胆地将 `vector` 内部的加锁操作消除。

## 3.3 锁粗化

**由来**

​	我们知道在使用同步锁的时候，需要**让同步块的作用范围尽可能小：仅在共享数据的实际作用域中才进行同步。**这样做的目的，是为了使需要同步的操作数量尽可能缩小，如果存在锁竞争，那么等待锁的线程也能尽快拿到锁。在大多数的情况下，这个观点是正确的，但是如果一系列的连续加锁解锁操作，可能会导致不必要的性能损耗，所以引入**锁粗化**的概念。

**定义**

锁粗化概念比较好理解，就是将==**多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁**==。

如上面实例：`vector` 每次 add 的时候都需要加锁操作，JVM 检测到对同一个对象（`vector`）连续加锁、解锁操作，会合并一个更大范围的加锁、解锁操作，即加锁解锁操作会移到 `for` 循环之外。

## 3.4 锁的升级

锁主要存在四种状态，依次是：==无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态==。它们会随着竞争的激烈而逐渐升级。**注意，锁可以升级不可降级，这种策略是为了提高获得锁和释放锁的效率**。

### 3.4.1 重量级锁

重量级锁通过对象内部的监视器（Monitor）实现。

其中，Monitor 的**本质**是，依赖于底层操作系统的 [Mutex Lock](http://dreamrunner.org/blog/2014/06/29/qian-tan-mutex-lock/) 实现。操作系统实现线程之间的切换，**需要从用户态到内核态的切换，切换成本非常高。**

### 3.4.2 轻量级锁

​	引入轻量级锁的主要目的，是在没有多线程竞争的前提下，减少传统的重量级锁使用操作系统互斥量产生的性能消耗。

当关闭偏向锁功能或者多个线程竞争偏向锁，导致**偏向锁升级为轻量级锁**，则会尝试获取轻量级锁，其步骤如下：

**获取锁**

1. 判断当前对象是否处于无锁状态？**若是**，则 JVM 首先将在当前线程的栈帧中，建立一个名为锁记录（Lock Record）的空间，用于存储锁对象目前的 Mark Word的 拷贝（官方把这份拷贝加了一个 Displaced 前缀，即 Displaced Mark Word）；**否则**，执行步骤（3）；
2. JVM 利用 CAS 操作尝试将对象的 Mark Word 更新为指向 Lock Record 的指正。如果**成功**，表示竞争到锁，则将锁标志位变成 `00`（表示此对象处于轻量级锁状态），执行同步操作；如果**失败**，则执行步骤（3）；
3. 判断当前对象的 Mark Word 是否指向当前线程的栈帧？如果**是**，则表示当前线程已经持有当前对象的锁，则直接执行同步代码块；**否则**，只能说明该锁对象已经被其他线程抢占了，当前线程便尝试使用**自旋**来获取锁。若自旋后没有获得锁，此时轻量级锁会升级为重量级锁，锁标志位变成 `10`，当前线程会被阻塞。

**释放锁**

轻量级锁的释放也是通过 CAS 操作来进行的，主要步骤如下：

1. 取出在获取轻量级锁保存在 Displaced Mark Word 中 数据。
2. 使用 CAS 操作将取出的数据替换当前对象的 Mark Word 中。如果**成功**，则说明释放锁成功；**否则**，执行（3）。
3. 无论（**2**）是否释放成功，都会唤醒被挂起的线程，重新争夺锁，访问同步代码块。

下图是争夺锁导致的**锁膨胀**的流程图：

![争夺锁导致的锁膨胀](/Users/jack/Desktop/md/images/201812081005.png)

- 其中，绿框的 `0` 指的是无偏向锁，`01` 指的是无锁状态。

------

**注意事项**

对于轻量级锁，其性能提升的依据是：“**对于绝大部分的锁，在整个生命周期内都是不会存在竞争的**”。如果打破这个依据则除了互斥的开销外，还有额外的 CAS 操作，因此在有多线程竞争的情况下，轻量级锁比重量级锁更慢。

### 3.4.3 偏向锁

==引入偏向锁主要**目的**是：为了在无多线程竞争的情况下，尽量减少不必要的轻量级锁执行路径。==

> 在上文，我们可以看到**偏向锁**时，Mark Word 的数据结构为：线程 ID、Epoch( 偏向锁的时间戳 )、对象分带年龄、是否是偏向锁( `1` )、锁标识位( `01` )。

只需要检查是否为偏向锁、锁标识为以及 ThreadID 即可，处理流程如下：

**获取偏向锁**

1. 检测 Mark Word是 否为可偏向状态，即是否为偏向锁的标识位为 `1` ，锁标识位为 `01` 。
2. 若为可偏向状态，则测试线程 ID 是否为当前线程 ID ？如果**是**，则执行步骤（5）；**否则**，执行步骤（3）。
3. 如果线程 ID 不为当前线程 ID ，则通过 CAS 操作竞争锁。竞争**成功**，则将 Mark Word 的线程 ID 替换为**当前**线程 ID ，则执行步骤（5）；**否则**，执行线程（4）。
4. 通过 CAS 竞争锁失败，证明当前存在多线程竞争情况，当到达全局安全点，获得偏向锁的线程被挂起，**偏向锁升级为轻量级锁**，然后被阻塞在安全点的线程继续往下执行同步代码块。
5. 执行同步代码块

**撤销偏向锁**

**偏向锁的释放采用了一种只有竞争才会释放锁的机制**，线程是**不会主动**去释放偏向锁，需要等待其他线程来竞争。

偏向锁的撤销需要等待**全局安全点**（这个时间点是上没有正在执行的代码）。其步骤如下：

1. 暂停拥有偏向锁的线程，判断锁对象是否还处于被锁定状态。
2. 撤销偏向锁，恢复到无锁状态（ `01` ）或者轻量级锁的状态。

> 关于**偏向锁的撤销**，如下是 [《Java 8 并发篇 - 冷静分析 Synchronized（下）》](https://juejin.im/post/5abc9de851882555770c8c72#heading-35) 对这块的描述：
>
> 首先会**暂停拥有偏向锁的线程并检查该线程是否存活**：
>
> 1. 如果线程**非活动状态**，则将**对象头设置为无锁状态**（其他线程会重新获取该偏向锁）。
> 2. 如果线程是**活动状态**，拥有偏向锁的栈会被执行，遍历偏向对象的锁记录，并**将对栈中的锁记录和对象头的 MarkWord 进行重置**：
>    - 要么**重新偏向于其他线程**(==即将偏向锁交给其他线程，相当于当前线程”被”释放了锁==)
>    - 要么**恢复到无锁**或者**标记锁对象不适合作为偏向锁**(此时锁会被升级为轻量级锁)
>
> **最后唤醒暂停的线程，被阻塞在安全点的线程继续往下执行同步代码块**

下图是偏向锁的获取和释放流程：

![偏向锁的获取和释放流程](/Users/jack/Desktop/md/images/201812081006.png)

**关闭偏向锁**

偏向锁在 JDK 1.6 以上，默认开启。开启后程序启动几秒后才会被激活，可使用 JVM 参数 `-XX：BiasedLockingStartupDelay = 0` 来关闭延迟。

如果确定锁**通常处于竞争状态**，则可通过JVM参数 `-XX:-UseBiasedLocking=false` 关闭偏向锁，那么默认会进入轻量级锁。

> 如下是 [《Java 8 并发篇 - 冷静分析 Synchronized（下）》](https://juejin.im/post/5abc9de851882555770c8c72) 对这块的描述：
>
> - 优势：偏向锁只需要在置换 ThreadID 的时候依赖一次 CAS 原子指令，其余时刻不需要 CAS 指令(相比其他锁)。
> - 隐患：由于一旦出现多线程竞争的情况就必须撤销偏向锁，所以偏向锁的撤销操作的性能损耗必须小于节省下来的 CAS 原子指令的性能消耗（这个通常只能通过大量压测才可知）。
> - 对比：轻量级锁是为了在线程交替执行同步块时提高性能，而偏向锁则是在只有一个线程执行同步块时进一步提高性能

### 3.4.4 对比和转换

如下是引用自 [《Java并发编程的艺术》](http://www.iocoder.cn/JUC/sike/synchronized/?vip#) 的**对比图**：

![偏向锁 vs 轻量级锁 vs 重量级锁](/Users/jack/Desktop/md/images/1-20190307115420667.jpeg)

如下是三种锁之间的**转换图**：

![偏向锁 => 轻量级锁 => 重量级锁](/Users/jack/Desktop/md/images/9EB59781-D801-4922-90CA-C6D34944BB0C.png)

如果觉得解释不够清晰的胖友，推荐阅读**占小狼**的 [《JVM 源码分析之 synchronized 实现》](https://www.jianshu.com/p/c5058b6fe8e5) 。

# 二、volatile 实现原理

`volatile` 涉及的内容，其实蛮多的，所以胖友直接看：

- [《【死磕 Java 并发】—– 深入分析 volatile 的实现原理》](http://www.iocoder.cn/JUC/sike/volatile/?vip)
- [聊聊并发（一）——深入分析Volatile的实现原理](https://www.infoq.cn/article/ftf-java-volatile)

🦅 **volatile 有什么用？**

`volatile` 保证内存可见性和禁止指令重排。

> 同时，`volatile` 可以提供部分原子性。

简单来说，`volatile` 用于多线程环境下的单次操作(单次读或者单次写)。

🦅 **volatile 变量和 atomic 变量有什么不同？**

- `volatile` 变量，可以确保先行关系，即写操作会发生在后续的读操作之前，但它并不能保证原子性。例如用 `volatile` 修饰 `count` 变量，那么 `count++` 操作就不是原子性的。
- AtomicInteger 类提供的 atomic 方法，可以让这种操作具有原子性。例如 `getAndIncrement()` 方法，会原子性的进行增量操作把当前值加一，其它数据类型和引用变量也可以进行相似操作。

🦅 **可以创建 volatile 数组吗?**

​	==Java 中可以创建 `volatile` 类型数组，不过只是一个指向数组的引用，而不是整个数组。==如果改变引用指向的数组，将会受到 `volatile` 的保护，但是如果多个线程同时改变数组的元素，`volatile` 标示符就不能起到之前的保护作用了。

同理，对于 Java POJO 类，使用 `volatile` 修饰，只能保证这个引用的可见性，不能保证其内部的属性。

🦅 **volatile 能使得一个非原子操作变成原子操作吗?**

​	一个典型的例子是在类中有一个 `long` 类型的成员变量。如果你知道该成员变量会被多个线程访问，如计数器、价格等，你最好是将其设置为 `volatile` 。为什么？**因为 Java 中读取 `long` 类型变量不是原子的，需要分成两步，如果一个线程正在修改该 `long` 变量的值，另一个线程可能只能看到该值的一半（前 32 位）。但是对一个 `volatile` 型的 `long` 或 `double` 变量的读写是原子。**

> ​	一种实践是用 `volatile` 修饰 `long` 和 `double` 变量，使其能按原子类型来读写。`double` 和 `long` 都是64位宽，因此对这两种类型的读是分为两部分的，第一次读取第一个 32 位，然后再读剩下的 32 位，这个过程不是原子的，但 Java 中 `volatile` 型的 `long` 或 `double` 变量的读写是原子的。

🦅 **volatile 类型变量提供什么保证？**

`volatile` 主要有两方面的作用：

1. 避免指令重排
2. 可见性保证

例如，JVM 或者 JIT 为了获得更好的性能会对语句重排序，但是 `volatile` 类型变量即使在没有同步块的情况下赋值也不会与其他语句重排序。

- `volatile` 提供 happens-before 的保证，确保一个线程的修改能对其他线程是可见的。
- 某些情况下，`volatile` 还能提供原子性，如读 64 位数据类型，像 `long` 和 `double` 都不是原子的(低 32 位和高 32 位)，但 `volatile` 类型的 `double` 和 `long` 就是原子的。**不过需要在 64 位的 JVM 虚拟机上**。详细的分析，可以看看 [《Java中 long 和 double 的原子性》](https://my.oschina.net/u/1753415/blog/724242) 。

🦅 **volatile 和 synchronized 的区别？**

1. `volatile` 本质是在告诉 JVM 当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取。`synchronized` 则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
2. `volatile` 仅能使用在变量级别。`synchronized` 则可以使用在变量、方法、和类级别的。
3. `volatile` 仅能实现变量的修改可见性，不能保证原子性。而`synchronized` 则可以保证变量的修改可见性和原子性。
4. `volatile` 不会造成线程的阻塞。`synchronized` 可能会造成线程的阻塞。
5. `volatile` 标记的变量不会被编译器优化。`synchronized`标记的变量可以被编译器优化。

> 另外，会有面试官会问 `volatile` 能否取代 `synchronized` 呢？答案肯定是不能，虽然说 `volatile` 被称之为轻量级锁，但是和 `synchronized` 是有本质上的区别，原因就是上面的几点落。

🦅 **什么场景下可以使用 volatile 替换 synchronized ？**

1. 只需要保证共享资源的可见性的时候可以使用 `volatile` 替代，`synchronized` 保证可操作的原子性一致性和可见性。
2. `volatile` 适用于新值不依赖于旧值的情形。
3. 1 写 N 读。
4. 不与其他变量构成不变性条件时候使用 `volatile` 。

![èå¾](/Users/jack/Desktop/md/images/synchroized-01.png)

# 三、死锁、活锁

死锁，是指两个或两个以上的进程（或线程）在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。

产生死锁的必要条件：

- 互斥条件：所谓互斥就是进程在某一时间内独占资源。
- 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
- 不剥夺条件：进程已获得资源，在末使用完之前，不能强行剥夺。
- 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系。

死锁的解决方法：

- 撤消陷于死锁的全部进程。
- 逐个撤消陷于死锁的进程，直到死锁不存在。
- 从陷于死锁的进程中逐个强迫放弃所占用的资源，直至死锁消失。
- 从另外一些进程那里强行剥夺足够数量的资源分配给死锁进程，以解除死锁状态。

🦅 **什么是活锁？**

活锁，任务或者执行者没有被阻塞，由于某些条件没有满足，导致一直重复尝试，失败，尝试，失败。

🦅 **死锁与活锁的区别？**

活锁和死锁的区别在于，处于活锁的实体是在不断的改变状态，所谓的“活”，而处于死锁的实体表现为等待；活锁有可能自行解开，死锁则不能。

实际上，聪慧的胖友是不是已经发现，死锁就是悲观锁可能产生的结果，而活锁是乐观锁可能产生的结果。

# 四、悲观锁、乐观锁

1）悲观锁

悲观锁，总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁。

- 传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。
- 再比如 Java 里面的同步原语 `synchronized` 关键字的实现也是悲观锁。

2）乐观锁

乐观锁，顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量。

- 像数据库提供的类似于 write_condition 机制，其实都是提供的乐观锁。

  > 例如，version 字段（比较跟上一次的版本号，如果一样则更新，如果失败则要重复读-比较-写的操作）

- 在 Java 中 `java.util.concurrent.atomic` 包下面的原子变量类就是使用了乐观锁的一种实现方式 CAS 实现的。

乐观锁的实现方式：

- 使用版本标识来确定读到的数据与提交时的数据是否一致。提交后修改版本标识，不一致时可以采取丢弃和再次尝试的策略。
- Java 中的 Compare and Swap 即 CAS ，当多个线程尝试使用 CAS 同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。

# 五、Java Lock 接口

> 艿艿：虽然 Lock 也翻译成锁，但是和上面的 [「Java 锁」](http://svip.iocoder.cn/Java/Concurrent/Interview/#) 分开，它更多强调的是 `synchronized` 和 `volatile` 关键字带来的重量级和轻量级锁。而 Lock 是 Java 锁接口，提供了更多灵活的功能。

![Lock](http://static.iocoder.cn/d2cd1e16577dd6482d1e58ace2062408)

`java.util.concurrent.locks.Lock` 接口，比 `synchronized` 提供更具拓展行的锁操作。它允许更灵活的结构，可以具有完全不同的性质，并且可以支持多个相关类的条件对象。它的优势有：

- 可以使锁更公平。
- 可以使线程在等待锁的时候响应中断。
- 可以让线程尝试获取锁，并在无法获取锁的时候立即返回或者等待一段时间。
- 可以在不同的范围，以不同的顺序获取和释放锁。

# 六、Java AQS

`java.util.concurrent.locks.AbstractQueuedSynchronizer` 抽象类，简称 AQS ，是一个用于构建锁和同步容器的同步器。事实上`concurrent` 包内许多类都是基于 AQS 构建。例如 ReentrantLock，Semaphore，CountDownLatch，ReentrantReadWriteLock，等。AQS 解决了在实现同步容器时设计的大量细节问题。

AQS 使用一个 FIFO 的队列表示排队等待锁的线程，队列头节点称作“哨兵节点”或者“哑节点”，它不与任何线程关联。其他的节点与等待线程关联，每个节点维护一个等待状态 `waitStatus` 。

可能这么说，胖友会一脸懵逼，最好的方式，还是直接去撸源码，可见如下的四篇文章。

> 可能胖友在阅读时，会有一定的挫败感，没关系，大家都是如此，包括艿艿，还有我认识的各种大佬。

- [《【死磕 Java 并发】—– J.U.C 之 AQS：AQS 简介》](http://www.iocoder.cn/JUC/sike/aqs-0-intro?vip)
- [《【死磕 Java 并发】—– J.U.C 之 AQS：CLH 同步队列》](http://www.iocoder.cn/JUC/sike/aqs-1-clh?vip)
- [《【死磕 Java 并发】—– J.U.C 之 AQS：同步状态的获取与释放》](http://www.iocoder.cn/JUC/sike/aqs-2?vip)
- [《【死磕 Java 并发】—– J.U.C 之 AQS：阻塞和唤醒线程》](http://www.iocoder.cn/JUC/sike/aqs-3?vip)

## 什么是可重入锁（ReentrantLock）？

举例来说明锁的可重入性。代码如下：

```
public class UnReentrant{

    Lock lock = new Lock();

    public void outer() {
        lock.lock();
        inner();
        lock.unlock();
    }

    public void inner() {
        lock.lock();
        //do something
        lock.unlock();
    }

}
```

- `#outer()` 方法中调用了 `#inner()` 方法，`#outer()` 方法先锁住了 `lock` ，这样 `#inner()` 就不能再获取 `lock` 。
- 其实调用 `#outer()` 方法的线程已经获取了 `lock` 锁，但是不能在 `#inner()` 方法中重复利用已经获取的锁资源，这种锁即称之为不可重入。
- 可重入就意味着：线程可以进入任何一个它已经拥有的锁所同步着的代码块。

`synchronized`、ReentrantLock 都是可重入的锁，可重入锁相对来说简化了并发编程的开发。

关于 ReentrantLock 类，详细的源码解析，可以看看 [《【死磕 Java 并发】—– J.U.C 之重入锁：ReentrantLock》](http://www.iocoder.cn/JUC/sike/ReentrantLock/?vip) 。

> 简单来说，ReenTrantLock 的实现是一种自旋锁，通过循环调用 CAS 操作来实现加锁。它的性能比较好也是因为避免了使线程进入内核态的阻塞状态。想尽办法避免线程进入内核的阻塞状态是我们去分析和理解锁设计的关键钥匙。

🦅 **synchronized 和 ReentrantLock 异同？**

- 相同点
  - 都实现了多线程同步和内存可见性语义。
  - 都是可重入锁。
- 不同点
  - 同步实现机制不同
    - `synchronized` 通过 Java 对象头锁标记和 Monitor 对象实现同步。
    - ReentrantLock 通过CAS、AQS（AbstractQueuedSynchronizer）和 LockSupport（用于阻塞和解除阻塞）实现同步。
      *
  - 可见性实现机制不同
    - `synchronized` 依赖 JVM 内存模型保证包含共享变量的多线程内存可见性。
    - ReentrantLock 通过 ASQ 的 `volatile state` 保证包含共享变量的多线程内存可见性。
  - 使用方式不同
    - `synchronized` 可以修饰实例方法（锁住实例对象）、静态方法（锁住类对象）、代码块（显示指定锁对象）。
    - ReentrantLock 显示调用 tryLock 和 lock 方法，需要在 `finally` 块中释放锁。
  - 功能丰富程度不同
    - `synchronized` 不可设置等待时间、不可被中断（interrupted）。
    - ReentrantLock 提供有限时间等候锁（设置过期时间）、可中断锁（lockInterruptibly）、condition（提供 await、condition（提供 await、signal 等方法）等丰富功能
  - 锁类型不同
    - `synchronized` 只支持非公平锁。
    - ReentrantLock 提供公平锁和非公平锁实现。当然，在大部分情况下，非公平锁是高效的选择。

> 在 `synchronized` 优化以前，它的性能是比 ReenTrantLock 差很多的，但是自从 `synchronized` 引入了偏向锁，轻量级锁（自旋锁）后，两者的性能就差不多了，在两种方法都可用的情况下，官方甚至建议使用 `synchronized` 。
>
> 并且，实际代码实战中，可能的优化场景是，通过读写分离，进一步性能的提升，所以使用 ReentrantReadWriteLock 。😝

## ReadWriteLock 是什么？

ReadWriteLock ，读写锁是，用来提升并发程序性能的锁分离技术的 Lock 实现类。可以用于 “多读少写” 的场景，读写锁支持多个读操作并发执行，写操作只能由一个线程来操作。

ReadWriteLock 对向数据结构相对不频繁地写入，但是有多个任务要经常读取这个数据结构的这类情况进行了优化。ReadWriteLock 使得你可以同时有多个读取者，只要它们都不试图写入即可。如果写锁已经被其他任务持有，那么任何读取者都不能访问，直至这个写锁被释放为止。

ReadWriteLock 对程序性能的提高主要受制于如下几个因素：

1. 数据被读取的频率与被修改的频率相比较的结果。
2. 读取和写入的时间
3. 有多少线程竞争
4. 是否在多处理机器上运行

ReadWriteLock 的源码解析，可以看看 [《【死磕 Java 并发】—– J.U.C 之读写锁：ReentrantReadWriteLock》](http://www.iocoder.cn/JUC/sike/ReentrantReadWriteLock/) 。

## Condition 是什么？

在没有 Lock 之前，我们使用 `synchronized` 来控制同步，配合 Object 的 `#wait()`、`#notify()` 等一系列方法可以实现**等待 / 通知模式**。在 Java SE 5 后，Java 提供了 Lock 接口，相对于 `synchronized` 而言，Lock 提供了条件 Condition ，对线程的等待、唤醒操作更加详细和灵活。下图是 Condition 与 Object 的监视器方法的对比（摘自《Java并发编程的艺术》）：

![Condition 与 Object 的监视器方法的对比](http://static.iocoder.cn/e7e7bb0837bbe68a4364366d4ec9c5db)

- Condition 的使用，可以看看 [《怎么理解 Condition》](http://www.importnew.com/9281.html)
- Condition 的源码，可以看看 [《【死磕 Java 并发】—– J.U.C 之 Condition》](http://www.iocoder.cn/JUC/sike/Condition/) 。

🦅 **用三个线程按顺序循环打印 abc 三个字母，比如 abcabcabc ？**

- 使用 Lock + Condition 来实现。具体代码，参看 [《用三个线程按顺序循环打印 abc 三个字母，比如 abcabcabc》](https://blog.csdn.net/Big_Blogger/article/details/65629204) 。
- 使用 `synchronized` + await/notifyAll 来实现，参看 [《Java用三个线程按顺序循环打印 abc 三个字母,比如 abcabcabc》](https://blog.csdn.net/weixin_41704428/article/details/80482928) 。

## LockSupport 是什么？

LockSupport 是 JDK 中比较底层的类，用来创建锁和其他同步工具类的基本线程阻塞。

- Java 锁和同步器框架的核心 AQS(AbstractQueuedSynchronizer)，就是通过调用 `LockSupport#park()`和 `LockSupport#unpark()`方法，来实现线程的阻塞和唤醒的。
- LockSupport 很类似于二元信号量(只有 1 个许可证可供使用)，如果这个许可还没有被占用，当前线程获取许可并继续执行；如果许可已经被占用，当前线程阻塞，等待获取许可。

对于 LockSupport 了解即可，面试一般问的不多。感兴趣的胖友，可以看看如下文章：

- [《多线程同步工具 —— LockSupport》](https://www.cnblogs.com/hvicen/p/6217303.html)
- [《Java 并发编程 —— LockSupport》](http://www.tianshouzhi.com/api/tutorials/mutithread/303) 带部分源码解析。

# Java 内存模型

关于 Java 内存模型，涉及的内容会很多，所以建议胖友看如下的 [《深入Java内存模型.pdf》](https://files-cdn.cnblogs.com/files/skywang12345/%E6%B7%B1%E5%85%A5Java%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B.pdf) 这本小书。

然后，看完之后你肯定会忘记，就可以靠 [《《深入理解 Java 内存模型》读书笔记》](http://www.iocoder.cn/JUC/zhisheng/Java-Memory-Model?vip) 来补刀。

再另外，[《深入拆解 Java 虚拟机》](http://gk.link/a/100kc) 的 [「第五部分 高效并发」](http://svip.iocoder.cn/Java/Concurrent/Interview/#) 也推荐阅读。

## 什么是 Java 内存模型？

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