![Lock](http://static.iocoder.cn/d2cd1e16577dd6482d1e58ace2062408)

`java.util.concurrent.locks.Lock` 接口，比 `synchronized` 提供更具拓展行的锁操作。它允许更灵活的结构，可以具有完全不同的性质，并且可以支持多个相关类的条件对象。它的优势有：

- 可以使锁更公平。
- 可以使线程在等待锁的时候响应中断。
- 可以让线程尝试获取锁，并在无法获取锁的时候立即返回或者等待一段时间。
- 可以在不同的范围，以不同的顺序获取和释放锁。

# 1、Java AQS

`java.util.concurrent.locks.AbstractQueuedSynchronizer` 抽象类，简称 AQS ，是一个用于构建锁和同步容器的同步器。事实上`concurrent` 包内许多类都是基于 AQS 构建。例如 ReentrantLock，Semaphore，CountDownLatch，ReentrantReadWriteLock，等。AQS 解决了在实现同步容器时设计的大量细节问题。

## 1.1 简介

​	==AQS 使用一个 FIFO 的队列表示排队等待锁的线程，队列头节点称作“哨兵节点”或者“哑节点”，它不与任何线程关联。其他的节点与等待线程关联，每个节点维护一个等待状态 `waitStatus` 。==

### 同步状态：

AQS 的主要使用方式是**继承**，子类通过继承同步器，并实现它的**抽象方法**来管理同步状态。

AQS 使用一个 `int` 类型的成员变量 `state` 来**表示同步状态**：

- 当 `state > 0` 时，表示已经获取了锁。
- 当 `state = 0` 时，表示释放了锁。

它提供了三个方法，来对同步状态 `state` 进行操作，并且 AQS 可以确保对 `state` 的操作是**安全**的：

- `setState()`
- `setState(int newState)`
- `compareAndSetState(int expect, int update)`

### 同步队列：

AQS 通过内置的双向 FIFO 同步队列来完成资源获取线程的**排队工作**：

- 如果当前线程获取同步状态失败（锁）时，AQS 则会将当前线程以及等待状态等信息构造成一个节点（Node）并将其加入同步队列，同时会阻塞当前线程
- 当同步状态**释放**时，则会把节点中的线程唤醒，使其再次尝试获取同步状态。

### 主要方法：

AQS 主要提供了如下**方法**：

- `getState()`：返回同步状态的当前值。
- `setState(int newState)`：设置当前同步状态。
- `compareAndSetState(int expect, int update)`：使用 CAS 设置当前状态，该方法能够保证状态设置的原子性。
- 【可重写】`#tryAcquire(int arg)`：独占式获取同步状态，获取同步状态成功后，其他线程需要等待该线程释放同步状态才能获取同步状态。
- 【可重写】`#tryRelease(int arg)`：独占式释放同步状态。
- 【可重写】`#tryAcquireShared(int arg)`：共享式获取同步状态，返回值大于等于 0 ，则表示获取成功；否则，获取失败。
- 【可重写】`#tryReleaseShared(int arg)`：共享式释放同步状态。
- 【可重写】`#isHeldExclusively()`：当前同步器是否在独占式模式下被线程占用，一般该方法表示是否被当前线程所独占。
- `acquire(int arg)`：独占式获取同步状态。如果当前线程获取同步状态成功，则由该方法返回；否则，将会进入同步队列等待。该方法将会调用**可重写**的 `#tryAcquire(int arg)` 方法；
- `#acquireInterruptibly(int arg)`：与 `#acquire(int arg)` 相同，但是该方法响应中断。当前线程为获取到同步状态而进入到同步队列中，如果当前线程被中断，则该方法会抛出InterruptedException 异常并返回。
- `#tryAcquireNanos(int arg, long nanos)`：超时获取同步状态。如果当前线程在 nanos 时间内没有获取到同步状态，那么将会返回 false ，已经获取则返回 true 。
- `#acquireShared(int arg)`：共享式获取同步状态，如果当前线程未获取到同步状态，将会进入同步队列等待，与独占式的主要区别是在同一时刻可以有多个线程获取到同步状态；
- `#acquireSharedInterruptibly(int arg)`：共享式获取同步状态，响应中断。
- `#tryAcquireSharedNanos(int arg, long nanosTimeout)`：共享式获取同步状态，增加超时限制。
- `#release(int arg)`：独占式释放同步状态，该方法会在释放同步状态之后，将同步队列中第一个节点包含的线程唤醒。
- `#releaseShared(int arg)`：共享式释放同步状态。

从上面的方法看下来，基本上可以分成 **3** 类：

- **独占式**获取与释放同步状态
- **共享式**获取与释放同步状态
- 查询**同步队列**中的等待线程情况

## 1.2 [CLH 同步队列](http://www.iocoder.cn/JUC/sike/aqs-1-clh/)

### 1. 简介

==CLH 同步队列是一个 FIFO **双向**队列，AQS 依赖它来完成同步状态的管理==：

- **当前线程如果获取同步状态失败时，AQS则会将当前线程已经等待状态等信息构造成一个节点（Node）并将其加入到CLH同步队列，同时会阻塞当前线程**
- 当同步状态释放时，会把首节点唤醒（公平锁），使其再次尝试获取同步状态。

### 2. Node

> Node 是 AbstractQueuedSynchronizer 的内部静态类。

```java
static final class Node {

    // 共享
    static final Node SHARED = new Node();
    // 独占
    static final Node EXCLUSIVE = null;

    /**
     * 因为超时或者中断，节点会被设置为取消状态，被取消的节点时不会参与到竞争中的，他会一直保持取消状态不会转变为其他状态
     */
    static final int CANCELLED =  1;
    /**
     * 后继节点的线程处于等待状态，而当前节点的线程如果释放了同步状态或者被取消，将会通知后继节点，使后继节点的线程得以运行
     */
    static final int SIGNAL    = -1;
    /**
     * 节点在等待队列中，节点线程等待在Condition上，当其他线程对Condition调用了signal()后，该节点将会从等待队列中转移到同步队列中，加入到同步状态的获取中
     */
    static final int CONDITION = -2;
    /**
     * 表示下一次共享式同步状态获取，将会无条件地传播下去
     */
    static final int PROPAGATE = -3;

    /** 等待状态 */
    volatile int waitStatus;

    /** 前驱节点，当节点添加到同步队列时被设置（尾部添加） */
    volatile Node prev;

    /** 后继节点 */
    volatile Node next;

    /** 等待队列中的后续节点。如果当前节点是共享的，那么字段将是一个 SHARED 常量，也就是说节点类型（独占和共享）和等待队列中的后续节点共用同一个字段 */
    Node nextWaiter;
    
    /** 获取同步状态的线程 */
    volatile Thread thread;

    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() { // Used to establish initial head or SHARED marker
    }

    Node(Thread thread, Node mode) { // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }

    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
    
}
```

- waitStatus字段，等待状态，用来控制线程的阻塞和唤醒，并且可以避免不必要的调用LockSupport的park(...)和unpark(...)方法。目前有4种：CANCELLED，SIGNAL，CONDITION和PROPAGATE。

  - ==实际上，有第 **5** 种，`INITIAL` ，值为 0 ，初始状态。==
  - 每个等待状态代表的含义，它不仅仅指的是 Node **自己**的线程的等待状态，也可以是**下一个**节点的线程的等待状态。

- CLH 同步队列，结构图如下：

  ![CLH 同步队列](/Users/jack/Desktop/md/images/2018120810001.png)

  - `prev` 和 `next` 字段，是 **AbstractQueuedSynchronizer** 的字段，分别指向同步队列的头和尾。
  - `head` 和 `tail` 字段，分别指向 Node 节点的**前一个**和**后一个** Node 节点，从而实现**链式双向队列**。再配合上 `prev` 和 `next` 字段，快速定位到同步队列的头尾。

- `thread` 字段，Node 节点对应的**线程 Thread** 。

- ==nextWaiter字段，Node 节点获取同步状态的模型( Mode )==。tryAcquire(int args)和tryAcquireShared(int args)方法，分别是独占式和共享式获取同步状态。在获取失败时，它们都会调用addWaiter(Node mode)

   方法入队。而nextWaiter就是用来表示是哪种模式：

  - `SHARED` **静态 + 不可变**字段，枚举**共享**模式。
  - `EXCLUSIVE` **静态 + 不可变**字段，枚举**独占**模式。
  - `isShared()` 方法，判断是否为共享式获取同步状态。

- `predecessor()` 方法，获得 Node 节点的**前一个** Node 节点。在方法的内部，`Node p = prev` 的本地拷贝，是为了避免并发情况下，`prev` 判断完 `== null` 时，恰好被修改，从而保证线程安全。

- 构造方法有3个，分别是：

  - `Node()` 方法：用于 `SHARED` 的创建。
  - Node(Thread thread, Node mode)方法：用于addWaiter(Node mode)方法。
    - 从 `mode` 方法参数中，我们也可以看出它代表获取同步状态的**模式**。
    - 在本文中，我们会看到这个构造方法的使用。
  - Node(Thread thread, int waitStatus)方法，用于addConditionWaiter()方法。

### 3. 入列

步骤：

- `tail` 指向新节点。
- 新节点的 `prev` 指向当前最后的节点。
- 当前最后一个节点的 `next` 指向当前节点。

过程图如下：

![入列 流程](/Users/jack/Desktop/md/images/2018120810002.png)

但是，实际上，入队逻辑实现的 `addWaiter(Node)` 方法，需要考虑**并发**的情况。它通过 **CAS** 的方式，来保证正确的添加 Node 。这个方法作用就是：==**请求失败后，将当前线程链入队尾并挂起，之后等待被唤醒**==。

代码如下：

```java
/**
     * 将Node节点加入等待队列
     * 1）快速入队，入队成功的话，返回node
     * 2）入队失败的话，使用正常入队
     * 注意：快速入队与正常入队相比，可以发现，正常入队仅仅比快速入队多而一个判断队列是否为空且为空之后的过程
     * @return 返回当前要插入的这个节点，注意不是前一个节点
     */
 1: private Node addWaiter(Node mode) {
 2:     // 新建以当前线程为node的节点，
 3:     Node node = new Node(Thread.currentThread(), mode);
     //快速入队
 4:     // 记录原来队列的尾节点
 5:     Node pred = tail;
 6:     // 快速尝试，如果原尾节点不为空，添加新节点为尾节点，即新节点的prev指向原来的尾节点
 /**
   * 基于CAS将node设置为尾节点，如果设置失败，说明在当前线程获取尾节点到现在这段过程中已经有其他线程将尾节点给替换过了
   * 注意：假设有链表node1-->node2-->pred（当然是双链表，这里画成双链表才合适）,
   * 通过CAS将pred替换成了node节点，即当下的链表为node1-->node2-->node,
   * 然后根据上边的"node.prev = pred"与下边的"pred.next = node"将pred插入到双链表中去，组成最终的链表如下：
   * node1-->node2-->pred-->node
   * 这样的话，实际上我们发现没有指定node2.next=pred与pred.prev=node2，这是为什么呢？
   * 因为在之前这两句就早就执行好了，即node2.next和pred.prev这连个属性之前就设置好了
*/
 7:     if (pred != null) {
 8:         // 设置新 Node 节点的尾节点为原尾节点，即新节点的prev指向原来的尾节点
 9:         node.prev = pred;
10:         // CAS 设置新的尾节点
11:         if (compareAndSetTail(pred, node)) {
12:             // 成功，原尾节点的下一个节点为新节点
13:             pred.next = node;
14:             return node;
15:         }
16:     }
17:     // 失败，多次尝试，直到成功
18:     enq(node);
19:     return node;
20: }
```

- 第 3 行：创建新节点 `node` 。在创建的构造方法，`mode` 方法参数，传递获取同步状态的模式。

- 第 5 行：记录原队列的尾节点 `tail` 。

- 在下面的代码，会分成2部分：

  - 第 6 至 16 行：**快速**尝试，添加新节点为尾节点。
  - 第 18 行：添加失败，**多次**尝试，直到成功添加。

-  第 **1** 部分

  - 第 7 行：当**原**尾节点非空，才执行**快速**尝试的逻辑。在下面的 `enq(Node node)` 方法中，我们会看到，**首**节点未初始化的时，`head` 和 `tail` 都为空。
  - 第 9 行：设置**新**节点的**尾**节点为**原**尾节点。
  - 第 11 行：调用 `compareAndSetTail(Node expect, Node update)` 方法，使用 **Unsafe** 来 **CAS** 设置**尾**节点 `tail` 为**新**节点。代码如下：

  ```java
  private static final Unsafe unsafe = Unsafe.getUnsafe();
  
  private static final long tailOffset = unsafe.objectFieldOffset (AbstractQueuedSynchronizer.class.getDeclaredField("tail"));  // 这块代码，实际在 static 代码块，此处为了方便理解，做了简化。
  
  private final boolean compareAndSetTail(Node expect, Node update) {
      return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
  }
  ```

  - 第 13 行：添加**成功**，最终，将**原**尾节点的下一个节点为**新**节点。
  - 第 14 行：返回**新**节点。
  - 如果添加**失败**，因为存在多线程并发的情况，此时需要执行【第 18 行】的代码。

- 第 **2** 部分

- 调用 `enq(Node node)` 方法，**多次**尝试，直到成功添加。

  正常入队代码如下：

  ```
  private Node enq(final Node node) {
       // 死循环，多次尝试，一定要阻塞到入队成功为止
       for (;;) {
           // 记录原尾节点
           Node t = tail;
           // 如果尾节点为null，说明当前等待队列为空，创建首尾节点都为 new Node()
           if (t == null) {
   /*
   基于CAS将新节点（一个dummy节点）设置到头上head去，如果发现内存中的当前值不是null，则说明，在这个过程中，已经有其他线程设置过了。
  * 当成功的将这个dummy节点设置到head节点上去时，我们又将这个head节点设置给了tail节点，即head与tail都是当前这个dummy节点，
  * 之后有新节点入队的话，就插入到该dummy之后
  */
               if (compareAndSetHead(new Node()))	
                   tail = head;
           // 原尾节点存在，添加新节点为尾节点
           } else {
               //设置为尾节点
               node.prev = t;
               // CAS 设置新的尾节点
               if (compareAndSetTail(t, node)) {
                   // 成功，原尾节点的下一个节点为新节点
                  t.next = node;
                   return t;
               }
           }
       }
   }
  ```

  - 第 3 行：“**死**”循环，多次尝试，直到成功添加**为止**【第 18 行】。

  - 第 5 行：记录原尾节点 `t` 。和 `addWaiter(Node node)` 方法的【第 5 行】相同。

  - 第 10 至 19 行：原尾节点存在，添加新节点为尾节点。和 `addWaiter(Node node)` 方法的【第 7 至 16 行】相同。

  - 第 6 至 9 行：原尾节点不存在，创建**首尾**节点都为 **new Node()** 。**注意**，此时修改的**首尾**节点是重新创建( `new Node()` )的，而不是**新节点**！

    - ==通过这样的方式，初始化好同步队列的首尾。==另外，在 AbstractQueuedSynchronizer 的设计中，==`head` 字段，是一个“占位节点”，代表**最后一个**获得到同步状态的节点(线程)，==实际它已经**出列**，所以它的 `Node.next` 才是**真正**的队首。当然，同步队列的初始时，`new Node()` 也是满足这个条件，因为有**新的** Node 进队列，**目前就已经有线程获得到同步状态**。

    - `compareAndSetHead(Node update)` 方法，使用 Unsafe 来 CAS 设置尾节点 `head`为新节点。代码如下：

      ```java
      private static final Unsafe unsafe = Unsafe.getUnsafe();
      
      private static final long headOffset = unsafe.objectFieldOffset (AbstractQueuedSynchronizer.class.getDeclaredField("head"));  // 这块代码，实际在 static 代码块，此处为了方便理解，做了简化。
      
      private final boolean compareAndSetHead(Node update) {
          return unsafe.compareAndSwapObject(this, headOffset, null, update);
      }
      ```

      - **注意**，==第三个方法参数为 `null` ，代表需要原 `head` 为空才可以设置。==和 `compareAndSetTail(Node expect, Node update)` 方法，类似。

### 4. 出列

​	CLH 同步队列遵循 FIFO，**首节点的线程释放同步状态后，将会唤醒它的下一个节点（`Node.next`）。而后继节点将会在获取同步状态成功时，将自己设置为首节点( `head` )。**

​	这个过程非常简单，`head` 执行该节点并断开原首节点的 `next` 和当前节点的 `prev` 即可。==注意，在这个过程是**不需要使用 CAS 来保证**的，因为**只有一个**线程，能够成功获取到同步状态。==

过程图如下：

![过程图](/Users/jack/Desktop/md/images/2018120810003.png)

`setHead(Node node)` 方法，实现上述的**出列**逻辑。代码如下：

```java
private void setHead(Node node) {
    head = node;
    node.thread = null;
    node.prev = null;
}
```

## 1.3 [同步状态的获取与释放](http://www.iocoder.cn/JUC/sike/aqs-2/)

​	AQS 的设计模式采用的**模板方法模式**，子类通过继承的方式，实现它的抽象方法来管理同步状态。对于子类而言，它并没有太多的活要做，AQS 已经提供了大量的模板方法来实现同步，主要是分为三类：

- 独占式获取和释放同步状态
- 共享式获取和释放同步状态
- 查询同步队列中的等待线程情况。

自定义子类使用 AQS 提供的模板方法，就可以实现**自己的同步语义**。

### 1. 独占式

​	独占式，**同一时刻，仅有一个线程持有同步状态**。

#### 1.1 独占式同步状态获取

`acquire(int arg)` 方法，为 AQS 提供的**模板方法**。该方法为独占式获取同步状态，但是该方法对**中断不敏感**。也就是说，==由于线程获取同步状态失败而加入到 CLH 同步队列中，后续对该线程进行中断操作时，线程**不会**从 CLH 同步队列中移除。==代码如下：

```java
public final void acquire(int arg) {
    // tryAcquire 由子类实现本身不会阻塞线程，如果返回 true,则线程继续，
    // 如果返回 false 那么就加入阻塞队列阻塞线程，并等待前继结点释放锁。
     if (!tryAcquire(arg) &&
         acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
         // acquireQueued返回true，说明当前线程被中断唤醒后获取到锁，
        // 重置其interrupt status为true。
         selfInterrupt();
}
```

- 第 2 行：调用 `tryAcquire(int arg)` 方法，去尝试获取同步状态，获取成功则设置锁状态并返回 true ，否则获取失败，返回 false。若获取成功，`acquire(int arg)` 方法，直接返回，**不用线程阻塞**，自旋直到获得同步状态成功。前面的判断如果返回false则直接跳出if语句，&&有短路的功能。

  - `tryAcquire(int arg)` 方法，**需要自定义同步组件自己实现，该方法必须要保证线程安全的获取同步状态。**代码如下：

    ```Java
    protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
    }
    ```

    - **直接**抛出 UnsupportedOperationException 异常。

- 第 3 行：如果 `tryAcquire(int arg)` 方法返回 false ，即获取同步状态失败(!false返回true，所以会继续执行&&后面的判断)，则调用 **`addWaiter(Node mode)` 方法，将当前线程加入到 CLH 同步队列尾部。并且， `mode` 方法参数为 `Node.EXCLUSIVE` ，表示独占模式。**

- 第 3 行：调用 `boolean acquireQueued(Node node, int arg)` 方法，**自旋直到获得同步状态成功**。详细解析，见 下面的1.1.1 acquireQueued中。另外，==该方法的返回值类型为 `boolean` ，当返回 true 时，表示在这个过程中，发生过**线程中断**。==但是呢，这个方法又会**清理**线程中断的**标识**，所以在种情况下，需要调用【第 4 行】的 `selfInterrupt()` 方法，**恢复线程中断的标识**，代码如下：

  ```Java
  static void selfInterrupt() {
      Thread.currentThread().interrupt();
  }
  ```

##### 1.1.1 acquireQueued

`boolean acquireQueued(Node node, int arg)` 方法，为一个**自旋**的过程，也就是说，**当前线程（Node）进入同步队列后，就会进入一个自旋的过程，每个节点都会自省地观察，当条件满足，获取到同步状态后，就可以从这个自旋过程中退出，否则会一直执行下去。**(自旋就是一个死循环的过程)

流程图如下：

![流程图](/Users/jack/Desktop/md/images/2018120811001.png)

代码如下：

```java
 1: final boolean acquireQueued(final Node node, int arg) {
 2:     // 记录是否获取同步状态成功
 3:     boolean failed = true;
 4:     try {
 5:         // 记录过程中，是否发生线程中断
 6:         boolean interrupted = false;
 7:         /*
 8:          * 自旋过程，其实就是一个死循环而已,根据一些条件判断是否跳出死循环，即自旋得到的结果
			   等待前继结点释放锁，直到node的前驱节点p之前的所有节点都执行完毕，p成为了head且node请求成功了
 9:          */
10:         for (;;) {
11:             // 通过predecessor，获取插入节点的前一个节点p
12:             final Node p = node.predecessor();
    /*
     * 注意：
     * 1、这个是跳出循环的唯一条件，除非抛异常
     * 2、如果p == head && tryAcquire(arg)第一次循环就成功了，interrupted为false，不需要中断自己
     *   如果p == head && tryAcquire(arg)第一次以后的循环中如果执行了挂起操作后才成功了，interrupted为true，就要中断自己了
   */
13:             // 当前线程的前驱节点是头结点，且同步状态成功，则返回true
14:             if (p == head && tryAcquire(arg)) {
15:                 setHead(node);	//前继出队，设置node为头节点
16:                 p.next = null; // help GC
17:                 failed = false;
18:                 return interrupted;
19:             }
20:             // 获取失败，线程等待--具体后面介绍
21:             if (shouldParkAfterFailedAcquire(p, node) &&
22:                     parkAndCheckInterrupt())
23:                 interrupted = true;
24:         }
25:     } finally {
26:         // 获取同步状态发生异常，取消获取。
27:         if (failed)
28:             cancelAcquire(node);
29:     }
30: }
```

- 第 3 行：`failed` 变量，记录是否**获取同步状态**成功。
- 第 6 行：`interrupted` 变量，记录获取过程中，是否发生**线程中断**。
- 第 7 至 24 行：“死”循环，自旋直到获得同步状态成功。
- 第 12 行：调用 `Node的predecessor()` 方法，获得当前线程的前一个节点(前驱结点) `p` 。
- 第 14 行：`p == head` 代码块，若满足，则表示当前线程的前一个节点为头节点，因为 `head` 是最后一个获得同步状态成功的节点，此时调用 `tryAcquire(int arg)` 方法，尝试获得同步状态。在 `acquire(int arg)` 方法的【第 2 行】，也调用了这个方法。
- 第 15 至 18 行：当前节点( 线程 )获取同步状态成功：
  - 第 15 行：设置当前节点( 线程 )为新的 `head` 。
  - 第 16 行：设置老的头节点 `p` 不再指向下一个节点，让它自身更快的被 GC 。
  - 第 17 行：标记 `failed = false` ，表示**获取同步状态**成功。
  - 第 18 行：返回记录获取过程中，是否发生**线程中断**。
- 第 20 至 24 行：获取失败，线程等待唤醒，从而进行下一次的同步状态获取的尝试。
  - 第 21 行：调用 `shouldParkAfterFailedAcquire(Node pre, Node node)` 方法，判断获取失败后，是否当前线程需要阻塞等待。
- 第 26 至 29 行：获取同步状态的过程中，**发生异常**，取消获取。
- 第 28 行：调用 `cancelAcquire(Node node)` 方法，取消获取同步状态。详细解析，见1.1.3 cancelAcquire。

##### 1.1.2 shouldParkAfterFailedAcquire

他的作用主要是：

- 确定后继是否需要park;
- 跳过被取消的结点;
- 设置前继的waitStatus为SIGNAL.

```java
	/**
     * 检测当前节点是否可以被安全的挂起（阻塞）
     * @param pred    当前节点的前驱节点
     * @param node    当前节点
     */
 1: private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
 2:     //获取前驱节点（即当前线程的前一个节点）的等待状态
 3:     int ws = pred.waitStatus;
 4:     if (ws == Node.SIGNAL) //  Node.SIGNAL，等于-1
 5:         /*
 6:          * 1）当ws>0(即CANCELLED==1），前驱节点的线程被取消了，我们会将该节点之前的连续几个被取消的前驱节点从队列中剔除，返回false（即不能挂起）
         	* 2）如果ws<=0&&!=SIGNAL,将当前节点的前驱节点的等待状态设为SIGNAL
         */
 9:         return true;
10:     if (ws > 0) { // Node.CANCEL
11:         /*
12:          * Predecessor was cancelled. Skip over predecessors and
13:          * indicate retry.
14:          */
    		// 跳过被取消的结点。
15:         do {
16:             node.prev = pred = pred.prev;
    		/*
           		 * 等同于
                 * pred = pred.prev;
                 * node.prev = pred;
            */
17:         } while (pred.waitStatus > 0);
18:         pred.next = node;
19:     } else { // 0 或者 Node.PROPAGATE，小于0
    /*
             * 尝试将当前节点的前驱节点的等待状态设为SIGNAL
             * 1，这为什么用CAS，现在已经入队成功了，前驱节点就是pred，除了node外应该没有别的线程在操作这个节点了，那为什么还要用CAS？而不直接赋值呢？
             * （解释：因为pred可以自己将自己的状态改为cancel，也就是pred的状态可能同时会有两条线程（pred和node）去操作）
             * 2，既然前驱节点已经设为SIGNAL了，为什么最后还要返回false
             * （因为CAS可能会失败，这里不管失败与否，都返回false，下一次执行该方法的之后，pred的等待状态就是SIGNAL了）
             */
25:         compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
26:     }
27:     return false;
28: }
```

- `pred` 和 `node` 方法参数，传入时，要求==前者必须是后者的前一个节点。==
- 第 3 行：获得前一个节点( `pre` )的等待状态。下面会根据这个状态有**三种**情况的处理。
- 第 4 至 9 行：==等待状态为 `Node.SIGNAL` 时，表示 `pred` 的下一个节点 `node` 的线程需要阻塞等待==。在 `pred`的线程释放同步状态时，会对 `node` 的线程进行**唤醒**通知。所以，【第 9 行】返回 true ，表明当前线程可以被 park**，**安全的阻塞等待。
- 第 19 至 26 行：**等待状态为0或者Node.PROPAGATE时，通过CAS设置，将状态修改为Node.SIGNAL，即下一次重新执行shouldParkAfterFailedAcquire(Node pred, Node node)方法时，满足【第 4 至 9 行】的条件。**
  - 但是，对于本次执行，【第 27 行】返回 false 。
  - ==另外，等待状态不会为 `Node.CONDITION` ，因为它用在 ConditonObject 中==。
- 第 10 至 18 行：==等待状态为NODE.CANCELLED时，则表明该线程的前一个节点已经等待超时或者被中断了，则需要从 CLH 队列中将该前一个节点删除掉，循环回溯，直到前一个节点状态<= 0。==
  - 对于本次执行，【第 27 行】返回 false ，需要下一次再重新执行 `shouldParkAfterFailedAcquire(Node pred, Node node)` 方法，看看满足哪个条件。
  - 整个过程如下图：![过程](/Users/jack/Desktop/md/images/shouldParkAfterFailedAcquire-02.png)

##### 1.1.3 cancelAcquire

```java
 1: private void cancelAcquire(Node node) {
 2:     // Ignore if node doesn't exist
 3:     if (node == null)
 4:         return;
 5: 
 6:     node.thread = null;
 7: 
 8:     // Skip cancelled predecessors
 9:     Node pred = node.prev;
10:     while (pred.waitStatus > 0)	//表示是CANCELLED状态的
    //从 CLH 队列中将该前一个节点删除掉，循环回溯
11:         node.prev = pred = pred.prev;	
12: 
13:     // predNext is the apparent node to unsplice. CASes below will
14:     // fail if not, in which case, we lost race vs another cancel
15:     // or signal, so no further action is necessary.
16:     Node predNext = pred.next;
17: 
18:     // Can use unconditional write instead of CAS here.
19:     // After this atomic step, other Nodes can skip past us.
20:     // Before, we are free of interference from other threads.
21:     node.waitStatus = Node.CANCELLED;
22: 
23:     // If we are the tail, remove ourselves.
     //compareAndSetTail是Boolean类型的，只能用于入队
24:     if (node == tail && compareAndSetTail(node, pred)) {//CAS设置pred为新的尾节点。
25:         compareAndSetNext(pred, predNext, null);
26:     } else {
27:         // If successor needs signal, try to set pred's next-link
28:         // so it will get one. Otherwise wake it up to propagate.
29:         int ws;
30:         if (pred != head &&
31:             ((ws = pred.waitStatus) == Node.SIGNAL ||
32:              (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
33:             pred.thread != null) {
34:             Node next = node.next;
35:             if (next != null && next.waitStatus <= 0)
36:                 compareAndSetNext(pred, predNext, next);
37:         } else {
38:             unparkSuccessor(node);
39:         }
40: 
41:         node.next = node; // help GC
42:     }
43: }
```

- 第 2 至 4 行：忽略，若传入参数 `node` 为空。

- 第 6 行：将节点的等待线程置空。

- 第 9 行：获得node节点的前一个节点pred。

  - 第 10 至 11 行： 逻辑同 `shouldParkAfterFailedAcquire(Node pred, Node node)` 的【第 15 至 17 行】。

- 第 16 行：获得pred的下一个节点predNext。predNext从表面上看，和node是等价的。

  - 但是实际上，存在多线程并发的情况，所以在【第 25 行】或者【第 36 行】中，我们调用 `compareAndSetNext(...)` 方法，使用 **CAS** 的方式，设置 `pred` 的下一个节点。
  - 如果设置失败，说明当前线程和其它线程竞争失败，不需要做其它逻辑，因为 `pred` 的下一个节点已经被其它线程设置成功。

- 第 21 行：设置node节点的为取消的等待状态Node.CANCELLED。

  - 这里可以使用**直接写**，而不是 CAS 。
  - 在这个操作之后，其它 Node 节点可以忽略 `node` 。

- 下面开始开始修改 `pred` 的新的下一个节点，一共分成三种情况。

- 第一种 

- 第 24 行：如果node是尾节点，调用compareAndSetTail(...)方法，CAS设置pred为新的尾节点。

  - 第 25 行：若上述操作成功，**调用 `compareAndSetNext(...)` 方法，CAS 设置 `pred` 的下一个节点为空( `null` )。**

- 第二种

- 第 30 行：`pred` 非首节点。

- 第 31 至 32 行：`pred` 的等待状态为 `Node.SIGNAL` ，或者可被 CAS 为 `Node.SIGNAL` 。

- 第 33 行：pred的线程非空。

  - 一开始 30 行为非头节点，在 33 的时候，结果成为头节点，线程已经为空了。

- 第 34 至 36 行：若 `node` 的下一个节点 `next` 的等待状态非 `Node.CANCELLED` ，则调用 `compareAndSetNext(...)` 方法，CAS设置 `pred` 的下一个节点为 `next` 。

- 第三种

- 第 37 至 39 行：如果pred为首节点( 在【第 31 至 33 行】也会有别的情况 )，调用unparkSuccessor(Node node)方法，唤醒node的下一个节点的线程等待。

  - 为什么此处需要唤醒呢？因为，`pred` 为首节点，`node` 的下一个节点的阻塞等待，需要 `node` 释放同步状态时进行唤醒。但是，`node` 取消获取同步状态，则不会再出现 `node` 释放同步状态时进行唤醒 `node` 的**下**一个节点。因此，需要此处进行唤醒。

- 第 二 + 三种

- 第 41 行：参照：

  - <http://donald-draper.iteye.com/blog/2360256>

  - `next` 的注释如下：

    ```Java
    /**
     * Link to the successor node that the current node/thread
     * unparks upon release. Assigned during enqueuing, adjusted
     * when bypassing cancelled predecessors, and nulled out (for
     * sake of GC) when dequeued.  The enq operation does not
     * assign next field of a predecessor until after attachment,
     * so seeing a null next field does not necessarily mean that
     * node is at end of queue. However, if a next field appears
     * to be null, we can scan prev's from the tail to
     * double-check.  The next field of cancelled nodes is set to
     * point to the node itself instead of null, to make life
     * easier for isOnSyncQueue.
     */
    ```

#### 1.2 独占式获取响应中断

​	AQS 提供了`acquire(int arg)` 方法，以供独占式获取同步状态，但是该方法对中断不响应，对线程进行中断操作后，该线程会依然位于CLH同步队列中，等待着获取同步状态。为了响应中断，AQS 提供了 `acquireInterruptibly(int arg)`方法。**该方法在等待获取同步状态时，如果当前线程被中断了，会立刻响应中断，并抛出 InterruptedException 异常。**

```Java
public final void acquireInterruptibly(int arg) throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (!tryAcquire(arg))
        doAcquireInterruptibly(arg);
}
```

- 首先，校验该线程是否已经中断了，如果是，则抛出InterruptedException 异常。
- 然后，调用 `tryAcquire(int arg)` 方法，尝试获取同步状态，如果获取成功，则直接返回。
- 最后，调用 `doAcquireInterruptibly(int arg)` 方法，自旋直到获得同步状态成功，**或线程中断抛出 InterruptedException 异常**。

##### 1.2.1 doAcquireInterruptibly

```java
private void doAcquireInterruptibly(int arg) throws InterruptedException {
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException(); // 直接抛出异常
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

它与 `acquire(int arg)` 方法**仅有两个差别**：

1. 方法声明抛出 InterruptedException 异常。
2. 在中断方法处不再是使用 `interrupted` 标志，而是直接抛出 InterruptedException 异常。

#### 1.3 独占式超时获取

​	AQS 除了提供上面两个方法外，还提供了一个增强版的方法 `tryAcquireNanos(int arg, long nanos)` 。该方法为 `acquireInterruptibly(int arg)` 方法的进一步增强，它除了响应中断外，还有超时控制。即如果当前线程没有在指定时间内获取同步状态，则会返回 false ，否则返回 true 。

**流程图**如下：

![流程图](/Users/jack/Desktop/md/images/2018120811002.png)

**代码**如下：

```Java
public final boolean tryAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    return tryAcquire(arg) ||
        doAcquireNanos(arg, nanosTimeout);
}
```

- 首先，校验该线程是否已经中断了，如果是，则抛出InterruptedException 异常。
- 然后，调用 `tryAcquire(int arg)` 方法，尝试获取同步状态，如果获取成功，则直接返回。
- 否则，调用 `doAcquireNanos(int arg)` 方法，自旋直到获得同步状态成功，或线程中断抛出 InterruptedException 异常，**或超过指定时间返回获取同步状态失败**。

##### 1.3.1 doAcquireNanos

```Java
static final long spinForTimeoutThreshold = 1000L;
  1: private boolean doAcquireNanos(int arg, long nanosTimeout)
  2:         throws InterruptedException {
  3:     // nanosTimeout <= 0
  4:     if (nanosTimeout <= 0L)
  5:         return false;
  6:     // 超时时间
  7:     final long deadline = System.nanoTime() + nanosTimeout;
  8:     // 新增 Node 节点
  9:     final Node node = addWaiter(Node.EXCLUSIVE);
 10:     boolean failed = true;
 11:     try {
 12:         // 自旋
 13:         for (;;) {
 14:             final Node p = node.predecessor();
 15:             // 获取同步状态成功
 16:             if (p == head && tryAcquire(arg)) {
 17:                 setHead(node);
 18:                 p.next = null; // help GC
 19:                 failed = false;
 20:                 return true;
 21:             }
 22:             /*
 23:              * 获取失败，做超时、中断判断
 24:              */
 25:             // 重新计算需要休眠的时间
 26:             nanosTimeout = deadline - System.nanoTime();
 27:             // 已经超时，返回false
 28:             if (nanosTimeout <= 0L)
 29:                 return false;
 30:             // 如果没有超时，则等待nanosTimeout纳秒
 31:             // 注：该线程会直接从LockSupport.parkNanos中返回，
 32:             // LockSupport 为 J.U.C 提供的一个阻塞和唤醒的工具类
 33:             if (shouldParkAfterFailedAcquire(p, node) &&
 34:                     nanosTimeout > spinForTimeoutThreshold)
 35:                 LockSupport.parkNanos(this, nanosTimeout);
 36:             // 线程是否已经中断了
 37:             if (Thread.interrupted())
 38:                 throw new InterruptedException();
 39:         }
 40:     } finally {
 41:         if (failed)
 42:             cancelAcquire(node);
 43:     }
 44: }
