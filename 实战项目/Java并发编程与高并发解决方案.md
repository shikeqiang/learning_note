# Java并发编程与高并发解决方案

**J.U.C核心由5大块组成：atomic包、locks包、collections包、tools包（AQS）、executor包（线程池）。**

![å¾çæè¿°](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/5aab34060001193d19561164-8152326.jpg)

![å¾çæè¿°](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/5aab34100001a9cd19781256-8152326.jpg)

# **1 基本概念**

## **1.1 并发**

​	同时拥有两个或者多个线程，如果程序在单核处理器上运行多个线程将交替地换入或者换出内存,这些线程是同时“存在"的，每个线程都处于执行过程中的某个状态，如果运行在多核处理器上,此时，程序中的每个线程都将分配到一个处理器核上，因此可以同时运行。

## **1.2 高并发( High Concurrency)**

互联网分布式系统架构设计中必须考虑的因素之一，通常是指，通过设计保证系统能够同时并行处理很多请求.

## **1.3 区别与联系**

- 并发: 多个线程操作相同的资源，保证线程安全，合理使用资源
- 高并发:服务能同时处理很多请求，提高程序性能

# **2 CPU**

## **2.1 CPU 多级缓存**

![image-20190120163526230](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190120163526230-8152326.png)

​	**因为CPU的频率太快了，快到主存跟不上，所以需要CPU cache。**因此，在处理器时钟周期内，CPU常常需要等待主存，浪费资源。**所以cache的出现，是为了缓解CPU和内存之间速度的不匹配问题(结构:cpu-> cache-> memory ).**

### CPU cache的意义

##### 1) 时间局部性：如果某个数据被访问，那么在不久的将来它很可能被再次访问

##### 2) 空间局部性：如果某个数据被访问，那么与它相邻的数据很快也可能被访问

## **2.2 缓存一致性(MESI)**

​	**用于保证多个 CPU cache 之间缓存共享数据的一致**

![image-20190120163957992](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190120163957992-8152326.png)

​	==MESI其实是这四种状态的缩写。==

### M-modified被修改

​	该缓存行只被缓存在该 CPU 的缓存中,并且是被修改过的,与主存中数据是不一致的,**需在未来某个时间点写回主存**,这个时间点是允许在其他CPU 读取主存中相应的内存之前**,当这里的值被写入主存之后,该缓存行状态变为E**

### E-exclusive独享

- 该缓存行只被缓存在该 CPU 的缓存中,未被修改过,与主存中数据一致
- 可在任何时刻当被其他 CPU读取该内存时变成 S 态,被修改时变为 M态

### S-shared共享

​	该缓存行可能被多个 CPU 进行缓存,并且各缓存中的数据与主存数据是一致的。**当有一个CPU修改该缓存行的时候，其他CPU中该缓存行变成Invalid。**

### I-invalid无效

- 乱序执行优化
- 处理器为提高运算速度而做出违背代码原有顺序的优化

### 四种操作(如上图颜色)：

- 本地读取 local read :读本地缓存
- 本地写入 local write : 写本地缓存
- 远端读取 remote rade : 将Memory中的数据读取过来
- 远端写入 remote write : 将数据写回Memory中 

# 3.JAVA 内存模型(JMM)

​	一种规范，规范了java虚拟机与计算机内存如何协同工作的。它规定了**一个线程如何和何时可以看到其他线程修改过的共享变量的值，以及在必须时如何同步地访问共享变量**。 ![这里写图片描述](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/70-8152326.png)

## 堆Heap

​	运行时数据区，有垃圾回收，堆的优势可以动态分配内存大小，生存期也不必事先告诉编译器，因为他是**在运行时动态分配内存。**缺点是由于==运行时动态分配内存，所以存取速度慢一些。==

## 栈Stack:

​	优势存取速度快，速度仅次于计算机的寄存器。**栈的数据是可以共享的**，但是缺点是存在栈中数据的大小与生存期必须是确定的。主要存放基本类型变量，对象据点。要求调用栈和本地变量存放在线程栈上。
​	**静态类型变量跟随类的定义存放在堆上。存放在堆上的对象可以被所持有对这个对象引用的线程访问。**

**如果两个线程同时调用了同一个对象的同一个方法，他们都会访问这个对象的成员变量。但是这两个线程都拥有的是该对象的成员变量（局部变量）的私有拷贝**。—[线程封闭中的堆栈封闭]![image-20190120170418373](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190120170418373-8152326.png)

## CPU Registers(寄存器):

​	==CPU内存的基础，CPU在寄存器上执行操作的速度远大于在主存上执行的速度。这是因为CPU访问寄存器速度远大于主存。==

## CPU Cache Memory(高速缓存):

​	由于计算机的存储设备与处理器的运算速度之间有着几个数量级的差距，所以现代计算机系统都不得不加入一层读写速度尽可能接近处理器运算速度的高级缓存，来作为内存与处理器之间的缓冲。将运算时所使用到的数据复制到缓存中,让运算能快速的进行。当运算结束后，再从缓存同步回内存之中，这样处理器就无需等待缓慢的内存读写了。

## RAM-Main Memory(主存/内存):

​	**运作原理**：当一个CPU需要读取主存的时候，他会将主存中的部分读取到CPU缓存中，甚至他可能将缓存中的部分内容读到他的内部寄存器里面，然后在寄存器中执行操作。当CPU需要将结果回写到主存的时候，他会将内部寄存器中的值刷新到缓存中，然后在某个时间点从缓存中刷回主存。

## Java内存模型抽象结构：

![image-20190120170743534](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190120170743534-8152326.png)

​	**每个线程都有一个私有的本地内存，**本地内存他是java内存模型的一个抽象的概念。它并不是真实存在的，它涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器的优化。本地内存中它存储了该线程以读或写共享变量拷贝的一个副本。

​	从更低的层次来说，主内存就是硬件的内存，是为了获取更高的运行速度，虚拟机及硬件系统可能会让工作内存优先存储于寄存器和高速缓存中，java内存模型中的线程的工作内存是CPU的寄存器和高速缓存的一个抽象的描述。而JVM的静态内存存储模型它只是对内存的一种物理划分而已。**它只局限在内存，而且只局限在JVM的内存。**

## Java内存模型-同步八种操作

​	**按顺序执行，但不一定要连续执行，顺序之间可以插入不同的指令。**

![image-20190120171054550](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190120171054550-8152326.png)

- lock(锁定) ：作用于主内存变量，把一个变量标识为一条线程独占状态
- unlock(解锁) ： 作用于主内存的变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
- read(读取) ： 作用于主内存的变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用
- load(载入) ：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中
- use(使用) ：作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎
- assign(赋值) ： 作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作内存的变量
- store(存储) ： 作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write操作
- write(写入) ：作用于主内存的变量中，它把store操作从工作内存中一个变量的值传送到主内存的变量中

## Java内存模型-同步规则

- 如果要把一个变量从主内存中复制到工作内存，就需要按顺序的执行read和load操作，如果把变量从工作内存中同步回主内存中，就要按顺序的执行store和write操作。但java内存模型只要求上述操作必须按顺序执行，而没有保证必须是连续执行
- 不允许read和load、store和write操作之一单独出现
- 不允许一个线程丢弃它的最近assign的操作，即变量在工作内存中改变了之后必须同步到主内存中
- 不允许一个线程无原因的（没有发生过任何assign操作）把数据从工作内存同步回主内存中
- 一个新的变量只能在主内存中诞生，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量。即就是对一个变量实施use和store操作之前，必须先执行过了assign和load操作。
- 一个变量早同一时刻只允许一条线程对其进行lock操作，但lock操作可以被同一条线程重复执行多次，多次执行lock后，只有执行相同次数的unlock操作，变量才会被解锁。lock和unlock必须是成对出现。
- 如果对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前需要重新执行load或assign操作初始化变量的值。
- 如果一个变量事先没有被lock锁定，则不允许对它执行unlock操作，也不允许去unlock一个被其他线程锁定的变量
- 对一个变量执行unlock操作之前，必须先把此变量同步到主内存中（执行store和write操作）

## 并发的优势与风险![10](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/10-8152326.jpg)

### 风险：

安全性：多个线程共享数据时可能会产生于期望不相符的结果
活跃性：某个操作无法继续进行下去时，就会发生活跃性问题。比如死锁、饥饿问题
性能：线程过多时会使得CPU频繁切换，调度时间增多；同步机制；消耗过多内存。

### 优势：

速度：同时处理多个请求，响应更快；复杂的操作可以分成多个进程同时进行。
设计：程序设计在某些情况下更简单，也可以有更多选择

资源利用：CPU能够在等待IO的时候做一些其他的事情

# 4.线程安全性

​	**线程安全性主要体现在三个方面：原子性、可见性、有序性**

- 原子性:提供了互斥访问，同一时刻只能有一个线程来对它进行操作
- 可见性:一个线程对主内存的修改可以及时的被其他线程观察到
- 有序性:一个线程观察其他线程中的指令执行顺序，由于指令重排序的存在，该观察结果一般杂乱无序。

## 4.1 原子性

​	说到原子性，一共有两个方面需要学习一下，一个是JDK中已经提供好的Atomic包，他们均使用了CAS完成线程的原子性操作，另一个是使用锁的机制来处理线程之间的原子性。锁包括：synchronized、Lock。

```Java
public class AtomicIntegerExample {
    //请求总数
    public static int clientTotal = 5000;
    //同时并发执行的线程数
    public static int threadTotal = 200;
    public static AtomicInteger count = new AtomicInteger(0);

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newCachedThreadPool();
        final Semaphore semaphore = new Semaphore(threadTotal);
        final CountDownLatch countDownLatch = new CountDownLatch(clientTotal);
        for (int i = 0; i < clientTotal; i++) {
            executorService.execute(() -> {
                try {
                    semaphore.acquire();    //判断当前线程是否允许被执行
                    add();
                    semaphore.release();
                } catch (Exception e) {
                    log.error("exception", e);
                }
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        executorService.shutdown();
        log.info("count:{}", count.get());
    }

    private static void add() {
        count.incrementAndGet();
    }
}
```

### atomic包相关的类：

![image-20190121210844397](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190121210844397-8152326.png)

### AtomicInteger

​	示例中，对count变量的+1操作，采用的是incrementAndGet方法，此方法的源码中调用了一个名为unsafe.getAndAddInt的方法：

```Java
public final int incrementAndGet() {
     return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
```

而getAndAddInt是Unsafe这个类里面的方法，具体实现为：

```Java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    return var5;
}
```

