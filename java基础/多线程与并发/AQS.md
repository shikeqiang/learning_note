# AQS及其组件

底层通过CAS和volatile实现

![1544061621591.png](/Users/jack/Desktop/md/images/1544061621591.png?raw=true)

### 1 AQS 简单介绍

AQS的全称为（AbstractQueuedSynchronizer），这个类在java.util.concurrent.locks包下面。

![1544061743209.png](/Users/jack/Desktop/md/images/1544061743209.png?raw=true)

​	AQS是一个用来构建锁和同步器的框架，使用AQS能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的ReentrantLock，Semaphore，其他的诸如ReentrantReadWriteLock，SynchronousQueue，FutureTask等等皆是基于AQS的。当然，我们自己也能利用AQS非常轻松容易地构造出符合我们自己需求的同步器。

### 2 AQS 原理

#### 2.1 AQS 原理概览

​	**<u>AQS核心思想是，==如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态==</u>。==如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。==**

> CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（**虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系**）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配。

看个AQS(AbstractQueuedSynchronizer)原理图：

![1544061850720.png](/Users/jack/Desktop/md/images/1544061850720.png?raw=true)

**AQS使用一个int成员变量来表示同步状态**，通过内置的FIFO队列来完成获取资源线程的排队工作。AQS使用CAS对该同步状态进行原子操作实现对其值的修改。

```Java
private volatile int state;		//共享变量，使用volatile修饰保证线程可见性
```

状态信息通过procted类型的getState，setState，compareAndSetState进行操作

```Java
//返回同步状态的当前值
protected final int getState() {  
        return state;
}
 // 设置同步状态的值
protected final void setState(int newState) { 
        state = newState;
}
//原子地（CAS操作）将同步状态值设置为给定值update如果当前同步状态的值等于expect（期望值）
protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

#### 2.2 AQS 对资源的共享方式

**AQS定义两种资源共享方式**

- Exclusive

  （独占）：只有一个线程能执行，如ReentrantLock。又可分为公平锁和非公平锁：

  - 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
  - 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的

- **Share**（共享）：多个线程可同时执行，如Semaphore/CountDownLatch。Semaphore、CountDownLatCh、 CyclicBarrier、ReadWriteLock 我们都会在后面讲到。

ReentrantReadWriteLock 可以看成是组合式，因为ReentrantReadWriteLock也就是读写锁允许多个线程同时对某一资源进行读。

不同的自定义同步器争用共享资源的方式也不同。**自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在上层已经帮我们实现好了。**

#### 2.3 AQS底层使用了模板方法模式

**同步器的设计是基于模板方法模式的**，如果需要自定义同步器一般的方式是这样（模板方法模式很经典的一个应用）：

1. 使用者继承AbstractQueuedSynchronizer并重写指定的方法。（这些重写方法很简单，无非是对于共享资源state的获取和释放）
2. 将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。

这和我们以往通过实现接口的方式有很大区别，这是模板方法模式很经典的一个运用。

> ​	模板方法模式是基于”继承“的，主要是为了在不改变模板结构的前提下在子类中重新定义模板中的内容以实现复用代码。举个很简单的例子假如我们要去一个地方的步骤是：购票`buyTicket()`->安检`securityCheck()`->乘坐某某工具回家`ride()`->到达目的地`arrive()`。我们可能乘坐不同的交通工具回家比如飞机或者火车，所以除了`ride()`方法，其他方法的实现几乎相同。我们可以定义一个包含了这些方法的抽象类，然后用户根据自己的需要继承该抽象类然后修改 `ride()`方法。
>
> ​	==模板方法一般是有一个顶级的抽象类，定义了一些抽象公共方法，并且一般会有一个执行所有公共方法的具体方法(this.method)，然后子类继承抽象类，实现具体的方法==。

**AQS使用了模板方法模式，自定义同步器时需要重写下面几个AQS提供的模板方法：**

```Java
isHeldExclusively()	//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)		//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)		//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

