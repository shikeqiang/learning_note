# 一、volatile

## 内存可见性

- 内存可见性(Memory Visibility)是指当某个线程正在使用对象状态 而另一个线程在同时修改该状态，需要确保当一个线程修改了对象 状态后，其他线程能够看到发生的状态变化。
- 可见性错误是指当读操作与写操作在不同的线程中执行时，我们无 法确保执行读操作的线程能适时地看到其他线程写入的值，有时甚 至是根本不可能的事情。
- 可以通过同步来保证对象被安全地发布。除此之外我们也可以 使用一种更加轻量级的 volatile 变量。

## volatile

​	Java 提供了一种稍弱的同步机制，即 volatile 变 量，用来确保将变量的更新操作通知到其他线程。 可以将 volatile 看做一个轻量级的锁，但是又与 锁有些不同:

- 对于多线程，不是一种互斥关系
- 不能保证变量状态的“原子性操作”

# 二、CAS

​	CAS (Compare-And-Swap) 是一种硬件对并发的支持，针对多处理器 操作而设计的处理器中的一种特殊指令，用于管理对共享数据的并 发访问。CAS 是一种无锁的非阻塞算法的实现。

CAS 包含了 3 个操作数:

- 需要读写的内存值 V
- 进行比较的值 A
- 拟写入的新值 B

当且仅当 V 的值等于 A 时，CAS 通过原子方式用新值 B 来更新 V 的 值，否则不会执行任何操作。

```java
// 模拟CAS算法
public class TestCompareAndSwap {
   public static void main(String[] args) {
      final CompareAndSwap cas = new CompareAndSwap();
      for (int i = 0; i < 10; i++) {
         new Thread(new Runnable() {
            
            @Override
            public void run() {
               int expectedValue = cas.get();
               boolean b = cas.compareAndSet(expectedValue, (int)(Math.random() * 101));
               System.out.println(b);
            }
         }).start();
      }
   }
}

class CompareAndSwap{
   private int value;
   
   //获取内存值
   public synchronized int get(){
      return value;
   }
   
   //比较
   public synchronized int compareAndSwap(int expectedValue, int newValue){
      int oldValue = value;
      
      if(oldValue == expectedValue){
         this.value = newValue;
      }
      
      return oldValue;
   }
   
   //设置
   public synchronized boolean compareAndSet(int expectedValue, int newValue){
      return expectedValue == compareAndSwap(expectedValue, newValue);
   }
}
```

## 原子变量

-  类的小工具包，支持在单个变量上解除锁的线程安全编程。事实上，此包中的类可 将 volatile 值、字段和数组元素的概念扩展到那些也提供原子条件更新操作的类。
- 类 AtomicBoolean、AtomicInteger、AtomicLong 和 AtomicReference 的实例各自提供对 相应类型单个变量的访问和更新。每个类也为该类型提供适当的实用工具方法。
- AtomicIntegerArray、AtomicLongArray和AtomicReferenceArray类进一步扩展了原子操作，对这些类型的数组提供了支持。这些类在为其数组元素提供 volatile 访问语义方 面也引人注目，这对于普通数组来说是不受支持的。
- 核心方法:**boolean compareAndSet(expectedValue, updateValue)**
- java.util.concurrent.atomic包下提供了一些原子操作的常用类:
  - AtomicBoolean、AtomicInteger、AtomicLong、AtomicReference
  - AtomicIntegerArray、AtomicLongArray
  - AtomicMarkableReference
  - AtomicReferenceArray
  - AtomicStampedReference

# 三、JUC包

## CountDownLatch

​	一个同步辅助类，**在完成一组正在其他线程中执行的操作 之前。闭锁可以延迟线程的进度直到其到达终止状态，闭锁可以用来确保某些活动直到其他活动都完成才继续执行:**

- 确保某个计算在其需要的所有资源都被初始化之后才继续执行;
- 确保某个服务在其依赖的所有其他服务都已经启动之后才启动;
- 等待直到某个操作所有参与者都准备就绪再继续执行。

## Callable接口

​	Callable 接口类似于 Runnable，两者都是为那些其实例可能被另一个线程执行的类设计的。但是 Runnable 不会返回结果，**并且无法抛出经过检查的异常。**==Callable 需要依赖FutureTask ，FutureTask 也可以用作闭锁。==