​	在此方法中，**方法参数为要操作的对象Object var1、期望底层当前的数值为var2、要修改的数值var4。**定义的**var5为真正从底层取出来的值**。采用do..while循环的方式去获取底层数值并与期望值进行比较，比较成功才将值进行修改。而这个比较再进行修改的方法就是compareAndSwapInt就是我们所说的CAS，它是一系列的接口，比如下面罗列的几个接口。使用native修饰，是底层的方法。CAS取的是compareAndSwap三个单词的首字母。

​	另外，示例代码中的count可以理解为JMM中的工作内存，而这里的底层数值即为主内存。

```java
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```

### AtomicLong 与 LongAdder

LongAdder是java8为我们提供的新的类，跟AtomicLong有相同的效果。首先看一下代码实现：

```Java
//AtomicLong的实现：
//变量声明
public static AtomicLong count = new AtomicLong(0);
//变量操作
count.incrementAndGet();
//变量取值
count.get();

//LongAdder的实现：
//变量声明
public static LongAdder count = new LongAdder();
//变量操作
count.increment();
//变量取值
count
```

​	CAS底层实现是在一个死循环中不断地尝试修改目标值，直到修改成功。如果竞争不激烈的时候，修改成功率很高，否则失败率很高。在失败的时候，这些重复的原子性操作会耗费性能。

**注意： 对于普通类型的long、double变量，JVM允许将64位的读操作或写操作拆成两个32位的操作。**

​	**LongAdder类的实现核心是将热点数据分离，**比如说它可以将AtomicLong内部的内部核心数据value分离成一个数组，每个线程访问时，通过hash等算法映射到其中一个数字进行计数，而最终的计数结果则为这个数组的求和累加，**其中热点数据value会被分离成多个单元的cell，每个cell独自维护内部的值。**当前对象的实际值由所有的cell累计合成，这样热点就进行了有效地分离，并提高了并行度。**这相当于将AtomicLong的单点的更新压力分担到各个节点上。在低并发的时候通过对base的直接更新，可以保障和AtomicLong的性能基本一致。而在高并发的时候通过分散提高了性能。**

源码：

```Java
/**
 * Adds the given value.
 *
 * @param x the value to add
 */
public void add(long x) {
    Cell[] as; long b, v; int m; Cell a;
    if ((as = cells) != null || !casBase(b = base, b + x)) {
        boolean uncontended = true;
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[getProbe() & m]) == null ||
            !(uncontended = a.cas(v = a.value, v + x)))
            longAccumulate(x, null, uncontended);
    }
}

/**
 * Equivalent to {@code add(1)}.
 */
public void increment() {
    add(1L);
}
```

#### LongAdder缺点：

​	如果在统计的时候，如果有并发更新，可能会有统计数据有误差。实际使用中在处理高并发计数的时候，优先使用LongAdder，而不是使用AtomicLong。在线程竞争很低的时候进行计数，使用AtomicLong会简单效率更高一些。比如序列号生成（准确性），全局唯一的，使用AtomicLong。

### AtomicBoolean

​	这个类中值得一提的是它包含了一个名为compareAndSet的方法，这个方法可以做到的是控制一个boolean变量在一件事情执行之前为false，事情执行之后变为true。或者也可以理解为可以控制某一件事只让一个线程执行，并仅能执行一次。 

```Java
public final boolean compareAndSet(boolean expect, boolean update) {
        int e = expect ? 1 : 0;
        int u = update ? 1 : 0;
        return unsafe.compareAndSwapInt(this, valueOffset, e, u);
    }
```

### AtomicIntegerFieldUpdater

​	这个类的核心作用是要更新一个指定的类的某一个字段的值。并且这个字段一定要用volatile修饰同时还不能是static的。

```java
@Slf4j
@ThreadSafe
public class AtomicIntegerFieldUpdaterExample {
    private static AtomicIntegerFieldUpdater<AtomicIntegerFieldUpdaterExample> updater=AtomicIntegerFieldUpdater.newUpdater(AtomicIntegerFieldUpdaterExample.class,"count");
    @Getter
    public volatile int count=100;
    public static void main(String[] args) {
        AtomicIntegerFieldUpdaterExample example=new AtomicIntegerFieldUpdaterExample();
        if (updater.compareAndSet(example,100,120)){
            log.info("update success 1,{}",example.getCount());		//执行
        }
        if(updater.compareAndSet(example,100,120)){
            log.info("update success 2,{}",example.getCount());		//不执行
        }else{
            log.info("update failed,{}",example.getCount());	//执行
        }
    }
}
```

### AtomicStampedReference与CAS的ABA问题

​	CAS操作的时候，其他线程将变量的值A改成了B，但是随后又改成了A，本线程在CAS方法中使用期望值A与当前变量进行比较的时候，发现变量的值未发生改变，于是CAS就将变量的值进行了交换操作。但是实际上变量的值已经被其他的变量改变过，这与设计思想是不符合的。所以就有了AtomicStampedReference。

```Java
private static class Pair<T> {
        final T reference;
        final int stamp;
        private Pair(T reference, int stamp) {
            this.reference = reference;
            this.stamp = stamp;
        }
        static <T> Pair<T> of(T reference, int stamp) {
            return new Pair<T>(reference, stamp);
        }
    }

private volatile Pair<V> pair;

private boolean casPair(Pair<V> cmp, Pair<V> val) {
        return UNSAFE.compareAndSwapObject(this, pairOffset, cmp, val);
    }

public boolean compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp) {
        Pair<V> current = pair;
        return
            expectedReference == current.reference &&
            expectedStamp == current.stamp &&
            ((newReference == current.reference &&   //排除新的引用和新的版本号与底层的值相同的情况
              newStamp == current.stamp) ||
             casPair(current, Pair.of(newReference, newStamp)));
}
```

​	AtomicStampedReference的处理思想是，**每次变量更新的时候，将变量的版本号+1**，之前的ABA问题中，变量经过两次操作以后，变量的版本号就会由1变成3，也就是说只要线程对变量进行过操作，变量的版本号就会发生更改。从而解决了ABA问题。

### AtomicLongArray

​	这个类实际上维护了一个Array数组，我们在对数值进行更新的时候，会多一个索引值让我们更新。

​	原子性，提供了互斥访问，同一时刻只能有一个线程来对它进行操作。那么在java里，保证同一时刻只有一个线程对它进行操作的，除了Atomic包之外，还有锁的机制。**JDK提供锁主要分为两种：synchronized和Lock。**

### synchronized:

​	依赖于JVM去实现锁，因此在这个关键字作用对象的作用范围内，都是同一时刻只能有一个线程对其进行操作的。 synchronized是java中的一个关键字，是一种同步锁。它可以修饰的对象主要有四种：

- **修饰代码块:大括号括起来的代码，作用于调用的对象**
- **修饰方法：整个方法，作用于调用的对象**
- **修饰静态方法：整个静态方法，作用于所有对象**
- **修饰类：==括号括起来的部分，作用于所有对象==**

#### synchronized 修饰一个代码块:

​	被修饰的代码称为同步语句块，作用的范围是大括号括起来的部分。作用的对象是**调用这段代码的对象**。 不同对象之间的操作互不影响 。如果代码块里面都被synchronized修饰的话，那么跟synchronized修饰一个方法一样。

#### synchronized 修饰一个方法:

​	被修饰的方法称为同步方法，作用的范围是大括号括起来的部分，作用的对象是**调用这段代码的对象**。 不同对象之间的操作互不影响 。

​	==如果当前类是一个父类，子类调用父类的被synchronized修饰的方法，不会携带synchronized属性，因为synchronized不属于方法声明的一部分。==

#### synchronized 修饰一个静态方法:

​	作用的范围是synchronized 大括号括起来的部分，作用的对象是**这个类的所有对象**。 同一时间只有一个线程可以执行 。

#### synchronized 修饰一个类:

​	同一时间只有一个线程可以执行 

```Java
public void test(int j){
        synchronized (this){
            for (int i = 0; i < 10; i++) {
                log.info("test - {} - {}",j,i);
            }
        }
    }
public synchronized void test(int j){
        for (int i = 0; i < 10; i++) {
            log.info("test - {} - {}",j,i);
        }
    }
    public static synchronized void test(int j){
        for (int i = 0; i < 10; i++) {
            log.info("test - {} - {}",j,i);
        }
    }
public static void test(int j){
        synchronized (SynchronizedExample.class){
            for (int i = 0; i < 10; i++) {
                log.info("test - {}-{}",j,i);
            }
        }
    }
```

### 原子性操作各方法间的对比

- synchronized:不可中断锁，适合竞争不激烈，可读性好
- Lock：可中断锁，多样化同步，竞争激烈时能维持常态
- Atomic:竞争激烈时能维持常态，比Lock性能好，每次只能同步一个值

## 4.2 可见性

> **可见性即一个线程对主内存的修改可以及时的被其他线程观察到**

### 导致共享变量在线程间不可见的原因

- 线程交叉执行
- 重排序结合线程交叉执行
- 共享变量更新后的值没有在工作内存与主存间及时更新

### JVM处理可见性

​	**JVM对于可见性，提供了synchronized和volatile**

#### JMM关于synchronized的两条规定：

- 线程解锁前，必须把共享变量的最新值刷新到主内存
- **线程加锁时，将清空工作内存中共享变量的值，从而使用共享变量时需要从主内存中重新读取最新的值（注意：加锁与解锁是同一把锁）**

### Volatile:通过加入内存屏障和禁止重排序优化来实现

- ==对volatile变量**写操作**时，会在写操作后加入一条store屏障指令，将本地内存中的共享变量值刷新到主内存。==![è¿éåå¾çæè¿°](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/70-8082672-8152326.png)

- ==对volatile变量**读操作**时，会在读操作前加入一条load屏障指令，从主内存中读取共享变量。==![è¿éåå¾çæè¿°](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/70-20190121225759365-8152326.png)

- volatile的屏障操作都是cpu级别的。
- 适合状态验证，不适合累加值，volatile关键字不具有原子性 

```java
public class CountExample4 {
    // 请求总数
    public static int clientTotal = 5000;

    // 同时并发执行的线程数
    public static int threadTotal = 200;

    public static volatile int count = 0;

    public static void main(String[] args) throws Exception {
        ExecutorService executorService = Executors.newCachedThreadPool();
        final Semaphore semaphore = new Semaphore(threadTotal);
        final CountDownLatch countDownLatch = new CountDownLatch(clientTotal);
        for (int i = 0; i < clientTotal ; i++) {
            executorService.execute(() -> {
                try {
                    semaphore.acquire();
                    add();
                    semaphore.release();
                } catch (Exception e) {
                    log.error("exception", e);
                }
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        executorService.shutdown();
        log.info("count:{}", count);
    }

    private static void add() {
        count++;
    }
}
```

