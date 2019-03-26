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

# 4、LockSupport 是什么？

**LockSupport 是 JDK 中比较底层的类，用来创建锁和其他同步工具类的基本线程阻塞。**

- Java 锁和同步器框架的核心 AQS(AbstractQueuedSynchronizer)，就是通过调用 `LockSupport.park()`和 `LockSupport.unpark()`方法，来实现线程的阻塞和唤醒的。

- LockSupport 很类似于二元信号量(只有 1 个许可证可供使用)，**如果这个许可还没有被占用，当前线程获取许可并继续执行；如果许可已经被占用，当前线程阻塞，等待获取许可。**

- **许可默认是被占用的** ，调用park()时获取不到许可，进入阻塞状态。如下面程序会一直阻塞。

  ```java
  public static void main(String[] args)
  {
       LockSupport.park();
       System.out.println("block.");
  }
  ```

  如下代码：**先释放许可，再获取许可，主线程能够正常终止**。LockSupport许可的获取和释放，一般来说是对应的，如果多次unpark，只有一次park也不会出现什么问题，结果是许可处于可用状态。

  ```Java
  public static void main(String[] args)
  {
       Thread thread = Thread.currentThread();
       LockSupport.unpark(thread);//释放许可
       LockSupport.park();// 获取许可
       System.out.println("b");
  }
  ```

- LockSupport是**不可重入** 的，如果一个线程连续2次调用 LockSupport .park()，那么该线程一定会一直阻塞下去。

  ```Java
  public static void main(String[] args) throws Exception
  {
    Thread thread = Thread.currentThread();
    
    LockSupport.unpark(thread);
    
    System.out.println("a");
    LockSupport.park();
    System.out.println("b");
    LockSupport.park();
    System.out.println("c");
  }
  ```

  这段代码打印出a和b，不会打印c，因为第二次调用park的时候，线程无法获取许可出现死锁。

对于 LockSupport 了解即可，面试一般问的不多。可以看看如下文章：

- [《多线程同步工具 —— LockSupport》](https://www.cnblogs.com/hvicen/p/6217303.html)
- [《Java 并发编程 —— LockSupport》](http://www.tianshouzhi.com/api/tutorials/mutithread/303) 带部分源码解析。