```java
/*
 * 一、创建执行线程的方式三：
 * 实现 Callable 接口。 相较于实现 Runnable 接口的方式，方法可以有返回值，并且可以抛出异常。
 * 二、执行 Callable 方式，需要 FutureTask 实现类的支持，用于接收运算结果。
 * FutureTask 是  Future 接口的实现类
 */
public class TestCallable {
   public static void main(String[] args) {
      ThreadDemo td = new ThreadDemo();
//1.执行 Callable 方式，需要 FutureTask 实现类的支持，用于接收运算结果。参数需要实现了Callable接口
      FutureTask<Integer> result = new FutureTask<>(td);
/*
new的线程方法
public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }  
*/    
      new Thread(result).start();
      
      //2.接收线程运算后的结果
      try {
         Integer sum = result.get();  //FutureTask 可用于 闭锁
         System.out.println(sum);
         System.out.println("------------------------------------");
      } catch (InterruptedException | ExecutionException e) {
         e.printStackTrace();
      }
   }
}

class ThreadDemo implements Callable<Integer>{

   @Override
   public Integer call() {
      int sum = 0;
      for (int i = 0; i <= 100000; i++) {
         sum += i;
      }
      return sum;
   }
}
```

## Lock同步锁

​	ReentrantLock 实现了 Lock 接口，并提供了与synchronized 相同的互斥性和内存可见性。但相较于synchronized 提供了更高的处理锁的灵活性。

```java
public class TestLock {
    public static void main(String[] args) {
        Ticket ticket = new Ticket();
      //创建3个线程，参数为实现了Runnable接口的类
        new Thread(ticket, "1号窗口").start();
        new Thread(ticket, "2号窗口").start();
        new Thread(ticket, "3号窗口").start();
    }
}

class Ticket implements Runnable {
    private int tick = 100;
    private Lock lock = new ReentrantLock();

    @Override
    public void run() {
        while (true) {
            lock.lock(); //上锁
            try {
                if (tick > 0) {
                    try {
                        Thread.sleep(200);
                    } catch (InterruptedException e) {
                    }
                    System.out.println(Thread.currentThread().getName() + " 完成售票，余票为：" + --tick);
                }
            } finally {
                lock.unlock(); //释放锁
            }
        }
    }
}
```

## Condition

​	Condition 接口描述了可能会与锁有关联的条件变量。这些变量在用法上与使用 Object.wait 访问的隐式监视器类似，但提供了更强大的 功能。需要特别指出的是，**单个 Lock 可能与多个 Condition 对象关 联。**为了避免兼容性问题，Condition 方法的名称与对应的 Object 版 本中的不同。

- 在 Condition 对象中，与 wait、notify 和 notifyAll 方法对应的分别是await、signal 和 signalAll。
- Condition 实例实质上被绑定到一个锁上。要为特定 Lock 实例获得Condition 实例，请使用其 newCondition() 方法。

# 四、锁相关描述

- 一个对象里面如果有多个synchronized方法，某一个时刻内，只要一个线程去调用 其中的一个synchronized方法了，其它的线程都只能等待，换句话说，某一个时刻 内，只能有唯一一个线程去访问这些synchronized方法
- **锁的是当前对象this，被锁定后，其它的线程都不能进入到当前对象的其它的synchronized方法**
- 加个普通方法后发现和同步锁无关
- 换成两个对象后，不是同一把锁了，情况立刻变化。
- 都换成静态同步方法后，情况又变化
- 所有的非静态同步方法用的都是同一把锁——实例对象本身，也就是说如果一个实例对象的非静态同步方法获取锁后，该实例对象的其他非静态同步方法必须等待获 取锁的方法释放锁后才能获取锁，可是别的实例对象的非静态同步方法因为跟该实 例对象的非静态同步方法用的是不同的锁，所以毋须等待该实例对象已获取锁的非 静态同步方法释放锁就可以获取他们自己的锁。
- 所有的静态同步方法用的也是同一把锁——类对象本身，这两把锁是两个不同的对 象，所以静态同步方法与非静态同步方法之间是不会有竞态条件的。但是一旦一个 静态同步方法获取锁后，其他的静态同步方法都必须等待该方法释放锁后才能获取 锁，而不管是同一个实例对象的静态同步方法之间，还是不同的实例对象的静态同 步方法之间，只要它们同一个类的实例对象!

### 线程池与Fork/Join框架差别

​	相对于一般的线程池实现，fork/join框架的优势体现在对其中包含的任务 的处理方式上。在一般的线程池中，如果一个线程正在执行的任务由于某些 原因无法继续运行，那么该线程会处于等待状态。而在fork/join框架实现中， 如果某个子问题由于等待另外一个子问题的完成而无法继续运行。那么处理 该子问题的线程会主动寻找其他尚未运行的子问题来执行.这种方式减少了 线程的等待时间，提高了性能。