​	多次运行代码我们发现：count的最终结果并不是预期的5000，而是有时为5000，但是大多数时间比5000小，因为在对count++的操作中，jvm对count做了三步操作：

> **1、从主存中取出count的值放入工作变量 count** 
> **2、对工作变量中的count进行+1** 
> **3、将工作变量中的count刷新回主存中**

​	在单线程执行此操作绝对没有问题，但是在多线程环境中，假设有两个线程A、B同时执行count++操作，某一刻A与B同时读取主存中count的值，然后在自己线程对应的工作空间中对count+1，最后又同时将count+1的值写回主存。到此，count+1的值被写回主存两遍，所以导致最终的count值小了1。在整体程序执行过程中，该事件发生一次或多次，自然结果就不正确。 
volatile比较适合做状态标记量（不会涉及到多线程同时读写的操作），而且要保证两点： 
（**1）对变量的写操作不依赖于当前值(数据依赖性)** 

**（2）该变量没有包含在具有其他变量的不变的式子中** 

## 4.3 有序性(happens-before原则)

​	有序性即在Java内存模型中，允许编译器和处理器对指令进行**重排序**，但是重排序过程不会影响到**单线程**程序的执行，却会影响到多线程并发执行的正确性。

### java中保证有序性

​	**java提供了 volatile、synchronized、Lock可以用来保证有序性** 

​	另外，java内存模型具备一些先天的有序性，即不需要任何手段就能得到保证的有序性。通常被我们成为happens-before原则（先行发生原则）。如果两个线程的执行顺序无法从happens-before原则推导出来，那么就不能保证它们的有序性，虚拟机就可以对它们进行重排序。

- 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作
- 锁定规则：一个unlock操作先行发生于后面对同一个锁的lock操作
- volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作（重要）
- 传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C
- 线程启动规则：Thread对象的start()方法先行发生于此线程的每一个动作
- 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
- 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行
- 对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始

# 3.安全发布对象与多种单例模式

## 发布对象

​	使一个对象能够被当前范围之外的代码所使用。 **在我们的日常开发中，我们经常要发布一些对象，比如通过类的非私有方法返回对象的引用，或者通过公有静态变量发布对象。**

## 对象逸出

​	**一种错误的发布。当一个对象还没有构造完成时，就使它被其他线程所见。**

##### 不安全发布对象代码如下：

```Java
public class UnsafePublish {

    private String[] states = {"a", "b", "c"};

    //类的非私有方法，返回私有对象的引用
    public String[] getStates() {
        return states;
    }
    public static void main(String[] args) {
        UnsafePublish unsafePublish = new UnsafePublish();
        log.info("{}", Arrays.toString(unsafePublish.getStates()));

        unsafePublish.getStates()[0] = "d";
        log.info("{}", Arrays.toString(unsafePublish.getStates()));
    }
}
```

​	上面的代码表示通过public访问级别发布了类的域，在类的任何外部的线程都可以访问这些域。我们无法保证其他线程会不会修改这个域，从而使私有域内的值错误（上述代码中就对私有域进行了修改）。

##### 对象逸出代码如下：

```Java
public class Escape {

    private Integer thisCanBeEscape = 0;

    public Escape () {
        new InnerClass();
        thisCanBeEscape = null;
    }

    //内部类构造方法调用外部类的私有域
    private class InnerClass {

        public InnerClass() {
            //this逃逸
            log.info("{}", Escape.this.thisCanBeEscape);
        }
    }

    public static void main(String[] args) {
        new Escape();
    }
}
```

​	上面代码表示：**这个内部类的实例里面==包含了对封装实例的私有域对象的引用，在对象没有被正确构造完成之前就会被发布==，有可能有不安全的因素在里面，会导致this引用在构造期间溢出的错误。**
​	上述代码在函数构造过程中启动了一个线程。无论是隐式的启动还是显式的启动，都会造成这个this引用的溢出。新线程总会在所属对象构造完毕之前就已经看到它了。==因此要在构造函数中创建线程，那么不要启动它，而是应该采用一个专有的start或者初始化的方法统一启动线程。==
​	**这里其实我们可以采用工厂方法和私有构造函数来完成对象创建和监听器的注册等等，这样才可以避免错误。**如果不正确的发布对象会导致两种错误： 
（1）发布线程意外的任何线程都可以看到被发布对象的过期的值 

（2）线程看到的被发布线程的引用是最新的，然而被发布对象的状态却是过期的	

#### 安全发布对象示例（多种单例模式演示）

##### 安全发布对象共有四种方法：

- 1、在静态初始化函数中初始化一个对象引用
- 2、将对象的引用保存到volatile类型域或者AtomicReference对象中
- 3、将对象的引用保存到某个正确构造对象的final类型域中
- 4、将对象的引用保存到一个由锁保护的域中

通过各种单例模式来演示其中的几种方法

##### 1、懒汉式（最简式,线程不安全）

```java
public class SingletonExample {
    //私有构造函数
    private SingletonExample(){
    }

    //单例对象
    private static SingletonExample instance = null;

    //静态工厂方法
    public static SingletonExample getInstance(){
        if(instance==null){
            return new SingletonExample();
        }
        return instance;
    }
}
```

​	在多线程环境下，当两个线程同时访问这个方法，同时制定到instance==null的判断。都判断为null，接下来同时执行new操作。这样类的构造函数被执行了两次。一旦构造函数中涉及到某些资源的处理，那么就会发生错误。所以说最简式是**线程不安全的**

##### 2、懒汉式（synchronized）

```java
public static class SingleTon3 {
    public static SingleTon3 instance = null;

    private SingleTon3() {
    }

    public static synchronized SingleTon3 getInstance() {
        if (instance == null) {
            instance = new SingleTon3();
        }
        return instance;
    }
}
```

​	使用synchronized修饰静态方法后，保证了方法的线程安全性，同一时间只有一个线程访问该方法，会造成性能损耗	。

##### 3、双重同步锁模式

```java
public class SingletonExample {
    // 私有构造函数
    private SingletonExample() {
    }
    // 单例对象
    private static SingletonExample instance = null;
    // 静态的工厂方法
    public static SingletonExample getInstance() {
        if (instance == null) { // 双重检测机制
            synchronized (SingletonExample.class) { // 同步锁
                if (instance == null) {
                    instance = new SingletonExample();
                }
            }
        }
        return instance;
    }
}
```

​	对第二个例子(懒汉式（synchronized))进行了改进，由synchronized修饰方法改为先判断后，再锁定整个类，再加上**双重的检测机制**，保证了最大程度上的避免耗损性能。 但其实**这个方法是线程不安全的**，可能大家会想在多线程情况下，只要有一个线程对类进行了上锁，那么无论如何其他线程也不会执行到new的操作上。接下来我们分析一下线程不安全的原因：

> 这里有一个知识点：CPU指令相关。在上述代码中，执行new操作的时候，CPU一共进行了三次指令 
> **（1）memory = allocate() 分配对象的内存空间** 
>
> **（2）ctorInstance() 初始化对象** 
>
> **（3）instance = memory 设置instance==指向==刚分配的内存**

​	在程序运行过程中，CPU为提高运算速度会做出违背代码原有顺序的优化。我们称之为乱序执行优化或者说是**指令重排序**。 

​	那么上面知识点中的三步指令极有可能被优化为（1）（3）（2）的顺序。当我们有两个线程A与B，A线程遵从132的顺序，经过了两次instance的空值判断后，执行了new操作，**并且cpu在某一瞬间刚结束指令（3），并且还没有执行指令（2）。**而在此时线程B恰巧在进行第一次的instance空值判断，**由于线程A执行完（3）指令，为instance分配了内存，线程B判断instance不为空，直接执行return，返回了instance，这样就出现了错误。** 

![è¿éåå¾çæè¿°](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/70-20190122181758169-8152326.png)

可以在**对象声明时使用volatile关键字修饰，阻止CPU的指令重排。**如：

```java
public static class SingleTon7 {
    private volatile static SingleTon7 instance = null;

    private SingleTon7() {
    }

    public static SingleTon7 getInstance() {
        if (instance == null) {
            synchronized (SingleTon7.class) {
                instance = new SingleTon7();
            }
        }
        return instance;
    }
}
```

##### 4、饿汉式（最简式）

```java
public class SingletonExample {
    // 私有构造函数
    private SingletonExample() {

    }
    // 单例对象
    private static SingletonExample instance = new SingletonExample();

    // 静态的工厂方法
    public static SingletonExample getInstance() {
        return instance;
    }
}
```

​1、饿汉模式由于单例实例是在类装载的时候进行创建，**因此只会被执行一次，所以它是线程安全的。** 
2、该方法存在缺陷：如果构造函数中有着大量的事情操作要做，那么类的装载时间会很长，影响性能。如果只是做的类的构造，却没有引用，那么会造成资源浪费 
3、**饿汉模式适用场景为：（1）私有构造函数在实现的时候没有太多的处理（2）这个类在实例化后肯定会被使用！**	

##### 5、饿汉式（静态块初始化）

```java
public static class SingleTon4 {
    public static SingleTon4 instance = null;

    static {
        instance = new SingleTon4();
    }

    private SingleTon4() {
    }

    public static synchronized SingleTon4 getInstance() {
        return instance;
    }
}
```

1、除了使用静态域直接初始化单例对象，还可以用静态块初始化单例对象。 
2、值得注意的一点是，**静态域与静态块的顺序一定不要反**，在写静态域和静态方法的时候，==一定要注意顺序，不同的静态代码块是按照顺序执行的，它跟我们正常定义的静态方法和普通方法是不一样的。==因为静态代码块和静态域是根据其顺序先后执行的。

##### 6、枚举式

```java
public class SingletonExample {

    private SingletonExample() {
    }

    public static SingletonExample getInstance() {
        return Singleton.INSTANCE.getInstance();
    }

    private enum Singleton {
        INSTANCE;
        private SingletonExample singleton;
		//JVM保证这个方法绝对只调用一次
        Singleton() {
            singleton = new SingletonExample();
        }

        public SingletonExample getInstance() {
            return singleton;
        }
    }
}
```

