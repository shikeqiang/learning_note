![Lock](http://static.iocoder.cn/d2cd1e16577dd6482d1e58ace2062408)

`java.util.concurrent.locks.Lock` 接口，比 `synchronized` 提供更具拓展行的锁操作。它允许更灵活的结构，可以具有完全不同的性质，并且可以支持多个相关类的条件对象。它的优势有：

- 可以使锁更公平。
- 可以使线程在等待锁的时候响应中断。
- 可以让线程尝试获取锁，并在无法获取锁的时候立即返回或者等待一段时间。
- 可以在不同的范围，以不同的顺序获取和释放锁。

# 1、什么是可重入锁及ReentrantLock

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

​	==简单来说，ReenTrantLock 的实现是一种自旋锁，通过循环调用 CAS 操作来实现加锁。它的性能比较好也是因为避免了使线程进入内核态的阻塞状态。==想尽办法避免线程进入内核的阻塞状态是我们去分析和理解锁设计的关键钥匙。

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

```java
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

```java
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

```Java
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

```Java
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

```Java
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

```Java
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

```Java
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

```Java
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

```Java
@Override
public void lockInterruptibly() throws InterruptedException {
    sync.acquireInterruptibly(1);	
}
```

#### 5.4 tryLock

```java
//只有当没有其他线程持有锁的时候才可以获得。
@Override
public boolean tryLock() {
    return sync.nonfairTryAcquire(1);
}
```

- `tryLock()` **实现**方法，在实现时，希望能**快速的**获得是否能够获得到锁，因此即使在设置为 `fair = true` ( 使用公平锁 )，依然调用 `Sync.nonfairTryAcquire(int acquires)` 方法。
- 如果**真的**希望 `tryLock()` 还是按照是否公平锁的方式来，可以调用 `tryLock(0, TimeUnit)` 方法来实现。

#### 5.5 tryLock

```java
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

```Java
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

## **6.synchronized 和 ReentrantLock 异同？**

- 相同点
  - 都实现了多线程同步和内存可见性语义。
  - 都是可重入锁。
- 不同点
  - 同步实现机制不同
    - `synchronized` 通过 Java 对象头锁标记和 Monitor 对象实现同步。
    - **ReentrantLock 通过CAS、AQS（AbstractQueuedSynchronizer）和 LockSupport（用于阻塞和解除阻塞）实现同步。**
  - 可见性实现机制不同
    - `synchronized` 依赖 JVM 内存模型保证包含共享变量的多线程内存可见性。
    - ReentrantLock 通过 ASQ 的 `volatile state` 保证包含共享变量的多线程内存可见性。
  - 使用方式不同
    - `synchronized` 可以修饰实例方法（锁住实例对象）、静态方法（锁住类对象）、代码块（显示指定锁对象）。
    - **ReentrantLock 显示调用 tryLock 和 lock 方法，需要在 `finally` 块中释放锁。**
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

# 2、ReentrantReadWriteLock 是什么？

​	**ReentrantReadWriteLock ，读写锁，是用来提升并发程序性能的锁分离技术的 Lock 实现类。==可以用于 “多读少写” 的场景，读写锁支持多个读操作并发执行，写操作只能由一个线程来操作。==**

​	ReadWriteLock 使得你可以同时有多个读取者，只要它们都不试图写入即可。如果写锁已经被其他任务持有，那么任何读取者都不能访问，直至这个写锁被释放为止。

#### ReadWriteLock 对程序性能的提高主要受制于如下几个因素：

1. 数据被读取的频率与被修改的频率相比较的结果。
2. 读取和写入的时间
3. 有多少线程竞争
4. 是否在多处理机器上运行

## 1. 简介

​	重入锁 ReentrantLock 是排他锁，**排他锁在同一时刻仅有一个线程可以进行访问**，但是在大多数场景下，大部分时间都是提供读服务，而写服务占有的时间较少。然而，**读服务不存在数据竞争问题，如果一个线程在读时禁止其他线程读势必会导致性能降低。所以就提供了读写锁。**

​	读写锁维护着一对锁，一个读锁和一个写锁。通过分离读锁和写锁，使得并发性比一般的排他锁有了较大的提升：

- **在同一时间，可以允许多个读线程同时访问。**
- **但是，在写线程访问时，所有读线程和写线程都会被阻塞。**

### 读写锁的**主要特性**：

1. 公平性：支持公平性和非公平性。
2. 重入性：支持重入。读写锁最多支持 65535 个递归写入锁和 65535 个递归读取锁。
3. ==锁降级：遵循获取写锁，再获取读锁，最后释放写锁的次序，如此写锁能够降级成为读锁。==

![image-20190319101414971](/Users/jack/Desktop/md/images/image-20190319101414971.png)

## 2. ReadWriteLock

`java.util.concurrent.locks.ReadWriteLock` ，**读写锁接口**。定义方法如下：

```java
Lock readLock();
Lock writeLock();
```

- 一对方法，**分别获得读锁和写锁 Lock 对象**。

## 3. ReentrantReadWriteLock

`java.util.concurrent.locks.ReentrantReadWriteLock` ，实现 ReadWriteLock 接口，可重入的读写锁实现类。在它内部，维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。**只要没有 Writer 线程，读取锁可以由多个 Reader 线程同时保持。也就说，写锁是独占的，读锁是共享的。**

ReentrantReadWriteLock 类的大体结构如下：

```java
/** 内部类  读锁 */
private final ReentrantReadWriteLock.ReadLock readerLock;
/** 内部类  写锁 */
private final ReentrantReadWriteLock.WriteLock writerLock;

final Sync sync;

/** 使用默认（非公平）的排序属性创建一个新的 ReentrantReadWriteLock */
public ReentrantReadWriteLock() {
    this(false);
}

/** 使用给定的公平策略创建一个新的 ReentrantReadWriteLock */
public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}

/** 返回用于写入操作的锁 */
@Override
public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
/** 返回用于读取操作的锁 */
@Override
public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }

abstract static class Sync extends AbstractQueuedSynchronizer {
    ...
}
public static class WriteLock implements Lock, java.io.Serializable {
    ...
}

public static class ReadLock implements Lock, java.io.Serializable {
   ...
}
```

- ReentrantReadWriteLock 与 ReentrantLock一样，其锁主体也是 Sync，它的读锁、写锁都是通过 Sync 来实现的。所以 ReentrantReadWriteLock 实际上**只有一个锁**，只是在获取读取锁和写入锁的方式上不一样。
- 它的读写锁对应两个类：**ReadLock 和 WriteLock 。这两个类都是 Lock 的子类实现。**

### 读写锁同步状态的设计

​	==在 ReentrantLock 中，使用 Sync ( 实际是 AQS )的 `int` 类型的 `state` 来表示同步状态，表示锁被一个线程重复获取的次数。==使用的是AQS中的一个同步状态state表示 当前共享资源是否被其他线程锁占用。如果为0则表示未被占用，其他值表示该锁被重 入的次数。但是，读写锁 ReentrantReadWriteLock 内部维护着一对读写锁，如果要用一个变量维护多种状态，需要采用“**按位切割使用**”的方式来维护这个变量，**将其切分为两部分：高16为表示读，低16为表示写。**

**分割之后，读写锁通过位运算迅速确定读锁和写锁的状态**。假如当前同步状态为S，那么：

- 写状态，等于 `S & 0x0000FFFF`（将高 16 位全部抹去）
- 读状态，等于 `S >>> 16` (无符号补 0 右移 16 位)。

代码如下：

```java
// Sync.java

static final int SHARED_SHIFT   = 16; // 位数
static final int SHARED_UNIT    = (1 << SHARED_SHIFT);	//左移n位相当于乘以2的n次方
static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1; // 每个锁的最大重入次数，65535
static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;

static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
// &表示两个数都转为二进制，然后从高位开始比较，如果两个数都为1则为1，否则为0
static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
```

- `exclusiveCount(int c)` 静态方法，获得持有写状态的锁的次数。(顾名思义，写锁是独占锁，重入次数)
- `sharedCount(int c)` 静态方法，获得持有读状态的锁的**线程数量**。不同于写锁，读锁可以同时被多个线程持有。而==每个线程持有的读锁支持重入的特性，所以需要对每个线程持有的读锁的数量单独计数，这就需要用到 HoldCounter 计数器。==(HoldCounter是Sync的静态内部类，主要起计算每个线程的数量的作用。)

> FROM 《Java并发编程的艺术》的 [「5.4 读写锁」](http://www.iocoder.cn/JUC/sike/ReentrantReadWriteLock/#) 章节
>
> ![åå](http://static.iocoder.cn/images/JUC/%E8%AF%BB%E5%86%99%E9%94%81-01.png)

​	**如果当前同步状态state不为0，那么先计算低16位写状态，如果低16为为0，也就是写 状态为0则表示高16为不为0，也就是读状态不为0，则读获取到锁;**如果此时低16为 不为0则抹去高16位得出低16位的值，判断是否与state值相同，如果相同则表示写获 取到锁。同样如果state不为0，低16为不为0，且低16位值不等于state，也可以通过 state的值减去低16位的值计算出高16位的值。上述计算过程都是通过位运算计算出来 的。

上图中为什么表示当前状态有一个线程已经获取了写锁，且重入了两次，同时也获取了两次读锁。这是因为:

#### 1、写锁的获取与释放

​	写锁是一个支持重进入的排它锁。如果当前线程已经获取了写锁，则增加写状态。**如果当前线程在获取写锁时，读锁已经被获取(读状态不为0)或者线程不是已经获取写锁的线程，则当前线程进入等待状态。**

#### 2、读锁的获取与释放

​	读锁是一个支持重进入的共享锁。它能够被多个线程同时获取，在没有其他线写线程访问(写状态为0)时，读锁总是会被成功获取，而所作的也只是增加读状态。如果当前线程在获取读锁时，写锁已被其他线程获取，则进入等待状态。

#### 3、锁降级

​	看到上述这两段似乎还是找不到为什么会出现高位和低位都不为0的情况怎样确定当前 线程获取的是写锁的解答，这就需要我们从另一个需要注意的地方说起:锁降级。

​	**何为锁降级，意思主要是为了保证数据的可见性，**假如有一个线程A已经获取了写锁， 并且修改了数据，如果当前线程A不获取读锁而直接释放写锁，此时，另一个线程B获 取到了写锁并修改了数据，那么当前线程A无法感知线程B的数据更新。如果当前线程 A获取读锁，即遵循降级的步骤，则线程B将会被阻塞，直到当前线程A使用数据并释 放读锁之后，线程B才能获取写锁进行数据更新。
另外，**锁降级中读锁的获取是必要的!!!**
​	正是由于锁降级的存在，才会出现上图中高16位和低16为都不为0，但可以确定是写 锁的问题。可以得出结论，如果高16位或者低16位为0，那么我们就可以判断获取到的是写锁或读锁;**如果高16位和低16位都不为0那获取到的应该是写锁。**就是说如果 当前线程已经获取到写锁的话，该线程也是可以通过CAS线程安全的增加读状态的，成功获取读锁。
​	虽然，为了保证数据的可见性引入锁降级可以将写锁降级为读锁，但是却不可以锁升级，将读锁升级为写锁的，也就是不会出现:当前线程已经获取到读锁了，通过某种方式增加写状态获取到写锁的情况。不允许升级的原因也是保证数据的可见性，如果读锁已被多个线程获取，其中任意线程成功获取了写锁并更新了数据，则其更新对其他获取到读锁的线程是不可见的。

### 3.1 构造方法

在上面的构造方法中，我们已经看到基于 `fair` 参数，创建 FairSync 还是 NonfairSync 对象。

### 3.2 getThreadId

`getThreadId(Thread thread)` **静态**方法，获得线程编号。代码如下：

```java
 /**
 * 返回线程对应的线程id，不能直接通过Thread.getId()，因为这个方法不是final修饰，不知道有没有被重写 
 */
static final long getThreadId(Thread thread) {
    return UNSAFE.getLongVolatile(thread, TID_OFFSET);
}

// Unsafe mechanics
private static final sun.misc.Unsafe UNSAFE;
private static final long TID_OFFSET;
static {
    try {
        UNSAFE = sun.misc.Unsafe.getUnsafe();
        Class<?> tk = Thread.class;
        TID_OFFSET = UNSAFE.objectFieldOffset
            (tk.getDeclaredField("tid"));
    } catch (Exception e) {
        throw new Error(e);
    }
}
```

- 按道理说，直接调用线程对应的 `Thread的getId()` 方法即可，代码如下：

  ```java
  private long tid;
  
  public long getId() {
      return tid;
  }
  ```

  - 但是实际上，Thread 的这个方法是**非 final 修饰的**，也就是说，如果我们有实现 Thread 的子类，完全可以重写这个方法，所以可能导致无法获得 `tid` 属性。因此上面的方法，使用 Unsafe*直接获得 `tid` 属性。

## 4. 读锁和写锁

​	上面提到，ReentrantReadWriteLock 的读锁和写锁，基于它内部的 Sync 实现，所以具体的实现方法，就是**对内部的 Sync 的方法的调用**。

### 4.1 ReadLock

​	ReadLock 是 ReentrantReadWriteLock 的内部静态类，实现 `java.util.concurrent.locks.Lock` 接口，读锁实现类。

#### 4.1.1 构造方法

```java
private final Sync sync;

protected ReadLock(ReentrantReadWriteLock lock) {
    sync = lock.sync;
}
```

- `sync` 字段，通过 ReentrantReadWriteLock 的构造方法，传入并使用它的 Sync 对象。

#### 4.1.2 lock

```java
@Override
public void lock() {
    sync.acquireShared(1);
}
```

- 调用 AQS 的 `acquireShared(int arg)` 方法，共享式获得同步状态。所以，**读锁可以同时被多个线程获取。**

#### 4.1.3 lockInterruptibly

```java
@Override
public void lockInterruptibly() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```

#### 4.1.4 tryLock

```java
@Override
public boolean tryLock() {
    return sync.tryReadLock();
}
```

- 和ReentrantLock的tryLock一样。
- `tryLock()` **实现**方法，在实现时，希望能**快速的**获得是否能够获得到锁，因此即使在设置为 `fair = true` ( 使用公平锁 )，依然调用 `Sync#tryReadLock()` 方法。
- 如果**真的**希望 `tryLock()` 还是按照是否公平锁的方式来，可以调用 `#tryLock(0, TimeUnit)` 方法来实现。

#### 4.1.5 tryLock

```java
@Override
public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException {
    return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
}
```

#### 4.1.6 unlock

```java
@Override
public void unlock() {
    sync.releaseShared(1);
}
```

- 调用 AQS 的 `releaseShared(int arg)` 方法，**共享式**释放同步状态。

#### 4.1.7 newCondition

```java
@Override
public Condition newCondition() {
    throw new UnsupportedOperationException();
}
```

- 不支持 Condition 条件。

### 4.2 WriteLock

> WriteLock 的代码，类似 ReadLock 的代码，差别在于**独占式**获取同步状态。

​	WriteLock 是 ReentrantReadWriteLock 的内部静态类，实现 `java.util.concurrent.locks.Lock` 接口，写锁实现类。

#### 4.2.1 构造方法

```java
private final Sync sync;

protected ReadLock(ReentrantReadWriteLock lock) {
    sync = lock.sync;
}
```

- `sync` 字段，通过 ReentrantReadWriteLock 的构造方法，传入并使用它的 Sync 对象。

#### 4.2.2 lock

```java
@Override
public void lock() {
    sync.acquire(1);
}
```

- 调用 AQS 的 `#.acquire(int arg)` 方法，**独占式**获得同步状态。所以，写锁只能**同时**被**一个**线程获取。

#### 4.2.3 lockInterruptibly

```java
@Override
public void lockInterruptibly() throws InterruptedException {
    sync.acquireInterruptibly(1);
}
```

#### 4.2.4 tryLock

```java

@Override
 public boolean tryLock( ) {
    return sync.tryWriteLock();
}
```

- 同上面的

#### 4.2.5 tryLock

```java
@Override 
public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException {
    return sync.tryAcquireNanos(1, unit.toNanos(timeout));
}
```

#### 4.2.6 unlock

```java
@Override
public void unlock() {
    sync.release(1);
}
```

- 调用 AQS 的 `release(int arg)` 方法，**独占式**释放同步状态。

#### 4.2.7 newCondition

```java
@Override
public Condition newCondition() {
    return sync.newCondition();
}
```

- 调用 `Sync的newCondition()` 方法，创建 Condition 对象。

#### 4.2.8 isHeldByCurrentThread

```java
public boolean isHeldByCurrentThread() {
    return sync.isHeldExclusively();
}
```

- 调用 `Sync的isHeldExclusively()` 方法，判断是否被当前线程**独占**锁。

#### 4.2.9 getHoldCount

```java
public int getHoldCount() {
    return sync.getWriteHoldCount();
}
```

- 调用 `Sync的getWriteHoldCount()` 方法，返回当前线程**独占**锁的持有数量。

## 5. Sync 抽象类

​	Sync 是 ReentrantReadWriteLock 的内部静态类，实现 AbstractQueuedSynchronizer 抽象类，同步器抽象类。它使用 AQS 的 `state` 字段，来表示当前锁的持有数量，从而实现**可重入**和**读写锁**的特性。

### 5.1 构造方法

```java
// transient保证不能被序列化
private transient ThreadLocalHoldCounter readHolds; // 当前线程的读锁持有数量
private transient Thread firstReader = null; // 第一个获取读锁的线程
private transient int firstReaderHoldCount; // 第一个获取读锁的重入数

private transient HoldCounter cachedHoldCounter; // 最后一个获得读锁的线程的 HoldCounter 的缓存对象

Sync() {
    readHolds = new ThreadLocalHoldCounter();
    setState(getState()); // ensures visibility of readHolds
}
```

- 见下面的HoldCounter

### 5.2 writerShouldBlock

```java
abstract boolean writerShouldBlock();
```

- 获取写锁时，如果有前序节点也获得锁时，是否阻塞。NonefairSync 和 FairSync 下有不同的实现。详细解析，见 下面的Sync 实现类。

### 5.3 readerShouldBlock

```java
abstract boolean readerShouldBlock();
```

- 获取**读锁**时，如果有前序节点也获得锁时，是否阻塞。NonefairSync 和 FairSync 下有不同的实现。详细解析，见下面的Sync 实现类 。

### 5.4 【写锁】tryAcquire

```java
@Override
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    //当前锁个数
    int c = getState();
    //写锁
    int w = exclusiveCount(c);
    if (c != 0) {
        //c != 0 && w == 0 表示存在读锁
        //当前线程不是已经获取写锁的线程
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        //超出最大范围
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        setState(c + acquires);
        return true;
    }
    // 是否需要阻塞
    if (writerShouldBlock() ||
            !compareAndSetState(c, c + acquires))
        return false;
    //设置获取锁的线程为当前线程
    setExclusiveOwnerThread(current);
    return true;
}
```

- 该方法和 ReentrantLock 的 `tryAcquire(int arg)` 大致一样，差别在判断重入时，增加了一项条件：**读锁是否存在。**==因为要确保写锁的操作对读锁是可见的==。如果在存在读锁的情况下允许获取写锁，那么那些已经获取读锁的其他线程可能就无法感知当前写线程的操作。因此只有等读锁完全释放后，写锁才能够被当前线程所获取，一旦写锁获取了，所有其他读、写线程均会被阻塞。
- ==调用 `writerShouldBlock()` 抽象方法，若返回 true ，则获取写锁失败==。

### 5.5 【读锁】tryAcquireShared

`tryAcqurireShared(int arg)` 方法，尝试获取读同步状态，获取成功返回 `>= 0` 的结果，否则返回 `< 0` 的结果。代码如下：

```java
protected final int tryAcquireShared(int unused) {
    //当前线程
    Thread current = Thread.currentThread();
    int c = getState();
    //exclusiveCount(c)计算写锁
    //如果存在写锁，且锁的持有者不是当前线程，直接返回-1
    //存在锁降级问题，后续阐述
    if (exclusiveCount(c) != 0 &&
            getExclusiveOwnerThread() != current)
        return -1;
    //读锁
    int r = sharedCount(c);

    /*
     * readerShouldBlock():读锁是否需要等待（公平锁原则）
     * r < MAX_COUNT：持有线程小于最大数（65535）
     * compareAndSetState(c, c + SHARED_UNIT)：设置读取锁状态
     */
    if (!readerShouldBlock() &&
            r < MAX_COUNT &&
            compareAndSetState(c, c + SHARED_UNIT)) { //修改高16位的状态，所以要加上2^16
        /*
         * holdCount部分后面讲解
         */
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            firstReaderHoldCount++;
        } else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current);
}
```

- 读锁获取的过程相对于独占锁而言会稍微复杂下，整个过程如下：
  1. 因为存在锁降级情况，如果存在写锁且锁的持有者不是当前线程，则直接返回失败，否则继续。
  2. 依据公平性原则，调用 `readerShouldBlock()` 方法来判断读锁是否不需要阻塞，**读锁持有线程数小于最大值（65535），且 CAS 设置锁状态成功，执行以下代码**，并返回 1 。如果不满足任一条件，则调用 `fullTryAcquireShared(Thread thread)` 方法，详细解析，见下面。

#### 5.5.1 fullTryAcquireShared

```java
final int fullTryAcquireShared(Thread current) {
   HoldCounter rh = null;
   for (;;) {
       int c = getState();
       // 锁降级
       if (exclusiveCount(c) != 0) {
           if (getExclusiveOwnerThread() != current)
               return -1;
       }
       // 读锁需要阻塞，判断是否当前线程已经获取到读锁
       else if (readerShouldBlock()) {
           //列头为当前线程
           if (firstReader == current) {
           }
           //HoldCounter后面讲解
           else {
               if (rh == null) {
                   rh = cachedHoldCounter;
                   if (rh == null || rh.tid != getThreadId(current)) {
                       rh = readHolds.get();
                       if (rh.count == 0) // 计数为 0 ，说明没得到读锁，清空线程变量
                           readHolds.remove();
                   }
               }
               if (rh.count == 0) // 说明没得到读锁
                   return -1;
           }
       }
       //读锁超出最大范围
       if (sharedCount(c) == MAX_COUNT)
           throw new Error("Maximum lock count exceeded");
       //CAS设置读锁成功
       if (compareAndSetState(c, c + SHARED_UNIT)) { //修改高16位的状态，所以要加上2^16  
           //如果是第1次获取“读取锁”，则更新firstReader和firstReaderHoldCount
           if (sharedCount(c) == 0) {
               firstReader = current;
               firstReaderHoldCount = 1;
           }
           //如果想要获取锁的线程(current)是第1个获取锁(firstReader)的线程,则将firstReaderHoldCount+1
           else if (firstReader == current) {
               firstReaderHoldCount++;
           } else {
               if (rh == null)
                   rh = cachedHoldCounter;
               if (rh == null || rh.tid != getThreadId(current))
                   rh = readHolds.get();
               else if (rh.count == 0)
                   readHolds.set(rh);
               //更新线程的获取“读取锁”的共享计数
               rh.count++;
               cachedHoldCounter = rh; // cache for release
           }
           return 1;
       }
   }
}
```

- 该方法会根据“是否需要阻塞等待”，“读取锁的共享计数是否超过限制”等等进行处理。如果不需要阻塞等待，并且锁的共享计数没有超过限制，则通过 **CAS** 尝试获取锁，并返回 1 。所以，`fullTryAcquireShared(Thread)` 方法，是 `tryAcquireShared(int unused)` 方法的**自旋重试的**逻辑。

### 5.6 【写锁】tryRelease

```java
protected final boolean tryRelease(int releases) {
    //释放的线程不为锁的持有者
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    int nextc = getState() - releases;
    //若写锁的新线程数为0，则将锁的持有者设置为null
    boolean free = exclusiveCount(nextc) == 0;
    if (free)
        setExclusiveOwnerThread(null);
    setState(nextc);
    return free;
}
```

- 写锁释放锁的整个过程，和独占锁 ReentrantLock **相似**，每次释放均是减少写状态，**当写状态为 0 时，表示写锁已经完全释放了，从而让等待的其他线程可以继续访问读、写锁，获取同步状态。同时，此次写线程的修改对后续的线程可见。**

### 5.7 【读锁】tryReleaseShared

```java
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    //如果想要释放锁的线程为第一个获取锁的线程
    if (firstReader == current) {
        //仅获取了一次，则需要将firstReader 设置null，否则 firstReaderHoldCount - 1
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    }
    //获取rh对象，并更新“当前线程获取锁的信息”
    else {
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }
    //CAS更新同步状态
    for (;;) {
        int c = getState();
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
```

- `unmatchedUnlockException()` 方法，返回 IllegalMonitorStateException 异常。代码如下：

  ```java
  private IllegalMonitorStateException unmatchedUnlockException() {
      return new IllegalMonitorStateException(
          "attempt to unlock read lock, not locked by current thread");
  }
  ```

  - 出现的情况是，unlock 读锁的线程，非获得读锁的线程。正常使用的情况，不会出现该情况。

### 5.8 tryWriteLock

`tryWriteLock()` 方法，尝试获取写锁。

- 若获取成功，返回 true 。
- 若失败，返回 false 即可，**不进行等待排队**。

代码如下：

```Java
final boolean tryWriteLock(){
    Thread current = Thread.currentThread();
    int c = getState();
    if(c != 0){
        int w = exclusiveCount(c); // 获得现在写锁获取的数量
        if(w == 0 || current != getExclusiveOwnerThread()){  // 判断是否是其他的线程获取了写锁或者当前没有线程占据写锁。若是，返回 false
            return false;
        }
        if(w == MAX_COUNT){ // 超过写锁上限，抛出 Error 错误
            throw new Error("Maximum lock count exceeded");
        }
    }

    if(!compareAndSetState(c, c + 1)){ //  CAS 设置同步状态，尝试获取写锁。若失败，返回 false
        return false;
    }
    setExclusiveOwnerThread(current); // 设置持有写锁为当前线程
    return true;
}
```

### 5.9 tryReadLock

`tryReadLock()` 方法，尝试获取读锁。

- 若获取成功，返回 true 。
- 若失败，返回 false 即可，**不进行等待排队**。

代码如下：

```java
final boolean tryReadLock() {
    Thread current = Thread.currentThread();
    for (;;) {
        int c = getState();
       //exclusiveCount(c)计算写锁
        //如果存在写锁，且锁的持有者不是当前线程，直接返回-1
        //存在锁降级问题，后续阐述
        if (exclusiveCount(c) != 0 &&
            getExclusiveOwnerThread() != current)
            return false;
        // 读锁
        int r = sharedCount(c);
        if (r == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            if (r == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                HoldCounter rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    cachedHoldCounter = rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
            }
            return true;
        }
    }
}
```

### 5.10 isHeldExclusively

```java
protected final boolean isHeldExclusively() {
    // While we must in general read state before owner,
    // we don't need to do so to check if current thread is owner
    return getExclusiveOwnerThread() == Thread.currentThread();
}
```

## 5.11 newCondition

```java
final ConditionObject newCondition() {
    return new ConditionObject();
}
```

## 6. Sync 实现类

### 6.1 NonfairSync

NonfairSync 是 ReentrantReadWriteLock 的内部静态类，实现 Sync 抽象类，非公平锁实现类。代码如下：

```java
static final class NonfairSync extends Sync {

    @Override
    final boolean writerShouldBlock() {
        return false; // writers can always barge
    }
    
    @Override
    final boolean readerShouldBlock() {
        /* As a heuristic to avoid indefinite writer starvation,
         * block if the thread that momentarily appears to be head
         * of queue, if one exists, is a waiting writer.  This is
         * only a probabilistic effect since a new reader will not
         * block if there is a waiting writer behind other enabled
         * readers that have not yet drained from the queue.
         */
        return apparentlyFirstQueuedIsExclusive();
    }
    
}
```

- 因为写锁是**独占排它**锁，所以在非公平锁的情况下，需要调用 AQS 的 `#apparentlyFirstQueuedIsExclusive()` 方法，**判断是否当前写锁已经被获取**。代码如下：

  ```java
  final boolean apparentlyFirstQueuedIsExclusive() {
      Node h, s;
      return (h = head) != null &&
          (s = h.next)  != null &&
          !s.isShared()         && // 非共享，即独占
          s.thread != null;
  }
  ```

### 6.2 FairSync

FairSync 是 ReentrantReadWriteLock 的内部静态类，实现 Sync 抽象类，公平锁实现类。代码如下：

```java
static final class FairSync extends Sync {

    @Override
    final boolean writerShouldBlock() {
        return hasQueuedPredecessors();
    }
    
    @Override
    final boolean readerShouldBlock() {
        return hasQueuedPredecessors();
    }
    
}
```

- 调用 AQS 的 `#hasQueuedPredecessors()` 方法，是否有**前序**节点，即自己不是首个等待获取同步状态的节点。

## 7. HoldCounter

​	在读锁获取锁和释放锁的过程中，我们一直都可以看到一个变量 `rh` （HoldCounter ），该变量在读锁中扮演着非常重要的作用。

​	我们了解**==读锁的内在机制其实就是一个共享锁==**，为了更好理解 HoldCounter ，我们暂且认为它不是一个锁的概率，而相当于一个**计数器**。**一次共享锁的操作就相当于在该计数器的操作。获取共享锁，则该计数器 + 1，释放共享锁，该计数器 - 1。只有当线程获取共享锁后才能对共享锁进行释放、重入操作。**所以==HoldCounter 的作用就是当前线程持有共享锁的数量，这个数量必须要与线程绑定在一起，否则操作其他线程锁就会抛出异常==。

> HoldCounter 是 Sync 的内部静态类。

```java
static final class HoldCounter {
    int count = 0; // 计数器
    final long tid = getThreadId(Thread.currentThread()); // 线程编号
}
```

- > ThreadLocalHoldCounter 是 Sync 的内部静态类。

  ```java
  static final class ThreadLocalHoldCounter extends ThreadLocal<HoldCounter> {
      
      @Override
      public HoldCounter initialValue() {
          return new HoldCounter();
      }
  }
  ```

​	**通过 ThreadLocalHoldCounter 类，HoldCounter 就可以与线程进行绑定了。故而，HoldCounter 应该就是绑定线程上的一个计数器，而 ThreadLocalHoldCounter 则是线程绑定的 ThreadLocal。**从上面我们可以看到 ThreadLocal 将 HoldCounter 绑定到当前线程上，同时 **HoldCounter 也持有线程编号，这样在释放锁的时候才能知道 ReadWriteLock 里面缓存的上一个读取线程（`cachedHoldCounter`）是否是当前线程。**这样做的好处是可以减少`ThreadLocal.get()` 方法的次调用数，因为这也是一个耗时操作。需要说明的是，==HoldCounter 绑定线程编号而不绑定线程对象的原因是，避免 HoldCounter 和 ThreadLocal 互相绑定而导致 GC 难以释放它们==（尽管 GC 能够智能的发现这种引用而回收它们，但是这需要一定的代价），所以其实这样做只是为了帮助 GC 快速回收对象而已。

看到这里我们明白了 HoldCounter 作用了，我们在看一个**获取读锁**的代码段：

```java
//如果获取读锁的线程为第一次获取读锁的线程，则firstReaderHoldCount重入数 + 1
else if (firstReader == current) {
    firstReaderHoldCount++;
} else {
    //非firstReader计数
    if (rh == null)
        rh = cachedHoldCounter;
    //rh == null 或者 rh.tid != current.getId()，需要获取rh
    if (rh == null || rh.tid != getThreadId(current))
        rh = readHolds.get();
        //加入到readHolds中
    else if (rh.count == 0)
        readHolds.set(rh);
    //计数+1
    rh.count++;
    cachedHoldCounter = rh; // cache for release
}
```

- 这里解释下为何要引入 `firstReader`、`firstReaderHoldCount` 变量。这是为了一个效率问题，`firstReader`是不会放入到 `readHolds` 中的，如果读锁仅有一个的情况下，就会避免查找 `readHolds` 。

## 8. 锁降级

​	**锁降级就意味着写锁是可以降级为读锁的，但是需要遵循先获取写锁、获取读锁在释放写锁的次序。**

注意：==如果当前线程先获取写锁，然后释放写锁，再获取读锁这个过程不能称之为锁降级，锁降级一定要遵循那个次序。==

在获取读锁的方法 `tryAcquireShared(int unused)` 中，有一段代码就是来判读锁降级的：

```java
int c = getState();
//exclusiveCount(c)计算写锁
//如果存在写锁，且锁的持有者不是当前线程，直接返回-1
//存在锁降级问题，后续阐述
if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
    return -1;
//读锁
int r = sharedCount(c);
```

​	锁降级中读锁的获取释放为必要？肯定是必要的。试想，假如当前线程 A 不获取读锁而是直接释放了写锁，这个时候另外一个线程 B 获取了写锁，那么这个线程 B 对数据的修改是不会对当前线程 A 可见的。如果获取了读锁，则线程B在获取写锁过程中判断如果有读锁还没有释放则会被阻塞，只有当前线程 A 释放读锁后，线程 B 才会获取写锁成功。

使用实例：

```java
private ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
private Lock readLock = lock.readLock();
private Lock writeLock = lock.writeLock();
private boolean update;
public void processData() {
    readLock.lock(); //读锁获取
    if (!update) {
        readLock.unlock(); //必须先释放读锁
        writeLock.lock(); //锁降级从获取写锁开始
        try {
            if (!update) {
                //准备数据流程（略）
                update = true;
            }
            //获取读锁。在写锁持有期间获取读锁
            //此处获取读锁，是为了防止，当释放写锁后，又有一个线程T获取锁，对数据进行改变，
            //而当前线程下面对改变的数据无法感知。
            //如果获取了读锁，则线程T则被阻塞，直到当前线程释放了读锁，那个T线程才有可能获取写锁。
            readLock.lock();
        } finally {
            writeLock.unlock();//释放写锁
        }
        //锁降级完成
    }

    try {
        //使用数据的流程
    } finally {
        readLock.unlock(); //释放读锁
    }
}
```
## 9.总结

1、读锁的重入是允许多个申请读操作的线程的，而写锁同时只允许单个线程占有，该 线程的写操作可以重入。

2、**如果一个线程占有了写锁，在不释放写锁的情况下，它还能占有读锁，即写锁降级为读锁。**

3、对于同时占有读锁和写锁的线程，如果完全释放了写锁，那么它就完全转换成了读锁，以后的写操作无法重入，在写锁未完全释放时写操作是可以重入的。

4、公平模式下无论读锁还是写锁的申请都必须按照AQS锁等待队列先进先出的顺序。==非公平模式下读操作插队的条件是锁等待队列head节点后的下一个节点是 SHARED型节点，写锁则无条件插队。==

5、读锁不允许newConditon获取Condition接口，而写锁的newCondition接口实现方 法同ReentrantLock。

ReadWriteLock 的源码解析，可以看看 [《【死磕 Java 并发】—– J.U.C 之读写锁：ReentrantReadWriteLock》](http://www.iocoder.cn/JUC/sike/ReentrantReadWriteLock/) 。

参照：<https://xuliugen.blog.csdn.net/article/details/78375986>

# 3、Condition 是什么？

​	在没有 Lock 之前，我们使用 `synchronized` 来控制同步，配合 Object 的 `wait()`、`notify()` 等一系列方法可以实现**等待 / 通知模式**。在 Java SE 5 后，Java 提供了 Lock 接口，相对于 `synchronized` 而言，Lock 提供了条件 Condition ，对线程的等待、唤醒操作更加详细和灵活。下图是 Condition 与 Object 的监视器方法的对比（摘自《Java并发编程的艺术》）：

![Condition 与 Object 的监视器方法的对比](http://static.iocoder.cn/e7e7bb0837bbe68a4364366d4ec9c5db)

## 3.1 Condition接口

​	`java.util.concurrent.locks.Condition` ，条件 Condition 接口，定义了一系列的方法，来对阻塞和唤醒线程：

```java
// ========== 阻塞 ==========

void await() throws InterruptedException; // 造成当前线程在接到信号或被中断之前一直处于等待状态。
void awaitUninterruptibly(); // 造成当前线程在接到信号之前一直处于等待状态。【注意：该方法对中断不敏感】。
long awaitNanos(long nanosTimeout) throws InterruptedException; // 造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。返回值表示剩余时间，如果在`nanosTimeout` 之前唤醒，那么返回值 `= nanosTimeout - 消耗时间` ，如果返回值 `<= 0` ,则可以认定它已经超时了。
boolean await(long time, TimeUnit unit) throws InterruptedException; // 造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。
boolean awaitUntil(Date deadline) throws InterruptedException; // 造成当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态。如果没有到指定时间就被通知，则返回 true ，否则表示到了指定时间，返回返回 false 。

// ========== 唤醒 ==========

void signal(); // 唤醒一个等待线程。该线程从等待方法返回前必须获得与Condition相关的锁。
void signalAll(); // 唤醒所有等待线程。能够从等待方法返回的线程必须获得与Condition相关的锁。
```

​	Condition 是一种广义上的条件队列。他为线程提供了一种更为灵活的**等待 / 通知**模式，线程在调用 await 方法后执行挂起操作，直到线程等待的某个条件为真时才会被唤醒。Condition 必须要配合 Lock 一起使用，因为对共享状态变量的访问发生在多线程环境下。一个 Condition 的实例必须与一个 Lock 绑定，因此 Condition 一般都是作为 Lock 的内部实现。

## 3.2 实例分析

​	在`java.util.concurrent`包中，有两个很特殊的工具类，`Condition`和`ReentrantLock`，使用过的人都知道，`ReentrantLock`（重入锁）是jdk的`concurrent`包提供的一种独占锁的实现。它继承自 `AbstractQueuedSynchronizer`（同步器），确切的说是`ReentrantLock`的一个内部类继承了`AbstractQueuedSynchronizer`，`ReentrantLock`只不过是代理了该类的一些方法，可能有人会问为什么要使用内部类在包装一层？ 我想是安全的关系，因为`AbstractQueuedSynchronizer`中有很多方法，还实现了共享锁，`Condition`(稍候再细说)等功能，如果直接使`ReentrantLock`继承它，则很容易出现`AbstractQueuedSynchronizer`中的API被无用的情况。

```Java
public static void main(String[] args) {
        final ReentrantLock reentrantLock = new ReentrantLock();
        final Condition condition = reentrantLock.newCondition();

        Thread thread = new Thread(new Runnable() {

            public void run(){
                try {
                    reentrantLock.lock();
                    System.out.println("我要等一个新信号"+ this);
// condition.wait();
                    condition.await();

                }
                catch (Exception e) {
                    e.printStackTrace();
                }
                System.out.println("拿到一个信号！！"+ this);
                reentrantLock.unlock();
            }
        }, "waitThread1");

        thread.start();

        Thread thread1 = new Thread(new Runnable() {
            public void run(){
                reentrantLock.lock();
                System.out.println("我拿到锁了");
                try {
                    Thread.sleep(3000);
                }
                catch (Exception e) {
                    e.printStackTrace();
                }
                condition.signalAll();
                System.out.println("我发了一个信号！！");
                reentrantLock.unlock();
            }
        }, "signalThread");

        thread1.start();

    }
```

运行结果如下：

```
我要等一个新信号aqs.ConditionExm$1@5e9d0904
我拿到锁了
我发了一个信号！！
拿到一个信号！！aqs.ConditionExm$1@5e9d0904
```

​	可以看到，`Condition`的执行方式，是当在线程1中调用`await`方法后，线程1将释放锁，并且将自己沉睡，等待唤醒，线程2获取到锁后，开始执行，完毕后，调用`Condition`的`signal`方法，唤醒线程1，线程1恢复执行。

​	以上说明**`Condition`是一个多线程间协调通信的工具类**，使得某个，或者某些线程一起等待某个条件（Condition）,只有当该条件具备( `signal` 或者 `signalAll`方法被带调用)时 ，这些等待线程才会被唤醒，从而重新争夺锁。

## 3.3 实现原理

首先还是要明白，`reentrantLock.newCondition()` 返回的是`Condition`的一个实现，该类在`AbstractQueuedSynchronizer`中被实现，叫做`newCondition()`

```java
public Condition newCondition() {
    return sync.newCondition();
}
final ConditionObject newCondition() {
            return new ConditionObject();
        }
```

​	它可以访问`AbstractQueuedSynchronizer`中的方法和其余内部类。

### 3.3.1 ConditionObject

​	获取一个 Condition 必须要通过 Lock 的 `#newCondition()` 方法。该方法定义在接口 Lock 下面，返回的结果是绑定到此 Lock 实例的**新 Condition 实例**。Condition 为一个接口，其下仅有一个实现类 ConditionObject ，**由于 Condition 的操作需要获取相关的锁，而 AQS则是同步锁的实现基础，所以 ConditionObject 则定义为 AQS 的内部类**。代码如下：

```Java
public class ConditionObject implements Condition, java.io.Serializable {
    /** First node of condition queue. */
    private transient Node firstWaiter; // 头节点
    /** Last node of condition queue. */
    private transient Node lastWaiter; // 尾节点
    
    public ConditionObject() {
    }

    // ... 省略内部代码
}
```

​	从上面代码可以看出，ConditionObject 拥有首节点（`firstWaiter`），尾节点（`lastWaiter`）。当前线程调用 `await()`方法时，将会以当前线程构造成一个节点（Node），并将节点加入到该队列的尾部。结构如下：

![Condition ç­å¾éå](/Users/jack/Desktop/md/images/2018120815002.png)

- Node 里面包含了当前线程的引用。**Node 定义与 AQS 的 CLH 同步队列的节点使用的都是同一个类（AbstractQueuedSynchronized 的 Node 静态内部类）。**
- ConditionObject 的队列结构比 CLH 同步队列的结构简单些，==新增过程较为简单，只需要将原尾节点的 `Node.next`指向新增节点，然后更新 `ConditionObject.lastWaiter` 即可。==

### 3.3.2 大致流程

AQS 等待队列与 Condition 队列是**两个相互独立的队列**

- `await()` 就是在当前线程持有锁的基础上释放锁资源，并新建 Condition 节点加入到 Condition 的队列尾部，阻塞当前线程 。
- `signal()` 就是将 Condition 的头节点移动到 AQS 等待节点尾部，让其等待再次获取锁。

以下是 AQS 队列和 Condition 队列的出入结点的示意图，可以通过这几张图看出线程结点在两个队列中的出入关系和条件。

**I.初始化状态**：AQS等待队列有 3 个Node，Condition 队列有 1 个Node(也有可能 1 个都没有)

![img](/Users/jack/Desktop/md/images/20150423091636088.png)

**II.节点1执行 Condition.await()**

1. 将 head 后移
2. 释放节点 1 的锁并从 AQS 等待队列中移除
3. 将节点 1 加入到 Condition 的等待队列中
4. 更新 lastWaiter 为节点 1

![img](/Users/jack/Desktop/md/images/20150423091555989.png)

**III.节点 2 执行 Condition.signal() 操作**

1. 将 firstWaiter后移
2. 将节点 4 移出 Condition 队列
3. 将节点 4 加入到 AQS 的等待队列中去
4. 更新 AQS 的等待队列的 tail

![img](/Users/jack/Desktop/md/images/20150423091621011.png)

## 3.4 等待

### 3.4.1 await

​	调用 Condition 的 `#await()` 方法，会使当前线程进入**等待**状态，同时会加入到 Condition 等待队列，并且同时释放锁。当从 `#await()` 方法结束时，当前线程一定是获取了Condition 相关联的锁。

```java
public final void await() throws InterruptedException {
     // 当前线程中断
    if (Thread.interrupted())
        throw new InterruptedException();
    // 将当前线程包装下后，添加到Condition自己维护的一个链表中。当前线程加入等待队列
    Node node = addConditionWaiter(); 
    // 释放当前线程占有的锁，从demo中看到，调用await前，当前线程是占有锁的
    int savedState = fullyRelease(node);
 
    int interruptMode = 0;
    /**
     * 检测此节点的线程是否在同步队上，如果不在，则说明该线程还不具备竞争锁的资格，则继续等待
     * 直到检测到此节点在同步队列上
     */
    while (!isOnSyncQueue(node)) {// 释放完毕后，遍历AQS的队列，看当前节点是否在队列中，
        // 不在 说明它还没有竞争锁的资格，所以继续将自己沉睡。
        // 直到它被加入到队列中，
        LockSupport.park(this);		// 线程挂起
        //如果已经中断了，则退出
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
  // 被唤醒后，重新开始正式竞争锁，同样，如果竞争不到还是会将自己沉睡，等待唤醒重新开始竞争。竞争同步状态
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    // 清理下条件队列中的不是在等待条件的节点
    if (node.nextWaiter != null)
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
```

- 首先，将当前线程新建一个节点同时加入到条件队列中。
- 然后，释放当前线程持有的同步状态。
- 之后，则是不断检测该节点代表的线程，出现在 CLH 同步队列中（收到 signal 信号之后，就会在 AQS 队列中检测到），如果不存在则一直挂起。
- 最后，重新参与竞争，获取到同步状态。

### 3.4.2 addConditionWaiter

 	**该方法主要是将当前线程加入到 Condition 条件队列中。**当然，在加入到尾节点之前，会调用 unlinkCancelledWaiters() 方法，清除所有状态不为 Condition 的节点。

```Java
private Node addConditionWaiter() {
    Node t = lastWaiter;    //尾节点
    //Node的节点状态如果不为CONDITION，则表示该节点不处于等待状态，需要清除节点
    if (t != null && t.waitStatus != Node.CONDITION) {
        //清除条件队列中所有状态不为Condition的节点
        unlinkCancelledWaiters();
        t = lastWaiter;
    }
    //当前线程新建节点，状态 CONDITION
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
    /**
     * 将该节点加入到条件队列中最后一个位置
     */
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    lastWaiter = node;
    return node;
}
```

### 3.4.3 fullyRelease	

​	**`fullyRelease(Node node)` 方法，负责完全释放该线程持有的锁，因为例如 ReentrantLock 是可以重入的。**代码如下：

```Java
final long fullyRelease(Node node) {
    boolean failed = true;
    try {
        // 节点状态--其实就是持有锁的数量
        long savedState = getState();
        // 释放锁
        if (release(savedState)) {
            failed = false;
            return savedState;
        } else {
            throw new IllegalMonitorStateException();
        }
    } finally {
        if (failed)
            node.waitStatus = Node.CANCELLED;
    }
}
```

- 正常情况下，释放锁都能成功，因为是先调用 `Lock的lock()` 方法，再调用 `Condition的await()` 方法。
- 那么什么情况下会失败，抛出 IllegalMonitorStateException 异常呢？例如，当前线程未持有锁，未调用 `Lock的lock()` 方法，而直接调用 `Condition的await()` 方法，此时就会抛出该异常。
- **另外，释放失败的情况下，会设置 Node 的等待状态为 `Node.CANCELED` 。**

### 3.4.4 isOnSyncQueue

​	`isOnSyncQueue(Node node)` 方法，如果一个节点刚开始在条件队列上，现在在同步队列上获取锁则返回 true 。代码如下：

```Java
final boolean isOnSyncQueue(Node node) {
    // 状态为 Condition，获取前驱节点为 null ，返回 false
    if (node.waitStatus == Node.CONDITION || node.prev == null)
        return false;
    // 后继节点不为 null，肯定在 CLH 同步队列中
    if (node.next != null)
        return true;

    return findNodeFromTail(node);
}
```

### 3.4.5 unlinkCancelledWaiters

​	`unlinkCancelledWaiters()` 方法，负责将条件队列中状态不为 Condition 的节点删除。代码如下：

```Java
// 等待队列是一个单向链表，遍历链表将已经取消等待的节点清除出去
// 纯属链表操作，很好理解，看不懂多看几遍就可以了
private void unlinkCancelledWaiters() {
    Node t = firstWaiter;
    Node trail = null; // 用于中间不需要跳过时，记录上一个 Node 节点
    while (t != null) {
        Node next = t.nextWaiter;
        // 如果节点的状态不是 Node.CONDITION 的话，这个节点就是被取消的
        if (t.waitStatus != Node.CONDITION) {
            t.nextWaiter = null;
            if (trail == null)
                firstWaiter = next;
            else
                trail.nextWaiter = next;
            if (next == null)
                lastWaiter = trail;
        }
        else
            trail = t;
        t = next;
    }
}
```

## 3.5 通知

### 3.5.1 signal

​	回到上面的demo，锁被释放后，线程1开始沉睡，这个时候线程因为线程1沉睡时，会唤醒AQS队列中的头结点，所所以线程2会开始竞争锁，并获取到，等待3秒后，线程2会调用signal方法，“发出”signal信号。

​	调用 ConditionObject的 `#signal()` 方法，将会唤醒在等待队列中等待最长时间的节点（条件队列里的首节点），在唤醒节点前，会将节点移到CLH同步队列中。

```java
public final void signal() {
    //检测当前线程是否为拥有锁的独
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    //头节点，唤醒条件队列中的第一个节点
    Node first = firstWaiter; // firstWaiter为condition自己维护的一个链表的头结点，
                              // 取出第一个节点后开始唤醒操作
    if (first != null)
        doSignal(first);
}
```

​	该方法首先会判断当前线程是否已经获得了锁，这是前置条件。然后调用 `#doSignal(Node first)` 方法，唤醒条件队列中的头节点。代码如下：

```java
private void doSignal(Node first) {
    do {
        //修改头结点，完成旧头结点的移出工作
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
        first.nextWaiter = null;
    } while (!transferForSignal(first) &&		// 将老的头结点，加入到AQS的等待队列中
            (first = firstWaiter) != null);
}
```

​	主要是做两件事：1）修改头节点；2）调用 `transferForSignal(Node first)` 方法将节点移动到 CLH 同步队列中。代码如下：

```java
final boolean transferForSignal(Node node) {
    //将该节点从状态CONDITION改变为初始状态0,
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;

    //将节点加入到syn队列中去，返回的是syn队列中node节点前面的一个节点
    Node p = enq(node);
    int ws = p.waitStatus;
    //如果结点p的状态为cancel 或者修改waitStatus失败，则直接唤醒
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);
    return true;
}
```

#### 整个通知的流程如下：

1. 判断当前线程是否已经获取了锁，如果没有获取则直接抛出异常，因为获取锁为通知的前置条件。
2. 如果线程已经获取了锁，则将唤醒条件队列的首节点
3. 唤醒首节点是先将条件队列中的头节点移出，然后调用 AQS 的 `#enq(Node node)` 方法将其安全地移到 CLH 同步队列中
4. 最后判断如果该节点的同步状态是否为 `Node.CANCEL` ，或者修改状态为 `Node.SIGNAL` 失败时，则直接调用 LockSupport 唤醒该节点的线程。

​	关键的就在于此，**我们知道AQS自己维护的队列是当前等待资源的队列，AQS会在资源被释放后，依次唤醒队列中从前到后的所有节点，使他们对应的线程恢复执行。直到队列为空。**

​	而==Condition自己也维护了一个队列，该队列的作用是维护一个等待signal信号的队列，两个队列的作用是不同，事实上，每个线程也仅仅会同时存在以上两个队列中的一个==，流程是这样的：

1. 线程1调用`reentrantLock.lock`时，线程被加入到AQS的等待队列中。
2. 线程1调用`await`方法被调用时，该线程从AQS中移除，对应操作是锁的释放。
3. 接着马上被加入到`Condition`的等待队列中，该线程需要`signal`信号。
4. 线程2，因为线程1释放锁的关系，被唤醒，并判断可以获取锁，于是线程2获取锁，并被加入到AQS的等待队列中。
5. 线程2调用`signal`方法，这个时候`Condition`的等待队列中只有线程1一个节点，于是它被取出来，并被加入到AQS的等待队列中。 注意，这个时候，线程1 并没有被唤醒。
6. `signal`方法执行完毕，线程2调用`reentrantLock.unLock()`方法，释放锁。这个时候因为AQS中只有线程1，于是，AQS释放锁后按从头到尾的顺序唤醒线程时，线程1被唤醒，于是线程1回复执行。
7. 直到释放所整个过程执行完毕。

​	可以看到，整个协作过程是靠结点在AQS的等待队列和`Condition`的等待队列中来回移动实现的，`Condition`作为一个条件类，很好的自己维护了一个等待信号的队列，并在适时的时候将结点加入到AQS的等待队列中来实现的唤醒操作。

- Condition 的使用，可以看看 [《怎么理解 Condition》](http://www.importnew.com/9281.html)
- Condition 的源码，可以看看 [《【死磕 Java 并发】—– J.U.C 之 Condition》](http://www.iocoder.cn/JUC/sike/Condition/) 。
- 详细阅读可以看[《一行一行源码分析清楚 AbstractQueuedSynchronizer (二)》](https://javadoop.com/post/AbstractQueuedSynchronizer-2#toc5) 。

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