​	默认情况下，每个方法都抛出 `UnsupportedOperationException`。 这些方法的实现必须是内部线程安全的，并且通常应该简短而不是阻塞。**AQS类中的其他方法都是final** ，所以无法被其他类使用，只有这几个方法可以被其他类使用。

​	以==ReentrantLock==为例，state初始化为0，表示未锁定状态。**A线程lock()时，会调用tryAcquire()独占该锁并将state+1。**此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），因为它是**可重入锁**。但要注意，**获取多少次就要释放多么次，这样才能保证state是能回到零态的**。

​	再以==CountDownLatch==以例，**任务分为N个子线程去执行，state也初始化为N（注意N要与线程个数一致）**。这N个子线程是并行执行的，每个子线程执行完后countDown()一次，state会CAS(Compare and Swap)减1。等到所有子线程都执行完后(即state=0)，会unpark()主调用线程，然后主调用线程就会从await()函数返回，继续后余动作。

​	一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现`tryAcquire-tryRelease`、`tryAcquireShared-tryReleaseShared`中的一种即可。**但AQS也支持自定义同步器同时实现独占和共享两种方式，如`ReentrantReadWriteLock`。**

推荐两篇 AQS 原理和相关源码分析的文章：

- <http://www.cnblogs.com/waterystone/p/4920797.html>
- <https://www.cnblogs.com/chengxiao/archive/2017/07/24/7141160.html>

### 3 Semaphore(信号量)-允许多个线程同时访问

​	==Semaphore 是一种基于计数的信号量。它可以设定一个阈值，基于此，多个线程竞争获取许可信号，做完自己的申请后归还，超过阈值后，线程申请许可信号将会被阻塞。== 

​	**synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，Semaphore(信号量)可以指定多个线程同时访问某个资源。**

​	==用于保证同一时间并发访问线程的数目==。**信号量在操作系统中是很重要的概念，Java并发库里的Semaphore就可以很轻松的完成类似操作系统信号量的控制。**Semaphore可以很容易控制系统中某个资源被同时访问的线程个数。在前面的链表中，链表正常是可以保存无限个节点的，而**Semaphore可以实现有限大小的列表。**

示例代码如下：

```java
public class SemaphoreExample1 {
	// 请求的数量
	private static final int threadCount = 550;
	public static void main(String[] args) throws InterruptedException {
		// 创建一个具有固定线程数量的线程池对象（如果这里线程池的线程数量给太少的话你会发现执行的很慢）
		ExecutorService threadPool = Executors.newFixedThreadPool(300);
		// 一次只能允许执行的线程数量。
		final Semaphore semaphore = new Semaphore(20);
		for (int i = 0; i < threadCount; i++) {
			final int threadnum = i;
			threadPool.execute(() -> {// Lambda 表达式的运用
				try {
					semaphore.acquire();// 获取一个许可，所以可运行线程数量为20/1=20
					test(threadnum);
					semaphore.release();// 释放一个许可
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}

			});
		}
		threadPool.shutdown();
		System.out.println("finish");
	}
	public static void test(int threadnum) throws InterruptedException {
		Thread.sleep(1000);// 模拟请求的耗时操作
		System.out.println("threadnum:" + threadnum);
		Thread.sleep(1000);// 模拟请求的耗时操作
	}
}
```

**执行 `acquire` 方法阻塞，直到有一个许可证可以获得然后拿走一个许可证；每个 `release` 方法增加一个许可证，这可能会释放一个阻塞的acquire方法。**然而，其实并没有实际的许可证这个对象，Semaphore只是维持了一个可获得许可证的数量。 **Semaphore经常用于限制获取某种资源的线程数量。**

当然一次也可以一次拿取和释放多个许可，不过一般没有必要这样做：

```
					semaphore.acquire(5);// 获取5个许可，所以可运行线程数量为20/5=4
					test(threadnum);
					semaphore.release(5);// 获取5个许可，所以可运行线程数量为20/5=4
```

除了 `acquire`方法之外，另一个比较常用的与之对应的方法是`tryAcquire`方法，该方法如果获取不到许可就立即返回false。

Semaphore 有两种模式，公平模式和非公平模式。