```

- 因为是在 `doAcquireInterruptibly(int arg)` 方法的基础上，做了超时控制的增强，所以相同部分，我们直接跳过。
- 第 3 至 5 行：如果超时时间小于 0 ，直接返回 false ，已经超时。
- 第 7 行：计算最终超时时间 `deadline` 。
- 第 9 行至 21 行：【相同，跳过】
- 第 26 行：重新计算剩余可获取同步状态的时间 `nanosTimeout` 。
- 第 27 至 29 行：如果剩余时间小于 0 ，直接返回 false ，已经超时。
- 第 33 行：【相同，跳过】
- 第 34 至 35 行：如果剩余时间大于 `spinForTimeoutThreshold` ，则**调用 `LockSupport的parkNanos(Object blocker, long nanos)` 方法，休眠 `nanosTimeout` 纳秒。否则，就不需要休眠了，直接进入快速自旋的过程。**原因在于，`spinForTimeoutThreshold` 已经非常小了，非常短的时间等待无法做到十分精确，如果这时再次进行超时等待，相反会让 `nanosTimeout` 的超时从整体上面表现得不是那么精确。所以，在超时非常短的场景中，AQS 会进行无条件的快速自旋。
- 第 36 至 39 行：若线程已经中断了，抛出 InterruptedException 异常。
- 第 40 至 43 行：【相同，跳过】

#### 1.4 独占式同步状态释放

​	当线程获取同步状态后，执行完相应逻辑后，就需要**释放同步状态**。AQS 提供了`release(int arg)`方法，释放同步状态。代码如下：

```Java
1: public final boolean release(int arg) {
2:     if (tryRelease(arg)) {
3:         Node h = head;
    	// waitStatus为0说明是初始化的空队列
4:         if (h != null && h.waitStatus != 0)	//如果为0表示是初始状态
    			// 唤醒后续的结点
5:             unparkSuccessor(h);
6:         return true;
7:     }
8:     return false;
9: }
```

- 第 2 行：调用 `tryRelease(int arg)` 方法，去尝试释放同步状态，释放成功则设置锁状态并返回 true ，否则获取失败，返回 false 。

  - `tryRelease(int arg)` 方法，**需要自定义同步组件自己实现，该方法必须要保证线程安全的释放同步状态**。代码如下：

    ```Java
    protected boolean tryRelease(int arg) {
        throw new UnsupportedOperationException();
    }
    ```

    - 直接抛出 UnsupportedOperationException 异常。

- 第 3 行：获得当前的 `head` ，避免并发问题。

- 第 4 行：头结点不为空，并且头结点状态不为 0 ( `INITIAL` 未初始化)。为什么会出现 0 的情况呢？老艿艿的想法是，以 ReentrantReadWriteLock ( 内部基于 AQS 实现 ) 举例子：

  - 线程 A 和线程 B ，都获取了读锁。

  - > 老艿艿：如上是我的猜想，并未实际验证。如果不正确，或者有其他情况，欢迎斧正。

- 第 5 行：调用 `unparkSuccessor(Node node)` 方法，唤醒下一个节点的线程等待。见下面的相关解析

#### 1.5 总结

> 在 AQS 中维护着一个 FIFO 的同步队列。
>
> - 当线程获取同步状态失败后，则会加入到这个 CLH 同步队列的队尾，并一直保持着自旋。
> - 在 CLH 同步队列中的线程在自旋时，会判断其前驱节点是否为首节点，如果为首节点则不断尝试获取同步状态，获取成功则退出CLH同步队列。
> - 当线程执行完逻辑后，会释放同步状态，释放后会唤醒其后继节点。

### 2. 共享式

共享式与独占式的最主要区别在于，同一时刻：

- 独占式只能有一个线程获取同步状态。
- 共享式可以有多个线程获取同步状态。

例如，读操作可以有多个线程同时进行，而写操作同一时刻只能有一个线程进行写操作，其他操作都会被阻塞。参见 ReentrantReadWriteLock 。

#### 2.1 共享式同步状态获取

> `acquireShared(int arg)` 方法，对标 `acquire(int arg)` 方法。

```Java
1: public final void acquireShared(int arg) {
2:     if (tryAcquireShared(arg) < 0)	//只有cancel状态才小于0，它是因为超时或者中断，节点会被设置为取消状态，被取消的节点时不会参与到竞争中的，他会一直保持取消状态不会转变为其他状态
3:         doAcquireShared(arg);
4: }
```

- 第 2 行：调用 `tryAcquireShared(int arg)` 方法，尝试获取同步状态，获取成功则设置锁状态并返回大于等于 0 ，否则获取失败，返回小于 0 。若获取成功，直接返回，不用线程阻塞，自旋直到获得同步状态成功。

  - `tryAcquireShared(int arg)` 方法，**需要自定义同步组件自己实现，该方法必须要保证线程安全的获取同步状态。**代码如下：

    ```
    protected int tryAcquireShared(int arg) {
        throw new UnsupportedOperationException();
    }
    ```

    - **直接**抛出 UnsupportedOperationException 异常。

##### 2.1.1 doAcquireShared

```Java
 1: private void doAcquireShared(int arg) {
 2:     // 共享式节点，在队尾新增节点
 3:     final Node node = addWaiter(Node.SHARED);
 4:     boolean failed = true;
 5:     try {
 6:         boolean interrupted = false;
 7:         for (;;) {
 8:             // 获取node的前驱节点
 9:             final Node p = node.predecessor();
10:             // 如果其前驱节点，获取同步状态
11:             if (p == head) {
12:                 // 尝试获取同步
13:                 int r = tryAcquireShared(arg);
14:                 if (r >= 0) {
15:                     setHeadAndPropagate(node, r);
16:                     p.next = null; // help GC
17:                     if (interrupted)
18:                         selfInterrupt();
19:                     failed = false;
20:                     return;
21:                 }
22:             }
23:             if (shouldParkAfterFailedAcquire(p, node) &&
24:                     parkAndCheckInterrupt())
25:                 interrupted = true;
26:         }
27:     } finally {
28:         if (failed)
29:             cancelAcquire(node);
30:     }
31: }
```

- 因为和 `acquireQueued(int arg)` 方法的基础上，所以**相同部分，我们直接跳过**。

- 第 3 行：调用 `addWaiter(Node mode)` 方法，将当前线程加入到 CLH 同步队列尾部。并且， `mode` 方法参数为 `Node.SHARED` ，表示**共享**模式。

- 第 6 行：【相同，跳过】

- 第 9 至 22 行：【大体相同，部分跳过】

  - 第 13 行：调用 `tryAcquireShared(int arg)` 方法，尝试获得同步状态。 在 `acquireShared(int arg)` 方法的【第 2 行】，也调用了这个方法。

  - 第 15 行：调用setHeadAndPropagate(Node node, int propagate)方法，设置新的首节点，并根据条件

    ，唤醒下一个节点。

    - 这里和**独占式**同步状态获取很大的不同：==通过这样的方式，不断唤醒下一个**共享式**同步状态， 从而实现同步状态被**多个**线程的**共享获取**==。

  - 第 17 至 18 行：和 `acquire(int arg)` 方法，对于**线程中断**的处理方式相同，只是代码放置的位置不同。

- 第 23 至 25 行：【相同，跳过】

- 第 27 至 30 行：【相同，跳过】

##### 2.1.2 setHeadAndPropagate

```Java
 1: private void setHeadAndPropagate(Node node, int propagate) {
 2:     Node h = head; // Record old head for check below
 3:     setHead(node);
 4:     /*
 5:      * Try to signal next queued node if:
 6:      *   Propagation was indicated by caller,
 7:      *     or was recorded (as h.waitStatus either before
 8:      *     or after setHead) by a previous operation
 9:      *     (note: this uses sign-check of waitStatus because
10:      *      PROPAGATE status may transition to SIGNAL.)
11:      * and
12:      *   The next node is waiting in shared mode,
13:      *     or we don't know, because it appears null
14:      *
15:      * The conservatism in both of these checks may cause
16:      * unnecessary wake-ups, but only when there are multiple
17:      * racing acquires/releases, so most need signals now or soon
18:      * anyway.
19:      */
20:     if (propagate > 0 || h == null || h.waitStatus < 0 ||
21:         (h = head) == null || h.waitStatus < 0) {
22:         Node s = node.next;
23:         if (s == null || s.isShared())
24:             doReleaseShared();
25:     }
26: }
```

- 第 2 行：记录原来的首节点 `h` 。
- 第 3 行：调用 `setHead(Node node)` 方法，设置 `node` 为**新**的**首**节点。
- 第 20 行：==`propagate > 0` 代码块，说明同步状态还能被其他线程获取。==
- 第 20 至 21 行：判断**原来**的或者**新**的**首**节点，**等待状态**为 `Node.PROPAGATE` 或者 `Node.SIGNAL` 时，可以继续向下**唤醒**。
- 第 23 行：调用 `Node#isShared()` 方法，判断**下**一个节点为**共享式**获取同步状态。
- 第 24 行：调用 `#doReleaseShared()` 方法，唤醒后续的**共享式**获取同步状态的节点。