- 由于枚举类的特殊性，枚举类的构造函数Singleton方法只会被实例化一次，且是这个类被调用之前。这个是JVM保证的。
- 对比懒汉与饿汉模式，它的优势很明显。

# 4.不可变对象

## 1、不可变对象

有一种对象只要它发布了就是安全的，它就是不可变对象。**一个不可变对象需要满足的条件：**

- 对象创建一个其状态不能修改
- 对象所有域都是final类型
- 对象是正确创建的(在对象创建期间，==this引用没有逸出==)

## 2、创建一个不可变对象的方法

#### （1）自己定义 

这里可以采用的方式包括： 
1、将类声明为final，这样这个类就不能被继承。 
2、**将所有的成员声明为私有的，这样就不允许直接访问这些成员。** 
3、对变量不提供set方法，将所有可变的成员声明为final，这样就只能赋值一次。通过构造器初始化所有成员进行深度拷贝。 
4、在get方法中不直接返回对象的本身，而是克隆对象，返回对象的拷贝。

#### （2）使用Java中提供的Collection类中的各种unmodifiable开头的方法 

#### （3）使用Guava中的Immutable开头的类

## 3、final关键字

==final关键字可以修饰类、修饰方法、修饰变量==

### 修饰类：类不能被继承。 

​	基础类型的包装类都是final类型的类。final类中的成员变量可以根据需要设置为final，但是要注意的是，final类中的所有成员方法都会被隐式的指定为final方法

### 修饰方法： 

(1)把方法锁定，以防任何继承类修改它的含义 

(2)效率：在早期的java实现版本中，会将final方法转为内嵌调用。但是如果方法过于庞大，可能看不见效果。一个private方法会被隐式的指定为final方法

### 修饰变量： 

​	基本数据类型变量，在初始化之后，它的值就不能被修改了。

​	**如果是引用类型变量，在它初始化之后便不能再指向另外的对象。** 

当final修饰方法的参数时：同样也是不允许在方法内部对其修改的。 

##### **==被final修饰的引用类型变量，虽然不能重新指向，但是可以修改==**。

## 4、Java:unmodifiable相关方法

​	使用Java的Collection类的unmodifiable相关方法，可以创建不可变对象。unmodifiable相关方法包含：Collection、List、Map、Set…. 

![è¿éåå¾çæè¿°](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/70-20190122232548029.png)



```Java
public static <K,V> Map<K,V> unmodifiableMap(Map<? extends K, ? extends V> m) {
        return new UnmodifiableMap<>(m);
}
private static class UnmodifiableMap<K,V> implements Map<K,V>, Serializable {
    ...
    public V put(K key, V value) {
        throw new UnsupportedOperationException();
    }
    ...
}
```

​	**Collections.unmodifiableMap在执行时，将参数中的map对象进行了转换，转换为Collection类中的内部类 UnmodifiableMap对象。**而 UnmodifiableMap对map的更新方法（比如put、remove等）进行了重写，均返回UnsupportedOperationException异常，这样就做到了map对象的不可变！	

## 5、Guava:Immutable相关类

​	使用Guava的Immutable相关类也可以创建不可变对象。同样包含很多类型：Collection、List、Map、Set…. 

![è¿éåå¾çæè¿°](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/70-20190122233621875.png)

### （1）ImmutableList

​	**对于ImmutableList.of方法，如果传多个参数，需要这样一直写下去，以逗号分隔每个参数。其源码中是这样实现的：**

```java
//单个或少于12个参数时
public static <E> ImmutableList<E> of() {
    return RegularImmutableList.EMPTY;
}

public static <E> ImmutableList<E> of(E element) {
    return new SingletonImmutableList(element);
}

public static <E> ImmutableList<E> of(E e1, E e2) {
    return construct(e1, e2);
}

public static <E> ImmutableList<E> of(E e1, E e2, E e3) {
    return construct(e1, e2, e3);
}
....

//多于12个参数时，参数列表中最后的E...other会以数组形式接收参数
@SafeVarargs
public static <E> ImmutableList<E> of(E e1, E e2, E e3, E e4, E e5,
 E e6, E e7, E e8, E e9, E e10, E e11, E e12, E... others) {
    Object[] array = new Object[12 + others.length];
    array[0] = e1;
    array[1] = e2;
    array[2] = e3;
    array[3] = e4;
    array[4] = e5;
    array[5] = e6;
    array[6] = e7;
    array[7] = e8;
    array[8] = e9;
    array[9] = e10;
    array[10] = e11;
    array[11] = e12;
    System.arraycopy(others, 0, array, 12, others.length);
    return construct(array);
}
```
### （2）ImmutableSet 

​	ImmutableSet除了使用of的方法进行初始化，还可以使用copyof方法，将Collection类型、Iterator类型作为参数。

```java
private final static ImmutableSet set = ImmutableSet.copyOf(list);
private final static ImmutableSet set = ImmutableSet.copyOf(list.iterator());
```

### （3）ImmutableMap 

ImmutableMap有特殊的builder写法：

```java
//参数里面是一对对k-v
private final static ImmutableMap<Integer, Integer> map = ImmutableMap.of(1, 2, 3, 4);
//通过put方法放进去然后调用build方法
private final static ImmutableMap<Integer, Integer> map2 = ImmutableMap.<Integer, Integer>builder()
        .put(1, 2).put(3, 4).put(5, 6).build();
```

# 5.线程封闭 - ThreadLocal

## 1、线程封闭

​	**它其实就是把对象封装到一个线程里，只有一个线程能看到这个对象，那么这个对象就算不是线程安全的，也不会出现任何线程安全方面的问题。**

线程封闭技术有一个常见的应用：

​	数据库连接对应jdbc的Connection对象，Connection对象在实现的时候并没有对线程安全做太多的处理，jdbc的规范里也没有要求Connection对象必须是线程安全的。 
​	实际在服务器应用程序中，**线程从连接池获取了一个Connection对象，使用完再把Connection对象返回给连接池，由于大多数请求都是由单线程采用同步的方式来处理的，并且在Connection对象返回之前，连接池不会将它分配给其他线程。**因此这种连接管理模式处理请求时隐含的将Connection对象封闭在线程里面，这样我们使用的connection对象虽然本身不是线程安全的，但是它通过线程封闭也做到了线程安全。

![image-20190127214746197](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190127214746197.png)

## 2、线程封闭的种类：

（1）Ad-hoc 线程封闭：

Ad-hoc线程封闭是指，维护线程封闭性的职责完全由程序实现来承担。Ad-hoc线程封闭是非常脆弱的，因为没有任何一种语言特性，例如可见性修饰符或局部变量，能将对象封闭到目标线程上。事实上，对线程封闭对象（例如，GUI应用程序中的可视化组件或数据模型等）的引用通常保存在公有变量中。

（2）堆栈封闭： 
堆栈封闭其实就是方法中定义局部变量。不存在并发问题。多个线程访问一个方法的时候，方法中的局部变量都会被拷贝一份到线程的栈中（Java内存模型），所以局部变量是不会被多个线程所共享的。

（3）ThreadLocal线程封闭： 

它是一个特别好的封闭方法，其实ThreadLocal内部维护了一个map,map的key是每个线程的名称，而map的value就是我们要封闭的对象。ThreadLocal提供了get、set、remove方法，每个操作都是基于当前线程的，所以它是线程安全的。 

![è¿éåå¾çæè¿°](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/70-20190127200849231.png)

```java
//ThreadLocal的get方法源码
    public T get() {
        Thread t = Thread.currentThread();//当前线程对象
        ThreadLocalMap map = getMap(t);//get操作基于当前线程
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```

## 3、Springboot框架中使用ThreadLocal

#### （1）创建一个包含ThreadLocal对象的类，并提供基础的添加、删除、获取操作。

```java
public class RequestHolder {

    private final static ThreadLocal<Long> requestHolder = new ThreadLocal<>();
	//后面会存入线程的id
    public static void add(Long id) {
        requestHolder.set(id);
    }

    public static Long getId() {
        return requestHolder.get();
    }

    public static void remove() {
        requestHolder.remove();
    }
}
```

#### （2）创建Filter，在Filter中对ThreadLocal做**添加操作**。

```java
public class HttpFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest,
                         ServletResponse servletResponse,
                         FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        //打印当前线程的ID与请求路径
        log.info("do filter, {}, {}", Thread.currentThread().getId(), request.getServletPath());
        //将当前线程ID添加到ThreadLocal中
        RequestHolder.add(Thread.currentThread().getId());
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {

    }
}
```

#### （3)创建controller，在controller中**获取到filter中存入的值**

```java
@Controller
@RequestMapping("/threadLocal")
public class ThreadLocalController {

    @RequestMapping("/test")
    @ResponseBody
    public Long test() {
        return RequestHolder.getId();
    }
}
```

#### （4）创建拦截器Interceptor，在拦截器中**删除刚才添加的值**

```java
public class HttpInterceptor extends HandlerInterceptorAdapter {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) throws Exception {
        log.info("preHandle");
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, 
                            Object handler, Exception ex) throws Exception {
        RequestHolder.remove();
        log.info("afterCompletion");
        return;
    }
}
```

#### （5）在springboot的启动类Application中注册filter与Interceptor。配置Interceptor的话，要继承WebMvcConfigurerAdapter.	

```java
@SpringBootApplication
public class ConcurrencyApplication extends WebMvcConfigurerAdapter {

    public static void main(String[] args) {
        SpringApplication.run(ConcurrencyApplication.class, args);
    }

    //注册过滤器
    @Bean
    public FilterRegistrationBean httpFilter() {
        FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        registrationBean.setFilter(new HttpFilter());
        //设置要过滤的url
        registrationBean.addUrlPatterns("/threadLocal/*");
        return registrationBean;
    }

    //注册拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new HttpInterceptor()).addPathPatterns("/**");
    }
}
```

​	![image-20190127214717040](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190127214717040.png)

​	从控制台的打印日志我们可以看出，首先filter过滤器先获取到我们当前的线程ID为40、我们当前的请求路径为/threadLocal/test ，紧接着进入了我们的Interceptor的preHandle方法中，打印了preHandle字样。最后进入了我们的Interceptor的afterCompletion方法，删除了我们之前存入的值，并打印了afterCompletion字样。

# 6.线程不安全类、同步容器

## 1、线程不安全的类

​	如果一个类的对象同时可以被多个线程访问，并且你不做特殊的同步或并发处理，那么它就很容易表现出线程不安全的现象。比如抛出异常、逻辑处理错误… 
下面列举一下常见的线程不安全的类及对应的线程安全类：

### （1）StringBuilder 与 StringBuffer