- **公平模式：** 调用acquire的顺序就是获取许可证的顺序，遵循FIFO；
- **非公平模式：** 抢占式的。

**Semaphore 对应的两个构造方法如下：**

```Java
   public Semaphore(int permits) {
        sync = new NonfairSync(permits);
    }

    public Semaphore(int permits, boolean fair) {
        sync = fair ? new FairSync(permits) : new NonfairSync(permits);
    }
```

**这两个构造方法，都必须提供许可的数量，第二个构造方法可以指定是公平模式还是非公平模式，默认非公平模式。**

具体可看：<https://blog.csdn.net/qq_19431333/article/details/70212663>

### 4 CountDownLatch （倒计时器）

​	==CountDownLatch是一个同步工具类，用来协调多个线程之间的同步。这个工具通常用来控制线程等待，它可以让某一个线程等待直到倒计时结束，再开始执行。==

#### 4.1 CountDownLatch 的两种典型用法

①某一线程在开始运行前等待n个线程执行完毕。将 CountDownLatch 的计数器初始化为n ：`new CountDownLatch(n) `，**每当一个任务线程执行完毕，就将计数器减1 `countdownlatch.countDown()`，**当计数器的值变为0时，在`CountDownLatch上 await()`的线程就会被唤醒。一个典型应用场景就是启动一个服务时，主线程需要等待多个组件加载完毕，之后再继续执行。**即可以通过初始化，定义线程个数。**

②实现多个线程开始执行任务的最大并行性。注意是并行性，不是并发，强调的是多个线程在某一时刻同时开始执行。类似于赛跑，将多个线程放到起点，等待发令枪响，然后同时开跑。做法是初始化一个共享的 `CountDownLatch` 对象，将其计数器初始化为 1 ：`new CountDownLatch(1) `，多个线程在开始执行任务前首先 `coundownlatch.await()`，当主线程调用 countDown() 时，计数器变为0，多个线程同时被唤醒。

#### 4.2 源码分析：

```Java
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```

​	**构造器中的计数值(count)实际上就是闭锁需要等待的线程数量。这个值只能被设置一次，而且CountDownLatch没有提供任何机制去重新设置这个计数值。** 

​	==与CountDownLatch的第一次交互是主线程等待其他线程。主线程必须在启动其他线 后立即调用CountDownLatch.await()方法。这样主线程的操作就会在这个方法上阻 塞，直到其他线程完成各自的任务。==

​	其他N 个线程必须引用闭锁对象，因为他们需要通知CountDownLatch对象，他们已经完成了各自的任务。这种通知机制是通过 CountDownLatch.countDown()方法来完成的;每调用一次这个方法，在构造函数中初始化的count值就减1。所以当N个线程都调用了这个方法，count的值等于0，然后主线程就能通过await()方法，恢复执行自己的任务。

注意：

> (1)CountDownLatch的构造函数
>
>  CountDownLatch countDownLatch = new CountDownLatch(7); 	//7表示需要等待执行完毕的线程数量。
>
> (2)在每一个线程执行完毕之后，都需要执行 countDownLatch.countDown() 方法，不然计数器就不会准确;
>
> **(3)只有所有的线程执行完毕之后，才会执行 countDownLatch.await() 之后的 代码;** 
>
> ==(4)CountDownLatch 阻塞的是主线程;== 

​	CountDownLatch实现了AQS接口，以他为例，任务分为 N 个子线程去执行，state 也初始化为 N(注意 N 要与线程个数一致)。==这 N 个子线程是并行执行的，每个子线程执行完后 countDown()一次，state会 CAS 减 1。==等到所有子线程都执行完后(即 state=0)，会 unpark()主调用线程，然后主调用线程就会从 await()函数返回，继续后余动作。 

#### 4.3 CountDownLatch 的使用示例