#### 2.2 共享式获取响应中断

`acquireSharedInterruptibly(int arg)` 方法，代码如下：

```Java
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}

private void doAcquireSharedInterruptibly(int arg)
    throws InterruptedException {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

- 与 [「1.2 独占式获取响应中断」](http://www.iocoder.cn/JUC/sike/aqs-2/?vip#) 类似，就不重复解析了。

#### 2.3 共享式超时获取

`tryAcquireSharedNanos(int arg, long nanosTimeout)` 方法，代码如下：

```java
public final boolean tryAcquireSharedNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    return tryAcquireShared(arg) >= 0 ||
        doAcquireSharedNanos(arg, nanosTimeout);
}

private boolean doAcquireSharedNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (nanosTimeout <= 0L)
        return false;
    final long deadline = System.nanoTime() + nanosTimeout;
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return true;
                }
            }
            nanosTimeout = deadline - System.nanoTime();
            if (nanosTimeout <= 0L)
                return false;
            if (shouldParkAfterFailedAcquire(p, node) &&
                nanosTimeout > spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

- 与 [「1.3 独占式超时获取」](http://www.iocoder.cn/JUC/sike/aqs-2/?vip#) 类似，就不重复解析了。

#### 2.4 共享式同步状态释放

​	当线程获取同步状态后，执行完相应逻辑后，就需要**释放同步状态**。AQS 提供了`releaseShared(int arg)`方法，释放同步状态。代码如下：

```Java
1: public final boolean releaseShared(int arg) {
2:     if (tryReleaseShared(arg)) {
3:         doReleaseShared();
4:         return true;
5:     }
6:     return false;
7: }
```

- 第 2 行：调用 `#tryReleaseShared(int arg)` 方法，去尝试释放同步状态，释放成功则设置锁状态并返回 true ，否则获取失败，返回 false 。同时，它们分别对应【第 3 至 5】和【第 6 行】的逻辑。

  - `#tryReleaseShared(int arg)` 方法，**需要**自定义同步组件**自己实现**，该方法必须要保证**线程安全**的释放同步状态。代码如下：

    ```
    protected boolean tryReleaseShared(int arg) {
        throw new UnsupportedOperationException();
    }
    ```

    - **直接**抛出 UnsupportedOperationException 异常。

- 第 3 行：调用 `#doReleaseShared()` 方法，唤醒后续的**共享式**获取同步状态的节点。

##### 2.4.1 doReleaseShared

```Java
 1: private void doReleaseShared() {
 2:     /*
 3:      * Ensure that a release propagates, even if there are other
 4:      * in-progress acquires/releases.  This proceeds in the usual
 5:      * way of trying to unparkSuccessor of head if it needs
 6:      * signal. But if it does not, status is set to PROPAGATE to
 7:      * ensure that upon release, propagation continues.
 8:      * Additionally, we must loop in case a new node is added
 9:      * while we are doing this. Also, unlike other uses of
10:      * unparkSuccessor, we need to know if CAS to reset status
11:      * fails, if so rechecking.
12:      */
13:     for (;;) {
14:         Node h = head;
15:         if (h != null && h != tail) {
16:             int ws = h.waitStatus;
17:             if (ws == Node.SIGNAL) {
18:                 if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
19:                     continue;            // loop to recheck cases
20:                 unparkSuccessor(h);
21:             }
22:             else if (ws == 0 &&
23:                      !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
24:                 continue;                // loop on failed CAS
25:         }
26:         if (h == head)                   // loop if head changed
27:             break;
28:     }
29: }
```

- doReleaseShared 的详细逻辑。可参考博客：<http://zhanjindong.com/2015/03/15/java-concurrent-package-aqs-AbstractQueuedSynchronizer>

## 1.4 阻塞和唤醒线程

### 1. parkAndCheckInterrupt

​	**在线程获取同步状态时，如果获取失败，则加入 CLH 同步队列，通过通过自旋的方式不断获取同步状态，但是在自旋的过程中，则需要判断当前线程是否需要阻塞，其主要方法在`acquireQueued(int arg)`** ，代码如下：

```Java
// ... 省略前面无关代码
if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
// ... 省略前面无关代码
```

- 通过这段代码我们可以看到，在获取同步状态失败后，线程并不是立马进行阻塞，需要检查该线程的状态，**检查状态的方法为 `shouldParkAfterFailedAcquire(Node pred, Node node)`方法，该方法主要靠前驱节点判断当前线程是否应该被阻塞。可以看上面的相关介绍。**

- 如果 `shouldParkAfterFailedAcquire(Node pred, Node node)` 方法返回 true，**则调用`parkAndCheckInterrupt()` 方法，阻塞当前线程**。代码如下：

  ```Java
  private final boolean parkAndCheckInterrupt() {
          LockSupport.park(this);//挂起当前的线程
          return Thread.interrupted();//如果当前线程已经被中断了，返回true
      }
  ```

  - 开始，调用 `LockSupport.park(Object blocker)` 方法，将当前线程挂起，此时就进入阻塞等待唤醒的状态。
  - 然后，在线程被唤醒时，调用Thread.interrupted()方法，返回当前线程是否被打断，并清理打断状态。所以，实际上，线程被唤醒有两种情况：
    - 第一种，当前节点(线程)的**前序节点**释放同步状态时，唤醒了该线程。详细解析，见下面
    - 第二种，当前线程被打断导致唤醒。

### 2. unparkSuccessor

当线程释放同步状态后，则需要唤醒该线程的后继节点。代码如下：

```Java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h); // 唤醒后继节点
        return true;
    }
    return false;
}
```

- 调用 `unparkSuccessor(Node node)` 方法，唤醒后继节点：

  ```Java
  private void unparkSuccessor(Node node) {
      //当前节点状态
      int ws = node.waitStatus;
      //当前状态 < 0 则设置为 0
      if (ws < 0)
          compareAndSetWaitStatus(node, ws, 0);
  
      //当前节点的后继节点
      Node s = node.next;
      //后继节点为null或者其状态 > 0 (超时或者被中断了)
      if (s == null || s.waitStatus > 0) {
          s = null;
          //从tail节点来找可用节点
          for (Node t = tail; t != null && t != node; t = t.prev)
              if (t.waitStatus <= 0)
                  s = t;
      }
      //唤醒后继节点
      if (s != null)
          LockSupport.unpark(s.thread);
  }
  ```

  - 可能会存在当前线程的后继节点为 `null`，例如：超时、被中断的情况。如果遇到这种情况了，则需要跳过该节点。
    - 但是，为何是从 `tail` 尾节点开始，而不是从 `node.next` 开始呢？原因在于，取消的 `node.next.next` 指向的是 `node.next` 自己(去掉一个节点后就是自己了)。如果顺序遍历下去，会导致死循环。所以此时，只能采用 `tail` 回溯的办法，找到第一个( 不是最新找到的，而是最前序的 )可用的线程。
    - 再但是，为什么取消的 `node.next.next` 指向的是 `node.next` 自己呢？在 `cancelAcquire(Node node)` 的末尾，`node.next = node;` 代码块，取消的 `node` 节点，将其 `next` 指向了自己。
  - 最后，调用 `LockSupport的unpark(Thread thread)` 方法，唤醒该线程。

### 3. LockSupport

> LockSupport 是用来创建锁和其他同步类的基本线程阻塞原语。

==每个使用 LockSupport 的线程都会与一个许可与之关联：==

- 如果该许可可用，并且可在进程中使用，则调用 `park(...)` 将会立即返回，否则可能阻塞。
- 如果许可尚不可用，则可以调用 `unpark(...)` 使其可用。
- 但是，注意许可不可重入，也就是说只能调用一次 `park(...)` 方法，否则会一直阻塞。

LockSupport 定义了一系列以 `park` 开头的方法来阻塞当前线程，`unpark(Thread thread)` 方法来唤醒一个被阻塞的线程。如下图所示：

![方法](/Users/jack/Desktop/md/images/2018120812001.png)

- `park(Object blocker)` 方法的blocker参数，主要是用来标识当前线程在等待的对象，该对象主要用于**问题排查和系统监控**。
- park 方法和 `unpark(Thread thread)` 方法，都是**成对出现**的。同时 `unpark(Thread thread)` 方法，必须要在 park 方法执行之后执行。当然，并不是说没有调用 `unpark(Thread thread)` 方法的线程就会一直阻塞，park 有一个方法，它是带了时间戳的 `#parkNanos(long nanos)` 方法：为了线程调度禁用当前线程，最多等待指定的等待时间，除非许可可用。

#### 3.1 park

```java
public static void park() {
    UNSAFE.park(false, 0L);
}
```

#### 3.2 unpark

```java
public static void unpark(Thread thread) {
    if (thread != null)
        UNSAFE.unpark(thread);
}
```

#### 3.3 实现原理

从上面可以看出，其内部的实现都是通过 `sun.misc.Unsafe` 来实现的，其定义如下：

```Java
// UNSAFE.java
public native void park(boolean var1, long var2);
public native void unpark(Object var1);
```



参照下面的几篇文章：

- [《【死磕 Java 并发】—– J.U.C 之 AQS：AQS 简介》](http://www.iocoder.cn/JUC/sike/aqs-0-intro?vip)
- [《【死磕 Java 并发】—– J.U.C 之 AQS：CLH 同步队列》](http://www.iocoder.cn/JUC/sike/aqs-1-clh?vip)
- [《【死磕 Java 并发】—– J.U.C 之 AQS：同步状态的获取与释放》](http://www.iocoder.cn/JUC/sike/aqs-2?vip)
- [《【死磕 Java 并发】—– J.U.C 之 AQS：阻塞和唤醒线程》](http://www.iocoder.cn/JUC/sike/aqs-3?vip)



# 2、什么是可重入锁及ReentrantLock

举例来说明锁的可重入性。代码如下：

```java
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

- `outer()` 方法中调用了 `inner()` 方法，`outer()` 方法先锁住了 `lock` ，这样 `inner()` 就不能再获取 `lock` 。
- 其实调用 `outer()` 方法的线程已经获取了 `lock` 锁，但是不能在 `inner()` 方法中重复利用已经获取的锁资源，这种锁即称之为不可重入。
- 可重入就意味着：线程可以进入任何一个它已经拥有的锁所同步着的代码块。

`synchronized`、ReentrantLock 都是可重入的锁，可重入锁相对来说简化了并发编程的开发。

## ReentrantLock

### 1. 简介

​	==ReentrantLock，可重入锁，是一种递归无阻塞的同步机制。它可以等同于 `synchronized` 的使用==，但是 ReentrantLock 提供了比 `synchronized` 更强大、灵活的锁机制，可以减少死锁发生的概率。

> 一个可重入的互斥锁定 Lock，它具有与使用 `synchronized` 方法和语句所访问的隐式监视器锁定相同的一些基本行为和语义，但功能更强大。
>
> ReentrantLock 将由最近成功获得锁定，并且还没有释放该锁定的线程所拥有。当锁定没有被另一个线程所拥有时，调用 lock 的线程将成功获取该锁定并返回。如果当前线程已经拥有该锁定，此方法将立即返回。可以使用 `isHeldByCurrentThread()` 和 `getHoldCount()` 方法来检查此情况是否发生。

​	ReentrantLock 还提供了**公平锁**和**非公平锁**的选择，通过构造方法接受一个可选的 `fair` 参数（**默认非公平锁）：当设置为 true 时，表示公平锁；否则为非公平锁。**

​	公平锁与非公平锁的区别在于，公平锁的锁获取是有顺序的。但是公平锁的效率往往没有非公平锁的效率高，在许多线程访问的情况下，公平锁表现出较低的吞吐量。

ReentrantLock 整体结构如下图：

![201702010001](/Users/jack/Desktop/md/images/2018120813001.png)

- ReentrantLock 实现 Lock 接口，基于内部的 Sync 实现。
- Sync 实现 AQS ，提供了 FairSync 和 NonFairSync 两种实现。

### 2. Sync 抽象类

​	Sync 是 ReentrantLock 的内部静态类，实现 AbstractQueuedSynchronizer 抽象类，同步器抽象类。它使用 AQS 的 `state` 字段，来表示当前锁的持有数量，从而实现可重入的特性。

```java
    /**
     * 该锁同步控制的一个基类.下边有两个子类：非公平机制和公平机制.使用了AbstractQueuedSynchronizer类的
     */
    static abstract class Sync extends AbstractQueuedSynchronizer
```

#### 2.1 lock

基于CAS尝试将state（锁数量）从0设置为1

A、如果设置成功，设置当前线程为独占锁的线程；

B、如果设置失败，还会再获取一次锁数量，

B1、如果锁数量为0，再基于CAS尝试将state（锁数量）从0设置为1一次，如果设置成功，设置当前线程为独占锁的线程；

B2、如果锁数量不为0或者上边的尝试又失败了，查看当前线程是不是已经是独占锁的线程了，如果是，则将当前的锁数量+1；如果不是，则将该线程封装在一个Node内，并加入到等待队列中去。等待被其前一个线程节点唤醒。

```java
/**
 * Performs {@link Lock#lock}. The main reason for subclassing
 * is to allow fast path for nonfair version.
 */
abstract void lock();
```

- **执行锁。抽象了该方法的原因是，允许子类实现快速获得非公平锁的逻辑。**

#### 2.2 nonfairTryAcquire

`nonfairTryAcquire(int acquires)` 方法，**非公平锁**的方式获得锁。代码如下：

```
final boolean nonfairTryAcquire(int acquires) {
    //当前线程
    final Thread current = Thread.currentThread();
    //获取同步状态
    int c = getState();
    //state == 0,表示没有该锁处于空闲状态
    if (c == 0) {
        //获取锁成功，设置为当前线程所有
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    //线程重入
    //判断锁持有的线程是否为当前线程
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

- 该方法主要逻辑：首先判断同步状态state == 0?
  - **如果是，表示该锁还没有被线程持有，直接通过CAS获取同步状态。**
    - 如果成功，返回 true 。
    - 否则，返回 false 。
  - 如果不是，则判断当前线程是否为获取锁的线程?
    - 如果是，则获取锁，成功返回 true 。成功获取锁的线程，再次获取锁，这是增加了同步状态 `state` 。通过这里的实现，我们可以看到上面提到的 “==它使用 AQS 的 `state` 字段，来表示当前锁的持有数量，从而实现可重入的特性==”。
    - 否则，返回 false 。
- 理论来说，这个方法应该在子类 **FairSync** 中实现，但是为什么会在这里呢？在下文的 `ReentrantLock.tryLock()`中，详细解析。

#### 2.3 tryRelease

`tryRelease(int releases)` 实现方法，释放锁。代码如下：

```
protected final boolean tryRelease(int releases) {
    // 减掉releases
    int c = getState() - releases;
    // 如果释放的不是持有锁的线程，抛出异常
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    // state == 0 表示已经释放完全了，其他线程可以获取同步状态了
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

- 通过判断判断是否为获得到锁的线程，保证该方法线程安全。
- 只有当同步状态彻底释放后，该方法才会返回 true 。**当 `state == 0` 时，则将锁持有线程设置为 null** ，`free= true`，表示释放成功。

#### 2.4 其他实现方法

```
// 是否当前线程独占
@Override
protected final boolean isHeldExclusively() {
    // While we must in general read state before owner,
    // we don't need to do so to check if current thread is owner
    return getExclusiveOwnerThread() == Thread.currentThread();
}

// 新生成条件
final ConditionObject newCondition() {
    return new ConditionObject();
}

// Methods relayed from outer class

// 获得占用同步状态的线程
final Thread getOwner() {
    return getState() == 0 ? null : getExclusiveOwnerThread();
}

// 获得当前线程持有锁的数量
final int getHoldCount() {
    return isHeldExclusively() ? getState() : 0;
}

// 是否被锁定
final boolean isLocked() {
    return getState() != 0;
}

/**
 * Reconstitutes the instance from a stream (that is, deserializes it).
 * 自定义反序列化逻辑
 */
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    s.defaultReadObject();
    setState(0); // reset to unlocked state
}
```

- 从这些方法中，我们可以看到，ReentrantLock 是独占获取同步状态的模式。

### 3. Sync 实现类

#### 3.1 NonfairSync

##### 获取锁的步骤

基于CAS尝试将state（锁数量）从0设置为1

A、如果设置成功，设置当前线程为独占锁的线程；

B、如果设置失败，还会再获取一次锁数量，

B1、如果锁数量为0，再基于CAS尝试将state（锁数量）从0设置为1一次，如果设置成功，设置当前线程为独占锁的线程；

B2、如果锁数量不为0或者上边的尝试又失败了，查看当前线程是不是已经是独占锁的线程了，如果是，则将当前的锁数量+1；如果不是，则将该线程封装在一个Node内，并加入到等待队列中去。等待被其前一个线程节点唤醒。

NonfairSync 是 ReentrantLock 的内部静态类，实现 Sync 抽象类，非公平锁实现类。

##### 3.1.1 lock

`lock()` 实现方法，首先基于 AQS `state` 进行 CAS操作，将 `0 设置为1` 。若成功，则获取锁成功。若失败，执行 AQS 的正常的同步状态获取逻辑。代码如下：

```java
/**
 * 1）首先基于CAS将state（锁数量）从0设置为1，如果设置成功，设置当前线程为独占锁的线程；-->请求成功-->第一次插队
 * 2）如果设置失败(即当前的锁数量可能已经为1了，即在尝试的过程中，已经被其他线程先一步占有了锁)，这个时候当前线程执行acquire(1)方法
 	//获取锁的方法
    public final void acquire(int arg) {
        if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();//中断自己
    }

  * 2.1）acquire(1)方法首先调用下边的tryAcquire(1)方法，在该方法中，首先获取锁数量状态，
  * 2.1.1）如果为0(证明该独占锁已被释放，当下没有线程在使用)，这个时候我们继续使用CAS将state（锁数量）从0设置为1，如果设置成功，当前线程独占锁；-->请求成功-->第二次插队；当然，如果设置不成功，直接返回false
  * 2.2.2）如果不为0，就去判断当前的线程是不是就是当下独占锁的线程，如果是，就将当前的锁数量状态值+1（这也就是可重入锁的名称的来源）-->请求成功
      * 
      * 下边的流程一句话：请求失败后，将当前线程链入队尾并挂起，之后等待被唤醒。
      * 
  * 2.2.3）如果最后在tryAcquire(1)方法中上述的执行都没成功，即请求没有成功，则返回false，继续执行acquireQueued(addWaiter(Node.EXCLUSIVE), arg)方法
  * 2.2）在上述方法中，首先会使用addWaiter(Node.EXCLUSIVE)将当前线程封装进Node节点node，然后将该节点加入等待队列（先快速入队，如果快速入队不成功，其使用正常入队方法无限循环(即自旋)一直到Node节点入队为止）
  * 2.2.1）快速入队：如果同步等待队列存在尾节点，将使用CAS(compareAndSetTail(pred, node))尝试将尾节点设置为node，并将之前的尾节点插入到node之前(即Node pred = tail;node.prev = pred;)
  * 2.2.2）正常入队：如果同步等待队列不存在尾节点或者上述CAS尝试不成功的话，就执行正常入队（enq(node);该方法是一个无限循环的过程，即直到入队为止）-->第一次阻塞
  * 2.2.2.1）如果尾节点为空（初始化同步等待队列），创建一个dummy节点，并将该节点通过CAS尝试设置到头节点上去，设置成功的话，将尾节点也指向该dummy节点（即头节点和尾节点都指向该dummy节点）
  * 2.2.2.1）如果尾节点不为空，执行与快速入队相同的逻辑，即使用CAS尝试将尾节点设置为node，并将之前的尾节点插入到node之前
 * 最后，如果顺利入队的话，就返回入队的节点node，如果不顺利的话，无限循环去执行2.2)下边的流程，直到入队为止
 * 2.3）node节点入队之后，就去执行acquireQueued(final Node node, int arg)（这又是一个无限循环的过程，这里需要注意的是，无限循环等于阻塞，多个线程可以同时无限循环--每个线程都可以执行自己的循环，这样才能使在后边排队的节点不断前进）
 * 2.3.1）获取node的前驱节点p，如果p是头节点，就继续使用tryAcquire(1)方法去尝试请求成功，-->第三次插队（当然，这次插队不一定不会使其获得执行权，请看下边一条），
 * 2.3.1.1）如果第一次请求就成功，不用中断自己的线程，如果是之后的循环中将线程挂起之后又请求成功了，使用selfInterrupt()中断自己
 * （注意p==head&&tryAcquire(1)成功是唯一跳出循环的方法，在这之前会一直阻塞在这里，直到其他线程在执行的过程中，不断的将p的前边的节点减少，直到p成为了head且node请求成功了--即node被唤醒了，才退出循环）
 * 2.3.1.2）如果p不是头节点，或者tryAcquire(1)请求不成功，就去执行shouldParkAfterFailedAcquire(Node pred, Node node)来检测当前节点是不是可以安全的被挂起，
 * 2.3.1.2.1）如果node的前驱节点pred的等待状态是SIGNAL（即可以唤醒下一个节点的线程的状态），则node节点的线程可以安全挂起，返回true，执行2.3.1.3）
 * 2.3.1.2.2）如果node的前驱节点pred的等待状态是CANCELLED，则pred的线程被取消了，我们会将pred之前的连续几个被取消的前驱节点从队列中剔除，返回false（即不能挂起），如下：
 do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
 
 之后继续执行2.3）中上述的代码的cancelAcquire(node);
 * 2.3.1.2.3）如果node的前驱节点pred的等待状态是除了上述两种的其他状态，则使用CAS尝试将前驱节点的等待状态设为SIGNAL，并返回false（因为CAS可能会失败，这里不管失败与否，都返回false，下一次执行该方法的之后，pred的等待状态就是SIGNAL了），之后继续执行2.3）中上述的代码的cancelAcquire(node);
 * 2.3.1.3）如果可以安全挂起，就执行parkAndCheckInterrupt()挂起当前线程，之后，继续执行2.3）中之前的代码
* 最后，直到该节点的前驱节点p之前的所有节点都执行完毕为止，我们的p成为了头节点，并且tryAcquire(1)请求成功，跳出循环，去执行。
 * （在p变为头节点之前的整个过程中，我们发现这个过程是不会被中断的）
 * 2.3.2）当然在2.3.1）中产生了异常，我们就会执行cancelAcquire(Node node)取消node的获取锁的意图。
*/
        final void lock() {
            if (compareAndSetState(0, 1))//如果CAS尝试成功
                setExclusiveOwnerThread(Thread.currentThread());//设置当前线程为独占锁的线程
            else
                acquire(1);
        }
```

- 优先基于 AQS `state` 进行 CAS 操作，已经能体现出非公平锁的特点。因为，此时有可能有 N + 1 个线程正在获得锁，其中 1 个线程已经获得到锁，释放的瞬间，恰好被新的线程抢夺到，而不是排队的 N 个线程。

##### 3.1.2 tryAcquire

`tryAcquire(int acquires)` 实现方法，非公平的方式，获得同步状态。代码如下：

```
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
```

- 直接调用 `nonfairTryAcquire(int acquires)` 方法，非公平锁的方式获得锁。

```java
/**
         * 非公平锁中被tryAcquire调用
         */
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();//获取当前线程
            int c = getState();//获取锁数量
            if (c == 0) {//如果锁数量为0，证明该独占锁已被释放，当下没有线程在使用
                if (compareAndSetState(0, acquires)) {//继续通过CAS将state由0变为1，注意这里传入的acquires为1
                    setExclusiveOwnerThread(current);//将当前线程设置为独占锁的线程
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {//查看当前线程是不是就是独占锁的线程
                int nextc = c + acquires;//如果是，锁状态的数量为当前的锁数量+1
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);//设置当前的锁数量
                return true;
            }
            return false;
        }
```

#### 3.2 FairSync

FairSync 是 ReentrantLock 的内部静态类，实现 Sync 抽象类，公平锁实现类。

##### lock的步骤：获取一次锁数量，

B1、如果锁数量为0，如果当前线程是等待队列中的头节点，基于CAS尝试将state（锁数量）从0设置为1一次，如果设置成功，设置当前线程为独占锁的线程；

B2、如果锁数量不为0或者当前线程不是等待队列中的头节点或者上边的尝试又失败了，查看当前线程是不是已经是独占锁的线程了，如果是，则将当前的锁数量+1；如果不是，则将该线程封装在一个Node内，并加入到等待队列中去。等待被其前一个线程节点唤醒。

##### 3.2.1 lock

`lock()` 实现方法，代码如下：

```
final void lock() {
    acquire(1);
}
```

- 直接执行 AQS 的正常的同步状态获取逻辑。

##### 3.2.2 tryAcquire

`tryAcquire(int acquires)` 实现方法，公平的方式，获得同步状态。代码如下：

```java
 	 /**
         * 获取公平锁的方法
         * 1）获取锁数量c
         * 1.1)如果c==0，如果当前线程是等待队列中的头节点，使用CAS将state（锁数量）从0设置为1，如果设置成功，当前线程独占锁-->请求成功
         * 1.2)如果c!=0，判断当前的线程是不是就是当下独占锁的线程，如果是，就将当前的锁数量状态值+1（这也就是可重入锁的名称的来源）-->请求成功
         * 最后，请求失败后，将当前线程链入队尾并挂起，之后等待被唤醒。
         */
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() && // <1>
                compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

​	比较非公平锁和公平锁获取同步状态的过程，会发现两者**唯一的区别**就在于，公平锁在获取同步状态时多了一个限制条件 `<1>` 处的 `hasQueuedPredecessors()` 方法，是否有**前序**节点，如果返回true，即表示自己不是首个**等待**获取同步状态的节点。代码如下：

```
// AbstractQueuedSynchronizer.java
public final boolean hasQueuedPredecessors() {
    Node t = tail;  //尾节点
    Node h = head;  //头节点
    Node s;

    //头节点 != 尾节点
    //同步队列第一个节点不为null
    //当前线程是同步队列第一个节点
    return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

- 该方法主要做一件事情：**主要是判断当前线程是否位于 CLH 同步队列中的第一个。如果是则返回 true ，否则返回 false 。**

##### **总结：公平锁与非公平锁对比**

- FairSync：lock()少了插队部分（即少了CAS尝试将state从0设为1，进而获得锁的过程）
- FairSync：tryAcquire(int acquires)多了需要判断当前线程是否在等待队列首部的逻辑（实际上就是少了再次插队的过程，但是CAS获取还是有的）。

最后说一句，

- ReentrantLock是基于AbstractQueuedSynchronizer实现的，AbstractQueuedSynchronizer可以实现独占锁也可以实现共享锁，ReentrantLock只是使用了其中的独占锁模式

### 4. Lock 接口

`java.util.concurrent.locks.Lock` 接口，定义方法如下：

```
void lock();
void lockInterruptibly() throws InterruptedException;
boolean tryLock();
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
void unlock();
Condition newCondition();
```

- > FROM 《Java并发编程的艺术》的 [「5.1 Lock 接口」](http://www.iocoder.cn/JUC/sike/ReentrantLock/?vip#) 章节。
  >
  > ![方法解释](/Users/jack/Desktop/md/images/Lock-01.png)

### 5. ReentrantLock

`java.util.concurrent.locks.ReentrantLock` ，实现 Lock 接口，重入锁。

ReentrantLock 的实现方法，基本是对 Sync 的调用。

#### 5.1 构造方法

```
public ReentrantLock() {
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

- 基于 `fair` 参数，创建 FairSync 还是 NonfairSync 对象。

#### 5.2 lock

```java
/**
     *获取一个锁
     *三种情况：
     *1、如果当下这个锁没有被任何线程（包括当前线程）持有，则立即获取锁，锁数量==1，之后再执行相应的业务逻辑
     *2、如果当前线程正在持有这个锁，那么锁数量+1，之后再执行相应的业务逻辑
     *3、如果当下锁被另一个线程所持有，则当前线程处于休眠状态，直到获得锁之后，当前线程被唤醒，锁数量==1，再执行相应的业务逻辑
     */
public void lock() {
    sync.lock();	//调用NonfairSync（非公平锁）或FairSync（公平锁）的lock()方法
}

//获取公平锁
/**
     *获取一个锁
     *三种情况：
     *1、如果当下这个锁没有被任何线程（包括当前线程）持有，则立即获取锁，锁数量==1，之后被唤醒再执行相应的业务逻辑
     *2、如果当前线程正在持有这个锁，那么锁数量+1，之后被唤醒再执行相应的业务逻辑
     *3、如果当下锁被另一个线程所持有，则当前线程处于休眠状态，直到获得锁之后，当前线程被唤醒，锁数量==1，再执行相应的业务逻辑
     */
```

#### 5.3 lockInterruptibly

```
@Override
public void lockInterruptibly() throws InterruptedException {
    sync.acquireInterruptibly(1);	
}
```

#### 5.4 tryLock

```
/**
 * Acquires the lock only if it is not held by another thread at the time
 * of invocation.
 *
 * <p>Acquires the lock if it is not held by another thread and
 * returns immediately with the value {@code true}, setting the
 * lock hold count to one. Even when this lock has been set to use a
 * fair ordering policy, a call to {@code tryLock()} <em>will</em>
 * immediately acquire the lock if it is available, whether or not
 * other threads are currently waiting for the lock.
 * This &quot;barging&quot; behavior can be useful in certain
 * circumstances, even though it breaks fairness. If you want to honor
 * the fairness setting for this lock, then use
 * {@link #tryLock(long, TimeUnit) tryLock(0, TimeUnit.SECONDS) }
 * which is almost equivalent (it also detects interruption).
 *
 * <p>If the current thread already holds this lock then the hold
 * count is incremented by one and the method returns {@code true}.
 *
 * <p>If the lock is held by another thread then this method will return
 * immediately with the value {@code false}.
 *
 * @return {@code true} if the lock was free and was acquired by the
 *         current thread, or the lock was already held by the current
 *         thread; and {@code false} otherwise
 */
@Override
public boolean tryLock() {
    return sync.nonfairTryAcquire(1);
}
```

- `tryLock()` **实现**方法，在实现时，希望能**快速的**获得是否能够获得到锁，因此即使在设置为 `fair = true` ( 使用公平锁 )，依然调用 `Sync.nonfairTryAcquire(int acquires)` 方法。
- 如果**真的**希望 `#tryLock()` 还是按照是否公平锁的方式来，可以调用 `#tryLock(0, TimeUnit)` 方法来实现。

#### 5.5 tryLock

```
@Override
public boolean tryLock(long timeout, TimeUnit unit)
        throws InterruptedException {
    return sync.tryAcquireNanos(1, unit.toNanos(timeout));
}
```

#### 5.6 unlock

```
@Override
public void unlock() {
    sync.release(1);
}
```

#### 5.7 newCondition

```
@Override
public Condition newCondition() {
    return sync.newCondition();
}
```

#### 5.8 其他实现方法

```
public int getHoldCount() {
    return sync.getHoldCount();
}
public boolean isHeldByCurrentThread() {
    return sync.isHeldExclusively();
}
public boolean isLocked() {
    return sync.isLocked();
}


public final boolean isFair() {
    return sync instanceof FairSync;
}

protected Thread getOwner() {
    return sync.getOwner();
}
public final boolean hasQueuedThreads() {
    return sync.hasQueuedThreads();
}
public final boolean hasQueuedThread(Thread thread) {
    return sync.isQueued(thread);
}
public final int getQueueLength() {
    return sync.getQueueLength();
}
protected Collection<Thread> getQueuedThreads() {
    return sync.getQueuedThreads();
}

public boolean hasWaiters(Condition condition) {
    if (condition == null)
        throw new NullPointerException();
    if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
        throw new IllegalArgumentException("not owner");
    return sync.hasWaiters((AbstractQueuedSynchronizer.ConditionObject)condition);
}
public int getWaitQueueLength(Condition condition) {
    if (condition == null)
        throw new NullPointerException();
    if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
        throw new IllegalArgumentException("not owner");
    return sync.getWaitQueueLength((AbstractQueuedSynchronizer.ConditionObject)condition);
}
protected Collection<Thread> getWaitingThreads(Condition condition) {
    if (condition == null)
        throw new NullPointerException();
    if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
        throw new IllegalArgumentException("not owner");
    return sync.getWaitingThreads((AbstractQueuedSynchronizer.ConditionObject)condition);
}
```

关于 ReentrantLock 类，详细的源码解析，可以看看 [《【死磕 Java 并发】—– J.U.C 之重入锁：ReentrantLock》](http://www.iocoder.cn/JUC/sike/ReentrantLock/?vip) 。

> 简单来说，ReenTrantLock 的实现是一种自旋锁，通过循环调用 CAS 操作来实现加锁。它的性能比较好也是因为避免了使线程进入内核态的阻塞状态。想尽办法避免线程进入内核的阻塞状态是我们去分析和理解锁设计的关键钥匙。

## **6.synchronized 和 ReentrantLock 异同？**

- 相同点
  - 都实现了多线程同步和内存可见性语义。
  - 都是可重入锁。
- 不同点
  - 同步实现机制不同
    - `synchronized` 通过 Java 对象头锁标记和 Monitor 对象实现同步。
    - ReentrantLock 通过CAS、AQS（AbstractQueuedSynchronizer）和 LockSupport（用于阻塞和解除阻塞）实现同步。
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
    - ReentrantLock 支持中断处理，且性能较 `synchronized` 会好些。

> 在 `synchronized` 优化以前，它的性能是比 ReenTrantLock 差很多的，但是自从 `synchronized` 引入了偏向锁，轻量级锁（自旋锁）后，**两者的性能就差不多了，在两种方法都可用的情况下，官方甚至建议使用 `synchronized` 。**
>
> 并且，实际代码实战中，可能的优化场景是，通过读写分离，进一步性能的提升，所以使用 ReentrantReadWriteLock 。

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