StringBuilder是线程不安全的，而StringBuffer是线程安全的。分析源码：StringBuffer的方法使用了synchronized关键字修饰。

```java
	@Override
    public synchronized StringBuffer append(String str) {
        toStringCache = null;
        super.append(str);
        return this;
    }
```

### （2）SimpleDateFormat(日期转换的类) 与 jodatime包

SimpleDateFormat 类在处理时间的时候，如下写法是线程不安全的：

```java
private static SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyyMMdd");

//线程调用方法
private static void update() {
    try {
        simpleDateFormat.parse("20180208");
    } catch (Exception e) {
        log.error("parse exception", e);
    }
}
```

但是我们可以变换其为线程安全的写法：在每次转换的时候使用线程封闭，新建变量

```java
private static void update() {
    try {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyyMMdd");
        simpleDateFormat.parse("20180208");
    } catch (Exception e) {
        log.error("parse exception", e);
    }
}
```

另外我们也可以使用jodatime插件来转换时间：其可以保证线程安全性 
Joda 类具有不可变性，因此它们的实例无法被修改。（不可变类的一个优点就是它们是线程安全的）

```java
private static DateTimeFormatter dateTimeFormatter = DateTimeFormat.forPattern("yyyyMMdd");

private static void update(int i) {
    log.info("{}, {}", i, DateTime.parse("20180208", dateTimeFormatter).toDate());
}
```

### （3）ArrayList,HashSet,HashMap 等Collection类

​	ArrayList,HashSet,HashMap 等Collection类均是线程不安全的，我们以ArrayList举例分析一下源码： 

#### 1、ArrayList的基本属性： 

​	在声明时使用了**transient** 关键字，**此关键字意为在采用Java默认的序列化机制的时候，被该关键字修饰的属性不会被序列化。**而ArrayList实现了序列化接口，自己定义了序列化方法。

```Java
//对象数组：ArrayList的底层数据结构
private transient Object[] elementData;
//elementData中已存放的元素的个数
private int size;
//默认数组容量
private static final int DEFAULT_CAPACITY = 10;
```

#### 2、初始化

```java
/**
 * 创建一个容量为initialCapacity的空的（size==0）对象数组
 */
 public ArrayList(int initialCapacity) {
    super();//即父类protected AbstractList() {}
    if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity:" + initialCapacity);
    this.elementData = new Object[initialCapacity];
}

/**
 * 默认初始化一个容量为10的对象数组
 */
 public ArrayList() {
    this(10);
 }
```

#### 3、添加元素方法（重点）

```java
//每次添加时将数组扩容1，然后再赋值
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!扩容
    elementData[size++] = e;
    return true;
}
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

#### 4、总结：

​	**==ArrayList每次对内容进行插入操作的时候，都会做扩容处理==，这是ArrayList的优点（无容量的限制）**，同时也是缺点，线程不安全。
一个 ArrayList ，在添加一个元素的时候，它可能会有两步来完成：

- 在 Items[Size] 的位置存放此元素；
- 增大 Size 的值。

​	在单线程运行的情况下，如果 Size = 0，添加一个元素后，此元素在位置 0，而且 Size=1； 而如果是在多线程情况下，比如有两个线程，线**程 A 先将元素存放在位置 0。但是此时 CPU 调度线程A暂停，线程 B 得到运行的机会。线程B也向此 ArrayList 添加元素，因为此时 Size 仍然等于 0 （注意，我们假设的是添加一个元素是要两个步骤哦，而线程A仅仅完成了步骤1）**，所以线程B也将元素存放在位置0。然后线程A和线程B都继续运行，都增加 Size 的值。 那好，现在我们来看看 ArrayList 的情况，**元素实际上只有一个，存放在位置 0，而 Size 却等于 2。这就是“线程不安全”了。**

## 2、同步容器

​	**同步容器分两类，一种是Java提供好的类，另一类是Collections类中的相关同步方法。**

### （1）ArrayList的线程安全类：Vector,Stack

​	Vector实现了List接口，Vector实际上就是一个数组，和ArrayList非常的类似，**但是内部的方法是使用synchronized修饰过的方法。** 

​	Stack它的方法也是使用synchronized修饰了，继承了Vector，实际上就是栈 。

但是==Vector也不是完全的线程安全的==，比如： 

#### **错误[1]：**删除与获取并发操作

```java
public class VectorExample2 {
    private static Vector<Integer> vector = new Vector<>();

    public static void main(String[] args) {
        while (true) {
            for (int i = 0; i < 10; i++) {
                vector.add(i);
            }
            Thread thread1 = new Thread() {
                public void run() {
                    for (int i = 0; i < vector.size(); i++) {
                        vector.remove(i);
                    }
                }
            };
            Thread thread2 = new Thread() {
                public void run() {
                    for (int i = 0; i < vector.size(); i++) {
                        vector.get(i);
                    }
                }
            };
            thread1.start();
            thread2.start();
        }
    }
}
```

运行结果：报错java.lang.ArrayIndexOutOfBoundsException: Array index out of range 	(数组越界)
原因分析：

​	**同时发生获取与删除的操作。**当两个线程在同一时间都判断了vector的size，假设都判断为9，而下一刻线程1执行了remove操作，随后线程2才去get，所以就出现了错误。**synchronized关键字可以保证同一时间只有一个线程执行该方法，但是多个线程同时分别执行remove、add、get操作的时候就无法控制了。**

#### 错误[2]：使用foreach/iterator遍历Vector的时候进行增删操作

```java
public class VectorExample3 {
    // 报错java.util.ConcurrentModificationException，只要不是第一个和最后一个就不会报错
    private static void test1(Vector<Integer> v1) { // foreach
        for (Integer i : v1) {
            if (i.equals(3)) {
                v1.remove(i);
            }
        }
    }

    // 报错java.util.ConcurrentModificationException
    private static void test2(Vector<Integer> v1) { // iterator
        Iterator<Integer> iterator = v1.iterator();
        while (iterator.hasNext()) {
            Integer i = iterator.next();
            if (i.equals(3)) {
                v1.remove(i);
            }
        }
    }

    // success
    private static void test3(Vector<Integer> v1) { // for
        for (int i = 0; i < v1.size(); i++) {
            if (v1.get(i).equals(3)) {
                v1.remove(i);
            }
        }
    }

    public static void main(String[] args) {
        Vector<Integer> vector = new Vector<>();
        vector.add(1);
        vector.add(2);
        vector.add(3);
       // vector.add(4);
        test1(vector);
    }
}
```

解决办法：在使用iterator进行增删操作的时候，加上Lock或者synchronized同步措施或者并发容器，或者先对要修改的值，在迭代器中先用一个值存起来，在循环外在做修改。

### （2）HashMap的线程安全类：HashTable

​源码分析：

- 保证安全性：使用了synchronized修饰
- 不允许空值（在代码中特殊做了判断）
- HashMap和HashTable都使用哈希表来存储键值对。在数据结构上是基本相同的，都创建了一个继承自Map.Entry的私有的内部类Entry，每一个Entry对象表示存储在哈希表中的一个键值对。	

> ==Entry对象唯一表示一个键值对==，有四个属性： 
> -K key 键对象 
> -V value 值对象 
> -int hash 键对象的hash值 
> -Entry entry 指向链表中下一个Entry对象，可为null，表示当前Entry对象在链表尾部

```java
public synchronized V put(K key, V value) {
    // Make sure the value is not null
    if (value == null) {
        throw new NullPointerException();
    }

    // Makes sure the key is not already in the hashtable.
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    for(; entry != null ; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }

    addEntry(hash, key, value, index);
    return null;
}
```

### （3）Collections类中的相关同步方法

​	==Collections类中提供了一系列的线程安全方法用于处理ArrayList等线程不安全的Collection类:==

![è¿éåå¾çæè¿°](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/70-20190128185337900.png)

使用方法：

```java
//定义,通过Collections的同步方法传入不同的数据类型
private static List<Integer> list = Collections.synchronizedList(Lists.newArrayList());
//多线程调用方法
private static void update(int i) {
    list.add(i);
}
```

源码分析： 
内部类。内部操作的方法使用了synchronized修饰符

```java
static class SynchronizedList<E>
        extends SynchronizedCollection<E>
        implements List<E> {
        ...
        public E get(int index) {
            synchronized (mutex) {return list.get(index);}
        }
        public E set(int index, E element) {
            synchronized (mutex) {return list.set(index, element);}
        }
        public void add(int index, E element) {
            synchronized (mutex) {list.add(index, element);}
        }
        public E remove(int index) {
            synchronized (mutex) {return list.remove(index);}
        }
        ...
 }
```

# 7.并发容器 J.U.C 

## 7.1 线程安全的集合与Map

### 1、概述

​	Java并发容器JUC是三个单词的缩写。是JDK下面的一个包名。即Java.util.concurrency。 上面描述了ArrayList、HashMap、HashSet对应的同步容器保证其线程安全。

### 2、ArrayList –> CopyOnWriteArrayList

​	**CopyOnWriteArrayList 写操作时复制，当有新元素添加到集合中时，从原有的数组中拷贝一份出来，然后在新的数组上作写操作，将原来的数组指向新的数组。**==整个数组的add操作都是在锁的保护下进行的，防止并发时复制多份副本。读操作是在原数组中进行，不需要加锁。==

缺点： 

1.写操作时复制数组消耗内存，**如果元素比较多时候，容易导致young gc 和full gc。** 
2.不能用于实时读的场景。由于复制和add操作等需要时间，故读取时可能读到旧值。 能做到最终一致性，但无法满足实时性的要求，==更适合读多写少的场景。== 
**如果无法知道数组有多大，或者add,set操作有多少，慎用此类,在大量的复制副本的过程中很容易出错。**

CopyOnWriteArrayList设计思想： 

**1.读写分离    2.最终一致性     3.使用时另外开辟空间，防止并发冲突**

源码分析：

```java
//构造方法
public CopyOnWriteArrayList(Collection<? extends E> c) {
    Object[] elements;//使用对象数组来承载数据
    if (c.getClass() == CopyOnWriteArrayList.class)
        elements = ((CopyOnWriteArrayList<?>)c).getArray();
    else {
        elements = c.toArray();
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elements.getClass() != Object[].class)
            elements = Arrays.copyOf(elements, elements.length, Object[].class);
    }
    setArray(elements);
}