```Java
public class CountDownLatchExample1 {
	// 请求的数量
	private static final int threadCount = 550;
	public static void main(String[] args) throws InterruptedException {
		// 创建一个具有固定线程数量的线程池对象（如果这里线程池的线程数量给太少的话你会发现执行的很慢）
		ExecutorService threadPool = Executors.newFixedThreadPool(300);
		final CountDownLatch countDownLatch = new CountDownLatch(threadCount);
		for (int i = 0; i < threadCount; i++) {
			final int threadnum = i;
			threadPool.execute(() -> {// Lambda 表达式的运用
				try {
					test(threadnum);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} finally {
					countDownLatch.countDown();// 表示一个请求已经被完成
				}

			});
		}
		countDownLatch.await();
		threadPool.shutdown();
		System.out.println("finish");  //当这550个请求被处理完成之后，才会执行
	}
	public static void test(int threadnum) throws InterruptedException {
		Thread.sleep(1000);// 模拟请求的耗时操作
		System.out.println("threadnum:" + threadnum);
		Thread.sleep(1000);// 模拟请求的耗时操作
	}
}
```

上面的代码中，我们定义了请求的数量为550，当这550个请求被处理完成之后，才会执行`System.out.println("finish");`。

#### 4.4 CountDownLatch 的不足

​	CountDownLatch是一次性的，**计数器的值只能在构造方法中初始化一次**，之后没有任何机制再次对其设置值，当CountDownLatch使用完毕后，它不能再次被使用。

### 5 CyclicBarrier(循环栅栏)

​	==CyclicBarrier是另一种多线程并发控制使用工具，和CountDownLatch非常类似，他也 可以实现线程间的计数等待，但他的功能要比CountDownLatch更加强大一些。==

​	CyclicBarrier 的字面意思是可循环使用(Cyclic)的屏障(Barrier)。它要做的事情是，**让一组线程到达一个屏障(也可以叫同步点)时被阻塞，直到最后一个线程到达 屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。** 

​	**CyclicBarrier默认的构造方法是CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。** 

​	CyclicBarrier 和 CountDownLatch 非常类似，它也可以实现线程间的计数等待，但是它的功能比 CountDownLatch 更加复杂和强大。主要应用场景和 CountDownLatch 类似。

==CyclicBarrier强调的是n个线程，大家相互等待，只要有一个没完成，所有人都得等着。==

#### 5.1 CyclicBarrier 的应用场景

​	**CyclicBarrier 可以用于多线程计算数据，最后合并计算结果的应用场景。**比如我们用一个Excel保存了用户所有银行流水，每个Sheet保存一个帐户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个sheet里的银行流水，都执行完之后，得到每个sheet的日均银行流水，最后，再用barrierAction用这些线程的计算结果，计算出整个Excel的日均银行流水。

#### 5.2 CyclicBarrier 的使用示例

示例1：

```Java
public class CyclicBarrierExample2 {
	// 请求的数量
	private static final int threadCount = 550;
	// 5是需要同步的线程数量，即屏障拦截的线程数量
	private static final CyclicBarrier cyclicBarrier = new CyclicBarrier(5);
	public static void main(String[] args) throws InterruptedException {
		// 创建线程池
		ExecutorService threadPool = Executors.newFixedThreadPool(10);
		for (int i = 0; i < threadCount; i++) {
			final int threadNum = i;
			Thread.sleep(1000);
			threadPool.execute(() -> {
				try {
					test(threadNum);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} catch (BrokenBarrierException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			});
		}
		threadPool.shutdown();
	}
	public static void test(int threadnum) throws InterruptedException, BrokenBarrierException {
		System.out.println("threadnum:" + threadnum + "is ready");
		try {
			cyclicBarrier.await(2000, TimeUnit.MILLISECONDS);
		} catch (Exception e) {
			System.out.println("-----CyclicBarrierException------");
		}
		System.out.println("threadnum:" + threadnum + "is finish");
	}

}
```

运行结果，如下：

```Java
threadnum:0is ready
threadnum:1is ready
threadnum:2is ready
threadnum:3is ready
threadnum:4is ready
threadnum:4is finish
threadnum:0is finish
threadnum:1is finish
threadnum:2is finish
threadnum:3is finish
threadnum:5is ready
threadnum:6is ready
threadnum:7is ready
threadnum:8is ready
threadnum:9is ready
threadnum:9is finish
threadnum:5is finish
threadnum:8is finish
threadnum:7is finish
threadnum:6is finish
......
```

可以看到当线程数量也就是请求数量达到我们定义的 5 个的时候， `await`方法之后的方法才被执行。即在最开始定义了屏障拦截的线程数。

另外，CyclicBarrier还提供一个更高级的构造函数`CyclicBarrier(int parties, Runnable barrierAction)`，用于在线程到达屏障时，优先执行`barrierAction`线程，方便处理更复杂的业务场景。示例代码如下：

```java
public class CyclicBarrierExample3 {
	// 请求的数量
	private static final int threadCount = 550;
	// 需要同步的线程数量，这里的() -> {
		System.out.println("------当线程数达到之后，优先执行------");
	}相当于Runnable barrierAction
	
	private static final CyclicBarrier cyclicBarrier = new CyclicBarrier(5, () -> {
		System.out.println("------当线程数达到之后，优先执行------");
	});

	public static void main(String[] args) throws InterruptedException {
		// 创建线程池
		ExecutorService threadPool = Executors.newFixedThreadPool(10);

		for (int i = 0; i < threadCount; i++) {
			final int threadNum = i;
			Thread.sleep(1000);
			threadPool.execute(() -> {
				try {
					test(threadNum);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} catch (BrokenBarrierException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			});
		}
		threadPool.shutdown();
	}

	public static void test(int threadnum) throws InterruptedException, BrokenBarrierException {
		System.out.println("threadnum:" + threadnum + "is ready");
		cyclicBarrier.await();
		System.out.println("threadnum:" + threadnum + "is finish");
	}

}
```

运行结果，如下：

```
threadnum:0is ready
threadnum:1is ready
threadnum:2is ready
threadnum:3is ready
threadnum:4is ready
------当线程数达到之后，优先执行------
threadnum:4is finish
threadnum:0is finish
threadnum:2is finish
threadnum:1is finish
threadnum:3is finish
threadnum:5is ready
threadnum:6is ready
threadnum:7is ready
threadnum:8is ready
threadnum:9is ready
------当线程数达到之后，优先执行------
threadnum:9is finish
threadnum:5is finish
threadnum:6is finish
threadnum:8is finish
threadnum:7is finish
......
```

#### 5.3 CyclicBarrier和CountDownLatch的区别

​	==CountDownLatch是计数器，只能使用一次，而CyclicBarrier的计数器提供reset()方法，可以多次使用。==但是此外，javadoc是这么描述它们的：

> CountDownLatch: A synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes.(CountDownLatch: 一个或者多个线程，等待其他多个线程完成某件事情之后才能执行；) 
>
> CyclicBarrier : A synchronization aid that allows a set of threads to all wait for each other to reach a common barrier point.(CyclicBarrier : 多个线程互相等待，直到到达同一个同步点，再继续一起执行。)

对于CountDownLatch来说，重点是“一个线程（多个线程）等待”，而其他的N个线程在完成“某件事情”之后，可以终止，也可以等待。而**对于CyclicBarrier，重点是多个线程，在任意一个线程没有完成，所有的线程都必须等待。**

CountDownLatch是计数器，线程完成一个记录一个，只不过计数不是递增而是递减，而CyclicBarrier更像是一个阀门，需要所有线程都到达，阀门才能打开，然后继续执行。

![image-20190411102812189](/Users/jack/Desktop/md/images/image-20190411102812189.png)

CyclicBarrier和CountDownLatch的区别这部分内容参考了如下两篇文章：

- <https://blog.csdn.net/u010185262/article/details/54692886>
- <https://blog.csdn.net/tolcf/article/details/50925145?utm_source=blogxgwz0>

### 6 ReentrantLock 和 ReentrantReadWriteLock

​	ReentrantLock 和 synchronized 的区别在上面已经讲过了这里就不多做讲解。另外，需要注意的是：读写锁 **ReentrantReadWriteLock 可以保证多个线程可以同时读，所以在读操作远大于写操作的时候，读写锁就非常有用了。**











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