//添加元素方法
public boolean add(E e) {
    final ReentrantLock lock = this.lock;//使用重入锁，保证线程安全
    lock.lock();
    try {
        Object[] elements = getArray();//获取当前数组数据
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);//复制当前数组并且扩容+1
        newElements[len] = e;//将要添加的数据放入新数组
        setArray(newElements);//将原来的数组指向新的数组
        return true;
    } finally {
        lock.unlock();
    }
}

//将原来的数组指向新的数组
final void setArray(Object[] a) {
        array = a;
    }

//获取数据方法，与普通的get没什么差别
private E get(Object[] a, int index) {
    return (E) a[index];
}
```

​	==读的时候在原数组读，不用加锁，写的时候需要加锁。==

### 3、HashSet –> CopyOnWriteArraySet

​	它是线程安全的，**底层实现使用的是CopyOnWriteArrayList，因此它也适用于大小很小的set集合，只读操作远大于可变操作。**==因为他需要copy整个数组，所以包括add、remove、set它的开销相对于大一些。==

迭代器不支持可变的remove操作。使用迭代器遍历的时候速度很快，而且不会与其他线程发生冲突。

源码分析：

```java
//构造方法
public CopyOnWriteArraySet() {
    al = new CopyOnWriteArrayList<E>();//底层使用CopyOnWriteArrayList
}

//添加元素方法，基本实现原理与CopyOnWriteArrayList相同
private boolean addIfAbsent(E e, Object[] snapshot) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] current = getArray();
        int len = current.length;
        if (snapshot != current) {//添加了元素去重操作
            // Optimize for lost race to another addXXX operation
            int common = Math.min(snapshot.length, len);
            for (int i = 0; i < common; i++)
                if (current[i] != snapshot[i] && eq(e, current[i]))
                    return false;
            if (indexOf(e, current, common, len) >= 0)
                    return false;
        }
        Object[] newElements = Arrays.copyOf(current, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

### 4、TreeSet –> ConcurrentSkipListSet

​	它是JDK6新增的类，同TreeSet一样支持自然排序，并且可以在构造的时候自己定义比较器。**同其他set集合，是基于map集合的（基于ConcurrentSkipListMap）**，==在多线程环境下，里面的contains、add、remove操作都是线程安全的。==
​	**多个线程可以安全的并发的执行插入、移除、和访问操作。但是对于批量操作addAll、removeAll、retainAll和containsAll并不能保证以原子方式执行，**==原因是addAll、removeAll、retainAll底层调用的还是contains、add、remove方法，只能保证每一次的执行是原子性的，代表在单一执行操纵时不会被打断，但是不能保证每一次批量操作都不会被打断==。在使用批量操作时，还是需要手动加上同步操作的。**不允许使用null元素的，它无法可靠的将参数及返回值与不存在的元素区分开来。**

源码分析：

```java
//构造方法
public ConcurrentSkipListSet() {
    m = new ConcurrentSkipListMap<E,Object>();//使用ConcurrentSkipListMap实现
}
```

### 5、HashMap –> ConcurrentHashMap

​	**不允许空值，在实际的应用中除了少数的插入操作和删除操作外，绝大多数我们使用map都是读取操作。**而且读操作大多数都是成功的。基于这个前提，**它针对读操作做了大量的优化。因此这个类在高并发环境下有特别好的表现。**
​	ConcurrentHashMap作为Concurrent一族，其有着高效地并发操作，相比Hashtable的笨重，ConcurrentHashMap则更胜一筹了。
​	在1.8版本以前，ConcurrentHashMap采用分段锁的概念，使锁更加细化，但是1.8已经改变了这种思路，而是利用CAS+Synchronized来保证并发更新的安全，当然底层采用数组+链表+红黑树的存储结构。

源码分析：https://blog.csdn.net/striveb/article/details/84106768

### 6、TreeMap –> ConcurrentSkipListMap(可对key进行排序)

​	底层实现采用SkipList跳表。曾经有人用ConcurrentHashMap与ConcurrentSkipListMap做性能测试，在4个线程1.6W的数据条件下，前者的数据存取速度是后者的4倍左右。但是ConcurrentSkipListMap有几个前者不能比拟的优点： 
1、Key是有序的 
2、支持更高的并发，存储时间与线程数无关

### 7、安全共享对象策略

- 线程限制：一个被线程限制的对象，由**线程独占**，并且只能被占有它的线程修改
- 共享只读：一个共享只读的对象，在没有额外同步的情况下，**可以被多个线程并发访问，但是任何线程都不能修改它**
- 线程安全对象：一个线程安全的对象或者容器，在内部通过==同步机制来保障线程安全==，多以其他线程无需额外的同步就可以通过公共接口随意访问他
- 被守护对象：**被守护对象只能通过获取特定的锁来访问。**

## 7.2 并发容器J.U.C -- AQS组件CountDownLatch、Semaphore、CyclicBarrier

### 1、AQS简介

​	AQS全名：AbstractQueuedSynchronizer，是并发容器J.U.C（java.lang.concurrent）下locks包内的一个类。它实现了一个**FIFO**(FirstIn、FisrtOut先进先出)的队列。底层实现的数据结构是一个**双向链表**。 	

![è¿éåå¾çæè¿°](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/70-20190130131322898.png)

Sync queue：同步队列，是一个双向链表。包括head节点和tail节点。head节点主要用作后续的调度。 
Condition queue：非必须，单向链表。当程序中存在cindition的时候才会存在此列表。

### 2、AQS设计思想

- 使用Node实现FIFO队列，可以用于构建锁或者其他同步装置的基础框架。

- 利用int类型标识状态。**在AQS类中有一个叫做state的成员变量**

  ```java
  /**
   * The synchronization state.
   */
  private volatile int state;
  ```

- 基于AQS有一个同步组件，叫做ReentrantLock。**在这个组件里，stste表示获取锁的线程数，假如state=0，表示还没有线程获取锁，1表示有线程获取了锁。大于1表示重入锁的数量。**
- 继承：子类通过继承并通过实现它的方法管理其状态（acquire和release方法操纵状态）。
- 可以同时实现排它锁和共享锁模式（独占、共享），站在一个使用者的角度，**AQS的功能主要分为两类：独占和共享。**它的所有子类中，要么实现并使用了它的独占功能的api，要么使用了共享锁的功能，而不会同时使用两套api，即便是最有名的子类ReentrantReadWriteLock也是通过两个内部类读锁和写锁分别实现了两套api来实现的。

### 3、AQS的大致实现思路

​	AQS内部维护了一个CLH队列来管理锁。**线程会首先尝试获取锁，如果失败就将当前线程及等待状态等信息包装成一个node节点加入到同步队列sync queue里。** 
​	接着会不断的循环尝试获取锁，==条件是当前节点为head的直接后继才会尝试==。如果失败就会阻塞自己直到自己被唤醒。而当持有锁的线程释放锁的时候，会唤醒队列中的后继线程。

### 4、AQS组件：CountDownLatch

![è¿éåå¾çæè¿°](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/70-20190130131859602.png)

​	**通过一个计数来保证线程是否需要被阻塞。实现一个或多个线程等待其他线程执行的场景。**

​	我们定义一个CountDownLatch，通过给定的计数器为其初始化，该计数器是原子性操作，**保证同时只有一个线程去操作该计数器。**调用该类await方法的线程会一直处于阻塞状态。**只有其他线程调用countDown方法（每次使计数器-1），使计数器归零才能继续执行。**

CountDownLatch的await方法还有重载形式，**可以设置等待的时间，如果超过此时间，计数器还未清零，则不继续等待,会执行await下面的代码，但是之前给定的线程还是会执行完。**

### 5、AQS组件：Semaphore

​	**==用于保证同一时间并发访问线程的数目==。**信号量在操作系统中是很重要的概念，Java并发库里的Semaphore就可以很轻松的完成类似操作系统信号量的控制。**Semaphore可以很容易控制系统中某个资源被同时访问的线程个数。**在前面的链表中，链表正常是可以保存无限个节点的，而Semaphore可以实现有限大小的列表。

使用场景：**仅能提供有限访问的资源。比如数据库连接。**==Semaphore使用acquire方法和release方法来实现控制：==

### 6、AQS组件：CyclicBarrier

![è¿éåå¾çæè¿°](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/70-20190130132035321.png)

从下往上看，当达到设置的值的时候，其他线程就会继续执行。

​	这也是一个同步辅助类，它允许一组线程相互等待，**直到到达某个公共的屏障点（循环屏障）**。通过它可以完成多个线程之间相互等待，只有当每个线程都准备就绪后才能继续往下执行后面的操作。**每当有一个线程执行了await方法，计数器就会执行+1操作，待计数器达到预定的值，所有的线程再同时继续执行。**==由于计数器释放之后可以重用（reset方法），所以称之为循环屏障。==

#### CyclicBarrier与CountDownLatch区别： 

1、计数器可重复用 

2、==描述一个或多个线程等待其他线程的关系/多个线程相互等待==

## 7.3 并发容器J.U.C -- AQS组件 锁：ReentrantLock、ReentrantReadWriteLock、StempedLock

### 1、ReentrantLock

​	**java中有两类锁，一类是Synchronized，而另一类就是J.U.C中提供的锁。**ReentrantLock与Synchronized都是可重入锁，本质上都是lock与unlock的操作。接下来我们介绍三种J.U.C中的锁，其中 ReentrantLock使用synchronized与之比对介绍。

#### 1.1 ReentrantLock与synchronized的区别

- 可重入性：两者的锁都是可重入的，差别不大，有线程进入锁，锁的计数器自增1，等下降为0时才可以释放锁
- 锁的实现：synchronized是基于JVM实现的（用户很难见到，无法了解其实现），ReentrantLock是JDK实现的。
- 性能区别：在最初的时候，二者的性能差别差很多，当synchronized引入了偏向锁、轻量级锁（自选锁）后，二者的性能差别不大，官方推荐synchronized（写法更容易、在优化时其实是借用了ReentrantLock的CAS技术，试图在用户态就把问题解决，避免进入内核态造成线程阻塞）
- 功能区别： 
  （1）便利性：synchronized更便利，它是由编译器保证加锁与释放。**ReentrantLock是需要手动释放锁，所以为了避免忘记手工释放锁造成死锁，所以最好在finally中声明释放锁。** 
  （2）锁的细粒度和灵活度，ReentrantLock优于synchronized

#### 1.2 ReentrantLock独有的功能

- 可以指定是公平锁还是非公平锁，sync只能是非公平锁。（所谓公平锁就是先等待的线程先获得锁）
- 提供了一个Condition类，**可以分组唤醒需要唤醒的线程**。不像是synchronized要么随机唤醒一个线程，要么全部唤醒。
- ==提供能够中断等待锁的线程的机制==，通过lock.lockInterruptibly()实现，这种机制 ReentrantLock是一种自选锁，通过循环调用CAS操作来实现加锁。**性能比较好的原因是避免了进入内核态的阻塞状态。**

​	J.U.C包中的锁定类是用于高级情况和高级用户的工具，除非说你对Lock的高级特性有特别清楚的了解以及有明确的需要，或这有明确的证据表明同步已经成为可伸缩性的瓶颈的时候，否则我们还是继续使用synchronized。相比较这些高级的锁定类，synchronized还是有一些优势的，比如synchronized会释放锁。还有当JVM使用synchronized管理锁定请求和释放时，JVM在生成线程转储时能够包括锁定信息，这些信息对调试非常有价值，它们可以标识死锁以及其他异常行为的来源。

```java
//创建锁：使用Lock对象声明，使用ReentrantLock接口创建
private final static Lock lock = new ReentrantLock();
//使用锁：在需要被加锁的方法中使用
private static void add() {
    lock.lock();
    try {
        count++;
    } finally {
        lock.unlock();
    }
}

//初始化方面：
//在new ReentrantLock的时候默认给了一个不公平锁
public ReentrantLock() {
    sync = new NonfairSync();
}
//也可以加参数来初始化指定使用公平锁还是不公平锁
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

#### 1.3 *ReentrantLock*内置函数（部分）

##### 基础特性：

- tryLock()：仅在调用时锁定未被另一个线程保持的情况下才获取锁定。
- tryLock(long timeout, TimeUnit unit)：如果锁定在给定的时间内没有被另一个线程保持且当前线程没有被中断，则获取这个锁定。
- lockInterruptbily：如果当前线程没有被中断的话，那么就获取锁定。如果中断了就抛出异常。
- isLocked：查询此锁定是否由任意线程保持
- isHeldByCurrentThread：查询当前线程是否保持锁定状态。
- isFair：判断是不是公平锁 

##### Condition相关特性：

- hasQueuedThread(Thread)：查询指定线程是否在等待获取此锁定
- hasQueuedThreads()：查询是否有线程在等待获取此锁定
- getHoldCount()：查询当前线程保持锁定的个数，也就是调用Lock方法的个数 

##### Condition的使用

Condition可以非常灵活的操作线程的唤醒，下面是一个线程等待与唤醒的例子，其中用1234序号标出了日志输出顺序

```java
public class ConditionExample {
    public static void main(String[] args) {
        ReentrantLock reentrantLock = new ReentrantLock();
        Condition condition = reentrantLock.newCondition();//创建condition
        //线程1
        new Thread(() -> {
            try {
                reentrantLock.lock();   //加入AQS等待队列
                log.info("wait signal"); // 1，输出等待信号
                condition.await();      //从AQS队列中移除，即锁的释放,然后加入condition的队列
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.info("get signal"); // 4
            reentrantLock.unlock();
        }).start();
        //线程2
        new Thread(() -> {
            reentrantLock.lock();   //因为线程1释放锁后获取锁，加入AQS等待队列
            log.info("get lock"); // 2
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            condition.signalAll();//发送信号,线程1中condition的节点加入AQS，还未被唤醒
            log.info("send signal"); // 3
            reentrantLock.unlock(); //释放锁，AQS中线程1被唤醒
        }).start();
    }
}
```

输出过程讲解：

1、线程1调用了reentrantLock.lock()，线程进入AQS等待队列，输出1号log 
2、接着调用了awiat方法，线程从AQS队列中移除，锁释放，直接加入condition的等待队列中 
3、线程2因为线程1释放了锁，拿到了锁，输出2号log 
4、线程2执行condition.signalAll()发送信号，输出3号log 
5、condition队列中线程1的节点接收到信号，从condition队列中拿出来放入到了AQS的等待队列,这时线程1并没有被唤醒。 
6、线程2调用unlock释放锁，因为AQS队列中只有线程1，因此AQS释放锁按照从头到尾的顺序，唤醒线程1 

7、线程1继续执行，输出4号log，并进行unlock操作。

### 2、读写锁：ReentrantReadWriteLock读写锁

​	在没有任何读写锁的时候才可以取得写入锁(悲观读取，容易写线程饥饿)，也就是说如果一直存在读操作，那么写锁一直在等待没有读的情况出现，这样我的写锁就永远也获取不到，就会造成等待获取写锁的线程饥饿。 平时使用的场景并不多。

### 3、票据锁：StampedLock

​	**它控制锁有三种模式（写、读、乐观读）。**一个StempedLock的状态是由版本和模式两个部分组成。锁获取方法返回一个数字作为票据（stamp），他用相应的锁状态表示并控制相关的访问。**数字0表示没有写锁被锁写访问，在读锁上分为悲观锁和乐观锁。**

> 乐观读： 
>
> 如果读的操作很多写的很少，我们可以乐观的认为读的操作与写的操作同时发生的情况很少，因此不悲观的使用完全的读取锁定。程序可以查看读取资料之后是否遭到写入资料的变更，再采取之后的措施。

### 4、总结

1、当只有少量竞争者，使用synchronized 
2、竞争者不少但是线程增长的趋势是能预估的，使用ReetrantLock 
3、synchronized不会造成死锁，jvm会自动释放死锁。

4、synchronized是JVM层面实现的锁，执行异常的时候JVM会释放锁定，其他三种锁是对象层面的锁定，要在finally中执行unlock操作。

## 7.4 并发容器J.U.C -- 组件FutureTask、ForkJoin、BlockingQueue

### 1.FutureTask

​	FutureTask是J.U.C中的类，是一个可删除的异步计算类。这个类提供了Future接口的的基本实现，**使用相关方法启动和取消计算，查询计算是否完成，并检索计算结果。**只有在计算完成时才能使用get方法检索结果;如果计算尚未完成，get方法将会阻塞。一旦计算完成，计算就不能重新启动或取消(除非使用runAndReset方法调用计算)。

Future实现了RunnableFuture接口，而RunnableFuture接口继承了Runnable与Future接口，所以它既可以作为Runnable被线程中执行，又可以作为callable获得返回值。

```java
public class FutureTask<V> implements RunnableFuture<V> {
    ...
}

public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
```

#### Runnable与Callable对比

​	通常实现一个线程我们会使用继承Thread的方式或者实现Runnable接口，**这两种方式有一个共同的缺陷就是在执行完任务之后无法获取执行结果。**从Java1.5之后就提供了Callable与Future，这两个接口就可以实现获取任务执行结果。

- Runnable接口：代码非常简单，只有一个方法run

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
```

- Callable泛型接口：有泛型参数，提供了一个call方法，**执行后可返回传入的泛型参数类型的结果**。

```java
public interface Callable<V> {
    V call() throws Exception;
}
```

FutureTask支持两种参数类型，Callable和Runnable，在使用Runnable 时，还可以多指定一个返回结果类型。

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}

public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}
```

### 2.Future接口

Future接口提供了一系列方法用于控制线程执行计算，如下：

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);//取消任务
    boolean isCancelled();//是否被取消
    boolean isDone();//计算是否完成
    V get() throws InterruptedException, ExecutionException;//获取计算结果，在执行过程中任务被阻塞
    V get(long timeout, TimeUnit unit)//timeout等待时间、unit时间单位
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

### 3.ForkJoin

​	ForkJoin是Java7提供的一个并行执行任务的框架，是**把大任务分割成若干个小任务，待小任务完成后将结果汇总成大任务结果的框架。**主要采用的是==**工作窃取算法**==，工作窃取算法是指某个线程从其他队列里窃取任务来执行。 ![è¿éåå¾çæè¿°](/Users/jack/Desktop/md/images/70-20190209123220985.png)

​	在窃取过程中两个线程会**访问同一个队列**，**为了减少窃取任务线程和被窃取任务线程之间的竞争，通常我们会使用双端队列来实现工作窃取算法。**被窃取任务的线程永远从队列的头部拿取任务，窃取任务的线程从队列尾部拿取任务。

局限性：
1、==任务只能使用fork和join作为同步机制==，如果使用了其他同步机制，当他们在同步操作时，工作线程就不能执行其他任务了。比如在fork框架使任务进入了睡眠，那么在睡眠期间内在执行这个任务的线程将不会执行其他任务了。 
2、我们所拆分的任务不应该去执行IO操作，如读和写数据文件。 
3、任务不能抛出检查异常。必须通过必要的代码来处理他们。

框架核心：
核心有两个类：ForkJoinPool和ForkJoinTask 

ForkJoinPool：负责来做实现，包括工作窃取算法、管理工作线程和提供关于任务的状态以及他们的执行信息。 

ForkJoinTask：提供在任务中执行fork和join的机制。

### 4.BlockingQueue阻塞队列

主要应用场景：生产者消费者模型，是**线程安全**的 

![è¿éåå¾çæè¿°](/Users/jack/Desktop/md/images/70-20190209123252790.png)

#### 阻塞情况：

1、当队列满了进行入队操作 
2、当队列空了的时候进行出队列操作

#### 四套方法：

BlockingQueue提供了四套方法，分别来进行插入、移除、检查。每套方法在不能立刻执行时都有不同的反应。 
![这里写图片描述](/Users/jack/Desktop/md/images/70-20190209123308024.png)

- Throws Exceptions ：如果不能立即执行就抛出异常。
- Special Value：如果不能立即执行就返回一个特殊的值，一般是true/false。
- Blocks：如果不能立即执行就阻塞
- Times Out：如果不能立即执行就阻塞一段时间，如果过了设定时间还没有被执行，则返回一个值

![image-20190210161730391](/Users/jack/Desktop/md/images/image-20190210161730391.png)

#### 实现类：

ArrayBlockingQueue：它是一个有界的阻塞队列，内部实现是数组，初始化时指定容量大小，一旦指定大小就不能再变。采用FIFO方式存储元素。

DelayQueue：阻塞内部元素，内部元素必须实现Delayed接口，**Delayed接口又继承了Comparable接口，原因在于DelayQueue内部元素需要排序，一般情况按过期时间优先级排序。**要保证元素类型具有可比性，所以要继承Comparable接口。

```
public interface Delayed extends Comparable<Delayed> {
    long getDelay(TimeUnit unit);
}
```

==DalayQueue内部采用PriorityQueue与ReentrantLock实现。==PriorityQueue优先队列，基于小顶堆实现的。

```java
public class DelayQueue<E extends Delayed> extends AbstractQueue<E>
    implements BlockingQueue<E> {

    private final transient ReentrantLock lock = new ReentrantLock();
    private final PriorityQueue<E> q = new PriorityQueue<E>();
    ...
}
```

LinkedBlockingQueue：大小配置可选，如果初始化时指定了大小，那么它就是有边界的。不指定就无边界（最大整型值）。内部实现是链表，采用FIFO形式保存数据。

```
public LinkedBlockingQueue() {
    this(Integer.MAX_VALUE);//不指定大小，无边界采用默认值，最大整型值
}
```

PriorityBlockingQueue:带优先级的阻塞队列。无边界队列，允许插入null。**插入的对象必须实现Comparator接口，队列优先级的排序规则就是按照我们对Comparable接口的实现来指定的。**我们可以从PriorityBlockingQueue中获取一个迭代器，但这个迭代器并不保证能按照优先级的顺序进行迭代。

```java
public boolean add(E e) {//添加方法
    return offer(e);
}
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    final ReentrantLock lock = this.lock;
    lock.lock();
    int n, cap;
    Object[] array;
    while ((n = size) >= (cap = (array = queue).length))
        tryGrow(array, cap);
    try {
        Comparator<? super E> cmp = comparator;//必须实现Comparator接口
        if (cmp == null)
            siftUpComparable(n, e, array);
        else
            siftUpUsingComparator(n, e, array, cmp);
        size = n + 1;
        notEmpty.signal();
    } finally {
        lock.unlock();
    }
    return true;
}
```

SynchronusQueue：只能插入一个元素，同步队列，无界非缓存队列，不存储元素。

# 8.线程池 Executor

## 1.线程池的好处

#### new Thread的弊端

- 每次new Thread 新建对象，性能差
- 线程缺乏统一管理，可能无限制的新建线程，相互竞争，可能占用过多的系统资源导致死机或者OOM（out of memory 内存溢出），这种问题的原因不是因为单纯的new一个Thread，而是可能因为程序的bug或者设计上的缺陷导致不断new Thread造成的。
- 缺少更多功能，如更多执行、定期执行、线程中断。

#### 线程池的好处

- 重用存在的线程，减少对象创建、消亡的开销，性能好
- 可有效控制最大并发线程数，提高系统资源利用率，同时可以避免过多资源竞争，避免阻塞。
- 提供定时执行、定期执行、单线程、并发数控制等功能。

## 2.线程池相关类

![è¿éåå¾çæè¿°](/Users/jack/Desktop/md/images/70-20190210183214452.png)

​	在线程池的类图中，我们最常使用的是最下边的Executors,用它来创建线程池使用线程。那么在上边的类图中，包含了一个Executor框架，它是一个根据一组执行策略的调用调度执行和控制异步任务的框架，目的是提供一种将任务提交与任务如何运行分离开的机制。它包含了三个executor接口：

- Executor:运行新任务的简单接口
- ExecutorService：扩展了Executor，添加了用来管理执行器生命周期和任务生命周期的方法
- ScheduleExcutorService：扩展了ExecutorService，支持Future和定期执行任务

### 2.1 线程池核心类-ThreadPoolExecutor

参数说明：ThreadPoolExecutor一共有七个参数，这七个参数配合起来，构成了线程池强大的功能。

corePoolSize：核心线程数量
maximumPoolSize：线程最大线程数

workQueue：阻塞队列，存储等待执行的任务，很重要，会对线程池运行过程产生重大影响

​	当我们提交一个新的任务到线程池，线程池会根据当前池中正在运行的线程数量来决定该任务的处理方式。处理方式有三种： 
1、直接切换（SynchronusQueue） 
2、无界队列（LinkedBlockingQueue）能够创建的最大线程数为corePoolSize,这时maximumPoolSize就不会起作用了。当线程池中所有的核心线程都是运行状态的时候，新的任务提交就会放入等待队列中。 
3、有界队列（ArrayBlockingQueue）最大maximumPoolSize，能够降低资源消耗，但是这种方式使得线程池对线程调度变的更困难。因为线程池与队列容量都是有限的。所以想让线程池的吞吐率和处理任务达到一个合理的范围，又想使我们的线程调度相对简单，并且还尽可能降低资源的消耗，我们就需要合理的限制这两个数量 
**分配技巧：** [如果想降低资源的消耗包括降低cpu使用率、操作系统资源的消耗、上下文切换的开销等等，可以设置一个较大的队列容量和较小的线程池容量，这样会降低线程池的吞吐量。如果我们提交的任务经常发生阻塞，我们可以调整maximumPoolSize。如果我们的队列容量较小，我们需要把线程池大小设置的大一些，这样cpu的使用率相对来说会高一些。但是如果线程池的容量设置的过大，提高任务的数量过多的时候，并发量会增加，那么线程之间的调度就是一个需要考虑的问题。这样反而可能会降低处理任务的吞吐量。]

- keepAliveTime：线程没有任务执行时最多保持多久时间终止（当线程中的线程数量大于corePoolSize的时候，如果这时没有新的任务提交核心线程外的线程不会立即销毁，而是等待，直到超过keepAliveTime）
- unit：keepAliveTime的时间单位
- threadFactory：线程工厂，用来创建线程，有一个默认的工场来创建线程，这样新创建出来的线程有相同的优先级，是非守护线程、设置好了名称）
- rejectHandler：当拒绝处理任务时(阻塞队列满)的策略（AbortPolicy默认策略直接抛出异常、CallerRunsPolicy用调用者所在的线程执行任务、DiscardOldestPolicy丢弃队列中最靠前的任务并执行当前任务、DiscardPolicy直接丢弃当前任务） 

![è¿éåå¾çæè¿°](/Users/jack/Desktop/md/images/70-20190210183509364.png)

#### corePoolSize、maximumPoolSize、workQueue 三者关系：

​	如果运行的线程数小于corePoolSize的时候，直接创建新线程来处理任务。即使线程池中的其他线程是空闲的。如果运行中的线程数大于corePoolSize且小于maximumPoolSize时，那么只有当workQueue满的时候才创建新的线程去处理任务。如果corePoolSize与maximumPoolSize是相同的，那么创建的线程池大小是固定的。这时有新任务提交，当workQueue未满时，就把请求放入workQueue中。等待空线程从workQueue取出任务。如果workQueue此时也满了，那么就使用另外的拒绝策略参数去执行拒绝策略。

初始化方法：由七个参数组合成四个初始化方法 
![这里写图片描述](/Users/jack/Desktop/md/images/70-20190210203627559.png)

其他方法：

![image-20190210203736234](/Users/jack/Desktop/md/images/image-20190210203736234.png)

线程池生命周期： 
![这里写图片描述](/Users/jack/Desktop/md/images/70-20190210203744638.png)

- running：能接受新提交的任务，也能处理阻塞队列中的任务
- shutdown：不能处理新的任务，但是能继续处理阻塞队列中任务
- stop：不能接收新的任务，也不处理队列中的任务
- tidying：如果所有的任务都已经终止了，这时有效线程数为0
- terminated：最终状态

### 2.2 使用Executor创建线程池

**使用Executor可以创建四种线程池：分别对应上边提到的四种线程池初始化方法**

#### 1、Executors.newCachedThreadPool 

创建一个可缓存的线程池，如果线程池的长度超过了处理的需要，可以灵活回收空闲线程。如果没有可回收的就新建线程。

```java
//源码：
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}