AQS 通过内置的**双向 FIFO 同步队列**来完成资源获取线程的**排队工作**：

- 如果当前线程获取同步状态失败（锁）时，AQS 则会将当前线程以及等待状态等信息构造成一个节点（Node）并将其加入同步队列，同时会阻塞当前线程
- 当同步状态**释放**时，则会把节点中的线程唤醒，使其再次尝试获取同步状态。

### 主要方法：

AQS 主要提供了如下**方法**：

- `getState()`：返回同步状态的当前值。
- `setState(int newState)`：设置当前同步状态。
- `compareAndSetState(int expect, int update)`：使用 CAS 设置当前状态，该方法能够保证状态设置的原子性。
- 【可重写】`tryAcquire(int arg)`：独占式获取同步状态，获取同步状态成功后，其他线程需要等待该线程释放同步状态才能获取同步状态。
- 【可重写】`tryRelease(int arg)`：独占式释放同步状态。
- 【可重写】`tryAcquireShared(int arg)`：共享式获取同步状态，返回值大于等于 0 ，则表示获取成功；否则，获取失败。
- 【可重写】`tryReleaseShared(int arg)`：共享式释放同步状态。
- 【可重写】`isHeldExclusively()`：当前同步器是否在独占式模式下被线程占用，一般该方法表示是否被当前线程所独占。
- `acquire(int arg)`：独占式获取同步状态。如果当前线程获取同步状态成功，则由该方法返回；否则，将会进入同步队列等待。该方法将会调用**可重写**的 `tryAcquire(int arg)` 方法；
- `acquireInterruptibly(int arg)`：与 `acquire(int arg)` 相同，但是该方法响应中断。当前线程为获取到同步状态而进入到同步队列中，如果当前线程被中断，则该方法会抛出InterruptedException 异常并返回。
- `tryAcquireNanos(int arg, long nanos)`：超时获取同步状态。如果当前线程在 nanos 时间内没有获取到同步状态，那么将会返回 false ，已经获取则返回 true 。
- `acquireShared(int arg)`：共享式获取同步状态，如果当前线程未获取到同步状态，将会进入同步队列等待，与独占式的主要区别是在同一时刻可以有多个线程获取到同步状态；
- `acquireSharedInterruptibly(int arg)`：共享式获取同步状态，响应中断。
- `tryAcquireSharedNanos(int arg, long nanosTimeout)`：共享式获取同步状态，增加超时限制。
- `release(int arg)`：独占式释放同步状态，该方法会在释放同步状态之后，将同步队列中第一个节点包含的线程唤醒。
- `releaseShared(int arg)`：共享式释放同步状态。

从上面的方法看下来，基本上可以分成 **3** 类：

- **独占式**获取与释放同步状态
- **共享式**获取与释放同步状态
- 查询**同步队列**中的等待线程情况

## 1.2 [CLH 同步队列](http://www.iocoder.cn/JUC/sike/aqs-1-clh/)

### 1. 简介

==CLH 同步队列是一个 FIFO **双向**队列，AQS 依赖它来完成同步状态的管理==：

- **当前线程如果获取同步状态失败时，AQS则会将当前线程已经等待状态等信息构造成一个节点（Node）并将其加入到CLH同步队列，同时会阻塞当前线程**
- 当同步状态释放时，**会把首节点唤醒（公平锁）**，使其再次尝试获取同步状态。

### 2. Node

> Node 是 AbstractQueuedSynchronizer 的内部静态类。

```java
static final class Node {
    // 共享
    static final Node SHARED = new Node();
    // 独占
    static final Node EXCLUSIVE = null;
    /**
     * 因为超时或者中断，节点会被设置为取消状态，被取消的节点时不会参与到竞争中的，他会一直保持取消状态不会转变为其他状态*/
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
   * 注意：快速入队与正常入队相比，可以发现，正常入队仅仅比快速入队多一个判断队列是否为空且为空之后的过程
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

- 第 **1** 部分

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

  ```Java
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