//使用方法：
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 10; i++) {
        final int index = i;
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                log.info("task:{}", index);
            }
        });
    }
    executorService.shutdown();
}
```

值得注意的一点是，newCachedThreadPool的返回值是ExecutorService类型，该类型只包含基础的线程池方法，但却不包含线程监控相关方法，因此在使用返回值为ExecutorService的线程池类型创建新线程时要考虑到具体情况。

 ![è¿éåå¾çæè¿°](/Users/jack/Desktop/md/images/70-20190210204025898.png)

#### 2、newFixedThreadPool 

定长线程池，可以线程现成的最大并发数，超出在队列等待

```java
//源码：
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}

//使用方法：
public static void main(String[] args) {
    ExecutorService executorService = Executors.newFixedThreadPool(3);
    for (int i = 0; i < 10; i++) {
        final int index = i;
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                log.info("task:{}", index);
            }
        });
    }
    executorService.shutdown();
}
```

#### 3、newSingleThreadExecutor 

单线程化的线程池，用唯一的一个共用线程执行任务，保证所有任务按指定顺序执行（FIFO、优先级…）

```java
//源码
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}

//使用方法：
public static void main(String[] args) {
    ExecutorService executorService = Executors.newSingleThreadExecutor();
    for (int i = 0; i < 10; i++) {
        final int index = i;
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                log.info("task:{}", index);
            }
        });
    }
    executorService.shutdown();
}
```

#### 4、newScheduledThreadPool 

定长线程池，支持定时和周期任务执行

```java
//源码：
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,//此处super指的是ThreadPoolExecutor
          new DelayedWorkQueue());
}

//基础使用方法：
public static void main(String[] args) {
    ScheduledExecutorService executorService = Executors.newScheduledThreadPool(1);
    executorService.schedule(new Runnable() {
        @Override
        public void run() {
            log.warn("schedule run");
        }
    }, 3, TimeUnit.SECONDS);//延迟3秒执行
    executorService.shutdown();
}
```

ScheduledExecutorService提供了三种方法可以使用： 
![这里写图片描述](/Users/jack/Desktop/md/images/70-20190210204543644.png) 
scheduleAtFixedRate：以指定的速率执行任务 
scheduleWithFixedDelay：以指定的延迟执行任务 

比如：

```java
executorService.scheduleAtFixedRate(new Runnable() {
    @Override
    public void run() {
        log.warn("schedule run");
    }
}, 1, 3, TimeUnit.SECONDS);//延迟一秒后每隔3秒执行
```

小扩展：延迟执行任务的操作，java中还有Timer类同样可以实现

```java
Timer timer = new Timer();
timer.schedule(new TimerTask() {
    @Override
    public void run() {
        log.warn("timer run");
    }
}, new Date(), 5 * 1000);
```











































































































































































































































































































































































参照：https://blog.csdn.net/jesonjoke/column/info/21011

[慕课网](https://www.baidu.com/s?wd=%E6%85%95%E8%AF%BE%E7%BD%91&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)jimin老师的[《Java并发编程与高并发解决方案》](https://www.baidu.com/s?wd=%E3%80%8AJava%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E4%B8%8E%E9%AB%98%E5%B9%B6%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E3%80%8B&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)课程

