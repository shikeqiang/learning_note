# java内存模型

# 1 Java内存模型的基础

## 1.1 并发编程模型的两个关键问题

​	在并发编程中，需要处理两个关键问题：**线程之间如何通信及线程之间如何同步（这里的线程是指并发执行的活动实体）。通信是指线程之间以何种机制来交换信息。**

​	在命令式编程中，**线程之间的通信机制有两种：共享内存和消息传递。**在共享内存的并发模型里，线程之间共享程序的公共状态，通过写-读内存中的公共状态进行隐式通信。

​	在消息传递的并发模型里，线程之间没有公共状态，线程之间必须通过发送消息来显式进行通信。**同步是指程序中用于控制不同线程间操作发生相对顺序的机制。在共享内存并发模型里，同步是显式进行的。**程序员必须显式指定某个方法或某段代码需要在线程之间互斥执行。

​	**在消息传递的并发模型里，由于消息的发送必须在消息的接收之前，因此同步是隐式进行的。**Java的并发采用的是共享内存模型，Java线程之间的通信总是隐式进行，整个通信过程对程序员完全透明。如果编写多线程程序的Java程序员不理解隐式进行的线程之间通信的工作机制，很可能会遇到各种奇怪的内存可见性问题。

## 1.2 Java内存模型的抽象结构

​	==在Java中，所有实例域、静态域和数组元素都存储在堆内存中，堆内存在线程之间共享==（“共享变量”这个术语代指实例域，静态域和数组元素）。局部变量（Local Variables），方法定义参数（Java语言规范称之为Formal Method Parameters）和异常处理器参数（ExceptionHandler Parameters）不会在线程之间共享，它们不会有内存可见性问题，也不受内存模型的影响。

​	Java线程之间的通信由Java内存模型（本文简称为JMM）控制，**JMM决定一个线程对共享变量的写入何时对另一个线程可见。**从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（Main Memory）中，每个线程都有一个私有的本地内存（Local Memory），本地内存中存储了该线程以读/写共享变量的副本。**本地内存是JMM的一个抽象概念，并不真实存在。它涵盖了缓存、写缓冲区、寄存器。**

![image-20190121173010844](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190121173010844.png)

从上图可见，如果线程A与线程B之间要通信的话，必须要经历下面2个步骤。

1）线程A把本地内存A中更新过的共享变量刷新到主内存中去。

2）线程B到主内存中去读取线程A之前已更新过的共享变量。

下图是这两个步骤的实现过程：

![image-20190121173102261](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190121173102261.png)

​	如上图所示，**本地内存A和本地内存B有主内存中共享变量x的副本。**假设初始时，这3个内存中的x值都为0。线程A在执行时，把更新后的x值（假设值为1）临时存放在自己的本地内存A中。当线程A和线程B需要通信时，线程A首先会把自己本地内存中修改后的x值刷新到主内存中，此时主内存中的x值变为了1。随后，线程B到主内存中去读取线程A更新后的x值，此时线程B的本地内存的x值也变为了1。

​	从整体来看，**这两个步骤实质上是线程A在向线程B发送消息，而且这个通信过程必须要经过主内存。JMM通过控制主内存与每个线程的本地内存之间的交互，来为Java程序员提供内存可见性保证。**

## 1.3 从源代码到指令序列的重排序

##### ​	在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排序。重排序分3种类型。

1）编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。

2）指令级并行的重排序。现代处理器采用了指令级并行技术（Instruction-LevelParallelism，ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。

3）内存系统的重排序。由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

从Java源代码到最终实际执行的指令序列，会分别经历下面3种重排序，如图：

![image-20190121173327935](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190121173327935.png)

​	上述的**1属于编译器重排序，2和3属于处理器重排序。**这些重排序可能会导致多线程程序出现内存可见性问题。==对于编译器，JMM的编译器重排序规则会禁止特定类型的编译器重排序==（不是所有的编译器重排序都要禁止）。==对于处理器重排序，JMM的处理器重排序规则会要求Java编译器在生成指令序列时，插入特定类型的内存屏障（Memory Barriers，Intel称之为Memory Fence）指令，通过内存屏障指令来禁止特定类型的处理器重排序。==

​	JMM属于语言级的内存模型，它确保在不同的编译器和不同的处理器平台之上，通过禁止特定类型的编译器重排序和处理器重排序，为程序员提供一致的内存可见性保证。

## 1.4 并发编程模型的分类

​	**现代的处理器使用==写缓冲区==临时保存向内存写入的数据。**写缓冲区可以保证指令流水线持续运行，它可以避免由于处理器停顿下来等待向内存写入数据而产生的延迟。同时，通过以批处理的方式刷新写缓冲区，以及合并写缓冲区中对同一内存地址的多次写，减少对内存总线的占用。**虽然写缓冲区有这么多好处，但每个处理器上的写缓冲区，仅仅对它所在的处理器可见。**这个特性会对内存操作的执行顺序产生重要的影响：==处理器对内存的读/写操作的执行顺序，不一定与内存实际发生的读/写操作顺序一致！==

​								**处理器操作内存的执行结果如下表：**

![image-20190121173522842](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190121173522842.png)

​	假设处理器A和处理器B按程序的顺序并行执行内存访问，最终可能得到x=y=0的结果。具体的原因如图：![image-20190121173604121](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190121173604121.png)

​	这里处理器A和处理器B可以同时把共享变量写入自己的写缓冲区（A1，B1），然后从内存中读取另一个共享变量（A2，B2），最后才把自己写缓存区中保存的脏数据刷新到内存中（A3，B3）。当以这种时序执行时，程序就可以得到x=y=0的结果。

​	**从内存操作实际发生的顺序来看，直到处理器A执行A3来刷新自己的写缓存区，写操作A1才算真正执行了。**虽然处理器A执行内存操作的顺序为：A1→A2，但内存操作实际发生的顺序却是A2→A1。此时，处理器A的内存操作顺序被重排序了。这里的关键是，**由于写缓冲区仅对自己的处理器可见，它会导致处理器执行内存操作的顺序可能会与内存实际的操作执行顺序不一致。**由于现代的处理器都会使用写缓冲区，因此现代的处理器都会允许对写-读操作进行重排序。

​							**常见处理器允许的重排序类型的列表，处理器的重排序规则**

![image-20190121173807026](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190121173807026.png)

​	单元格中的“N”表示处理器不允许两个操作重排序，“Y”表示允许重排序。从上表可以看出：**常见的处理器都允许Store-Load重排序；**常见的处理器都不允许对存在数据依赖的操作做重排序。sparc-TSO和X86拥有相对较强的处理器内存模型，它们仅允许对写-读操作做重排序（因为它们都使用了写缓冲区）。

​	**为了保证内存可见性，Java编译器在生成指令序列的适当位置会插入内存屏障指令来禁止特定类型的处理器重排序。JMM把内存屏障指令分为4类**，如下表：

![image-20190121173924601](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190121173924601.png)

​	StoreLoad Barriers是一个“全能型”的屏障，它同时具有其他3个屏障的效果。现代的多处理器大多支持该屏障（其他类型的屏障不一定被所有处理器支持）。执行该屏障开销会很昂贵，因为当前处理器通常要把写缓冲区中的数据全部刷新到内存中（Buffer Fully Flush）。

## 1.5 happens-before简介

​	在JMM中，如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before关系。**这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。**与程序员密切相关的happens-before规则如下：

- 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
- 监视器锁规则：对一个锁的解锁，happens-before于**随后**对这个锁的加锁。
- volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
- 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。

==注意==：两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行！happens-before仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前（the first is visible to and ordered before the second）。

happens-before与JMM的关系如图：![image-20190121174144893](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190121174144893.png)

​	如图，一个happens-before规则对应于一个或多个编译器和处理器重排序规则。对于Java程序员来说，happens-before规则简单易懂，它避免Java程序员为了理解JMM提供的内存可见性保证而去学习复杂的重排序规则以及这些规则的具体实现方法。

# 2 重排序

​	重排序是指编译器和处理器为了优化程序性能而对指令序列进行重新排序的一种手段。

## 2.1 数据依赖性	

​	如果两个操作访问同一个变量，且这两个操作中有一个为写操作，此时这两个操作之间就存在数据依赖性。数据依赖分为下列3种类型，如表：

![image-20190121180927609](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190121180927609.png)

​	上面3种情况，**只要重排序两个操作的执行顺序，程序的执行结果就会被改变。**编译器和处理器可能会对操作做重排序。**编译器和处理器在重排序时，会遵守数据依赖性，编译器和处理器不会改变存在数据依赖关系的两个操作的执行顺序。**

​	这里所说的数据依赖性仅针对单个处理器中执行的指令序列和单个线程中执行的操作，不同处理器之间和不同线程之间的数据依赖性不被编译器和处理器考虑。

## 2.2 as-if-serial语义

​	**as-if-serial语义的意思是：不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。**编译器、runtime和处理器都必须遵守as-if-serial语义。

​	**为了遵守as-if-serial语义，编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果。**但是，如果操作之间不存在数据依赖关系，这些操作就可能被编译器和处理器重排序。

​	as-if-serial语义把单线程程序保护了起来，遵守as-if-serial语义的编译器、runtime和处理器共同为编写单线程程序的程序员创建了一个幻觉：单线程程序是按程序的顺序来执行的。asif-serial语义使单线程程序员无需担心重排序会干扰他们，也无需担心内存可见性问题。

## 2.3 程序顺序规则

​	假设有这三种程序顺序规则：1）A happens-before B。2）B happens-before C。3）A happens-before C。

​	这里的第3个happens-before关系，是根据happens-before的传递性推导出来的。这里A happens-before B，但实际执行时B却可以排在A之前执行（看上面的重排序后的执行顺序）。**如果A happens-before B，JMM并不要求A一定要在B之前执行。JMM仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前。**这里操作A的执行结果不需要对操作B可见；而且重排序操作A和操作B后的执行结果，与操作A和操作B按happens-before顺序执行的结果一致。在这种情况下，JMM会认为这种重排序并不非法（notillegal），JMM允许这种重排序。

## 2.4 重排序对多线程的影响	

​	看下面这个例子：

```Java
class ReorderExample {
	int a = 0;
	boolean flag = false;
	public void writer() {
		a = 1; // 1
		flag = true; // 2
	}
	Public void reader() {
		if (flag) { // 3
			int i = a * a; // 4
			……
			}
		}
	}
```

​	flag变量是个标记，用来标识变量a是否已被写入。这里假设有两个线程A和B，A首先执行writer()方法，随后B线程接着执行reader()方法。线程B在执行操作4时，不一定能看到线程A在操作1对共享变量a的写入。

​	由于操作1和操作2没有数据依赖关系，编译器和处理器可以对这两个操作重排序；同样，操作3和操作4没有数据依赖关系，编译器和处理器也可以对这两个操作重排序。当操作1和操作2重排序时，执行顺序可能如下图：

![image-20190121181723386](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190121181723386.png)

​	如上图所示，操作1和操作2做了重排序。程序执行时，线程A首先写标记变量flag，随后线程B读这个变量。由于条件判断为真，线程B将读取变量a。此时，变量a还没有被线程A写入，在这里多线程程序的语义被重排序破坏了！虚箭线标识错误的读操作，用实箭线标识正确的读操作。

​	当操作3和操作4重排序时，执行顺序如下图：![image-20190121182042140](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190121182042140.png)

​	在程序中，操作3和操作4存在控制依赖关系。当代码中存在控制依赖性时，会影响指令序列执行的并行度。为此，**编译器和处理器会采用猜测（Speculation）执行来克服控制相关性对并行度的影响。**以处理器的猜测执行为例，执行线程B的处理器可以提前读取并计算a*a，然后把计算结果临时保存到一个名为重排序缓冲（Reorder Buffer，ROB）的硬件缓存中。当操作3的条件判断为真时，就把该计算结果写入变量i中。

​	从上图可以看出，猜测执行实质上对操作3和4做了重排序。重排序在这里破坏了多线程程序的语义！在单线程程序中，对存在控制依赖的操作重排序，不会改变执行结果（这也是as-if-serial语义允许对存在控制依赖的操作做重排序的原因）；但在多线程程序中，对存在控制依赖的操作重排序，可能会改变程序的执行结果。

# 3 顺序一致性

​	顺序一致性内存模型是一个理论参考模型，在设计的时候，处理器的内存模型和编程语言的内存模型都会以顺序一致性内存模型作为参照。	

## 3.1 数据竞争与顺序一致性

​	当程序未正确同步时，就可能会存在数据竞争。Java内存模型规范对数据竞争的定义如下：**在一个线程中写一个变量，在另一个线程读同一个变量，而且写和读没有通过同步来排序。**

​	当代码中包含数据竞争时，程序的执行往往产生违反直觉的结果。如果一个多线程程序能正确同步，这个程序将是一个没有数据竞争的程序。**JMM对正确同步的多线程程序的内存一致性做了如下保证**：如果程序是正确同步的，程序的执行将具有顺序一致性（Sequentially Consistent）——即程序的执行结果与该程序在顺序一致性内存模型中的执行结果相同。**这里的同步是指广义上的同步，包括对常用同步原语（synchronized、volatile和final）的正确使用。**

## 3.2 顺序一致性内存模型

​	顺序一致性内存模型是一个被计算机科学家理想化了的理论参考模型，它为程序员提供了极强的内存可见性保证。顺序一致性内存模型有两大特性：

1）一个线程中的所有操作必须按照程序的顺序来执行。

2）（不管程序是否同步）所有线程都只能看到一个单一的操作执行顺序。**在顺序一致性内存模型中，每个操作都必须原子执行且立刻对所有线程可见。**

顺序一致性内存模型为程序员提供的视图如图：

![image-20190122152028044](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190122152028044.png)

​	在概念上，**顺序一致性模型有一个单一的全局内存**，这个内存通过一个左右摆动的开关可以连接到任意一个线程，同时每一个线程必须按照程序的顺序来执行内存读/写操作。从上面的示意图可以看出，在任意时间点最多只能有一个线程可以连接到内存。当多个线程并发执行时，图中的开关装置能把所有线程的所有内存读/写操作==串行化（即在顺序一致性模型中，所有操作之间具有全序关系）。==

​	**未同步程序在顺序一致性模型中虽然整体执行顺序是无序的，但所有线程都只能看到一个一致的整体执行顺序，因为顺序一致性内存模型中的每个操作必须立即对任意线程可见。**

​	但是，在JMM中就没有这个保证。**未同步程序在JMM中不但整体的执行顺序是无序的，而且所有线程看到的操作执行顺序也可能不一致。**比如，在当前线程把写过的数据缓存在本地内存中，在没有刷新到主内存之前，这个写操作仅对当前线程可见；从其他线程的角度来观察，会认为这个写操作根本没有被当前线程执行。**只有当前线程把本地内存中写过的数据刷新到主内存之后，这个写操作才能对其他线程可见。**在这种情况下，当前线程和其他线程看到的操作执行顺序将不一致。

## 3.3 同步程序的顺序一致性效果

​	**顺序一致性模型中，所有操作完全按程序的顺序串行执行。**==而在JMM中，临界区内的代码可以重排序（但JMM不允许临界区内的代码“逸出”到临界区之外，那样会破坏监视器的语义）==。**JMM会在退出临界区和进入临界区这两个关键时间点做一些特别处理，使得线程在这两个时间点具有与顺序一致性模型相同的内存视图**。

​	JMM在具体实现上的基本方针为：在不改变（正确同步的）程序执行结果的前提下，尽可能地为编译器和处理器的优化打开方便之门。

## 3.4 未同步程序的执行特性

​	对于未同步或未正确同步的多线程程序，JMM只提供最小安全性：线程执行时读取到的值，要么是之前某个线程写入的值，要么是默认值（0，Null，False），JMM保证线程读操作读取到的值不会无中生有（Out Of Thin Air）的冒出来。为了实现最小安全性，JVM在堆上分配对象时，首先会对内存空间进行清零，然后才会在上面分配对象（JVM内部会同步这两个操作）。因此，在已清零的内存空间（Pre-zeroed Memory）分配对象时，域的默认初始化已经完成了。

​	**JMM不保证未同步程序的执行结果与该程序在顺序一致性模型中的执行结果一致。**因为如果想要保证执行结果一致，JMM需要禁止大量的处理器和编译器的优化，这对程序的执行性能会产生很大的影响。而且未同步程序在顺序一致性模型中执行时，整体是无序的，其执行结果往往无法预知。而且，保证未同步程序在这两个模型中的执行结果一致没什么意义。

​	未同步程序在JMM中的执行时，整体上是无序的，其执行结果无法预知。未同步程序在两个模型中的执行特性有如下几个差异：

​	1）顺序一致性模型保证单线程内的操作会按程序的顺序执行，而JMM不保证单线程内的操作会按程序的顺序执行（比如上面正确同步的多线程程序在临界区内的重排序）。

​	2）顺序一致性模型保证所有线程只能看到一致的操作执行顺序，而JMM不保证所有线程能看到一致的操作执行顺序。

​	3）JMM不保证对64位的long型和double型变量的写操作具有原子性，而顺序一致性模型保证对所有的内存读/写操作都具有原子性。

​	第3个差异与处理器总线的工作机制密切相关。在计算机中，数据通过总线在处理器和内存之间传递。每次处理器和内存之间的数据传递都是通过一系列步骤来完成的，**这一系列步骤称之为总线事务（Bus Transaction）。总线事务包括读事务（Read Transaction）和写事务（WriteTransaction）。读事务从内存传送数据到处理器，写事务从处理器传送数据到内存，每个事务会读/写内存中一个或多个物理上连续的字。**这里的关键是，==总线会同步试图并发使用总线的事务。在一个处理器执行总线事务期间，总线会禁止其他的处理器和I/O设备执行内存的读/写。==总线的工作机制示意图：![image-20190122152758632](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190122152758632.png)

​	由图可知，假设处理器A，B和C同时向总线发起总线事务，这时总线仲裁（Bus Arbitration）会对竞争做出裁决，这里假设总线在仲裁后判定处理器A在竞争中获胜（总线仲裁会确保所有处理器都能公平的访问内存）。此时处理器A继续它的总线事务，而其他两个处理器则要等待处理器A的总线事务完成后才能再次执行内存访问。假设在处理器A执行总线事务期间（不管这个总线事务是读事务还是写事务），处理器D向总线发起了总线事务，此时处理器D的请求会被总线禁止。**总线的这些工作机制可以把所有处理器对内存的访问以串行化的方式来执行。**在任意时间点，最多只能有一个处理器可以访问内存。==这个特性确保了单个总线事务之中的内存读/写操作具有原子性。==

​	在一些32位的处理器上，如果要求对64位数据的写操作具有原子性，会有比较大的开销。为了照顾这种处理器，Java语言规范鼓励但不强求JVM对64位的long型变量和double型变量的写操作具有原子性。当JVM在这种处理器上运行时，可能会把一个64位long/double型变量的写操作拆分为两个32位的写操作来执行。这两个32位的写操作可能会被分配到不同的总线事务中执行，此时对这个64位变量的写操作将不具有原子性。当单个内存操作不具有原子性时，可能会产生意想不到后果。示意图:![image-20190122152931537](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190122152931537.png)

​	如上图所示，假设处理器A写一个long型变量，同时处理器B要读这个long型变量。**处理器A中64位的写操作被拆分为两个32位的写操作，且这两个32位的写操作被分配到不同的写事务中执行。**同时，处理器B中64位的读操作被分配到单个的读事务中执行。当处理器A和B按上图的时序来执行时，处理器B将看到仅仅被处理器A“写了一半”的无效值。

​	注意，在JSR-133之前的旧内存模型中，一个64位long/double型变量的读/写操作可以被拆分为两个32位的读/写操作来执行。**从JSR-133内存模型开始（即从JDK5开始），仅仅只允许把一个64位long/double型变量的写操作拆分为两个32位的写操作来执行，任意的读操作在JSR-133中都必须具有原子性（即任意读操作必须要在单个读事务中执行）。**

# 4 volatile的内存语义

## 4.1 volatile的特性

​	可以把对volatile变量的单个读/写，看成是使用同一个锁对这些单个读/写操作做了同步。如下面的例子：

```Java
class VolatileFeaturesExample {
	volatile long vl = 0L; // 使用volatile声明64位的long型变量
	public void set(long l) {
		vl = l; // 单个volatile变量的写
	}
	public void getAndIncrement () {
		vl++; // 复合（多个）volatile变量的读/写
	}
	public long get() {
		return vl; // 单个volatile变量的读
		}
	}
```

假设有多个线程分别调用上面程序的3个方法，这个程序在语义上和下面程序等价。

```Java
class VolatileFeaturesExample {
	long vl = 0L; // 64位的long型普通变量
	public synchronized void set(long l) { // 对单个的普通变量的写用同一个锁同步
		vl = l;
	}
	public void getAndIncrement () { // 普通方法调用
		long temp = get(); // 调用已同步的读方法
		temp += 1L; // 普通写操作
		set(temp); // 调用已同步的写方法
	}
	public synchronized long get() { // 对单个的普通变量的读用同一个锁同步
		return vl;
	}
}
```

​	其实就是对volatile变量的读写方法进行加锁同步。

​	锁的happens-before规则保证释放锁和获取锁的两个线程之间的内存可见性，这意味着对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。

​	锁的语义决定了临界区代码的执行具有原子性。**这意味着，即使是64位的long型和double型变量，只要它是volatile变量，对该变量的读/写就具有原子性。如果是多个volatile操作或类似于volatile++这种复合操作，这些操作整体上不具有原子性。**

​	简而言之，volatile变量自身具有下列特性：

- 可见性：对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。
- 原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性。

## 4.2 volatile写-读建立的happens-before关系

​	从JSR-133开始（即从JDK5开始），volatile变量的写-读可以实现线程之间的通信。**==从内存语义的角度来说，volatile的写-读与锁的释放-获取有相同的内存效果：volatile写和锁的释放有相同的内存语义；volatile读与锁的获取有相同的内存语义。==**

## 4.3 volatile写-读的内存语义

​	**volatile写的内存语义：当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存。**

​	**volatile读的内存语义：当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效。线程接下来将从主内存中读取共享变量。**

### 对volatile写和volatile读的内存语义总结：

- 线程A写一个volatile变量，实质上是线程A向接下来将要读这个volatile变量的某个线程发出了（其对共享变量所做修改的）消息。
- 线程B读一个volatile变量，实质上是线程B接收了之前某个线程发出的（在写这个volatile变量之前对共享变量所做修改的）消息。
- 线程A写一个volatile变量，随后线程B读这个volatile变量，这个过程实质上是**线程A通过主内存向线程B发送消息。**

## 4.4 volatile内存语义的实现

​	下面来看看JMM如何实现volatile写/读的内存语义。**重排序分为编译器重排序和处理器重排序。为了实现volatile内存语义，JMM会分别限制这两种类型的重排序类型。**下表是JMM针对编译器制定的volatile重排序规则表。

![image-20190122165208807](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190122165208807.png)

​	第三行最后一个单元格的意思是：在程序中，当第一个操作为普通变量的读或写时，如果第二个操作为volatile写，则编译器不能重排序这两个操作。

从上表可以看出：

- 当第二个操作是volatile写时，不管第一个操作是什么，都不能重排序。**这个规则确保volatile写之前的操作不会被编译器重排序到volatile写之后。**
- 当第一个操作是volatile读时，不管第二个操作是什么，都不能重排序。**这个规则确保volatile读之后的操作不会被编译器重排序到volatile读之前。**
- 当第一个操作是volatile写，第二个操作是volatile读时，不能重排序。

​	为了实现volatile的内存语义，**编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。**对于编译器来说，发现一个最优布置来最小化插入屏障的总数几乎不可能。为此，JMM采取保守策略。下面是基于保守策略的JMM内存屏障插入策略。

- 在每个volatile写操作的前面插入一个StoreStore屏障。
- 在每个volatile写操作的后面插入一个StoreLoad屏障。
- 在每个volatile读操作的后面插入一个LoadLoad屏障。
- 在每个volatile读操作的后面插入一个LoadStore屏障。

​	这个要对应上面的屏障指令去看，比如第一个的话，就相当于：StoreStore这个屏障在volatile写操作前面，所以在volatile写之前的数据都会对其他处理器可见(即刷新到主内存)。

​	上述内存屏障插入策略非常保守，但它可以保证在任意处理器平台，任意的程序中都能得到正确的volatile内存语义。下面是保守策略下，volatile写插入内存屏障后生成的指令序列示意图，如图：![image-20190122165826574](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190122165826574.png)

​	上图StoreStore屏障可以保证在volatile写之前，其前面的所有普通写操作已经对任意处理器可见了。这是因为StoreStore屏障将保障上面所有的普通写在volatile写之前刷新到主内存。

​	**volatile写后面的StoreLoad屏障的作用是避免volatile写与后面可能有的volatile读/写操作重排序。**因为编译器常常无法准确判断在一个volatile写的后面是否需要插入一个StoreLoad屏障（比如，一个volatile写之后方法立即return）。为了保证能正确实现volatile的内存语义，**==JMM在采取了保守策略：在每个volatile写的后面，或者在每个volatile读的前面插入一个StoreLoad屏障。==**从整体执行效率的角度考虑，JMM最终选择了在每个volatile写的后面插入一个StoreLoad屏障。**因为volatile写-读内存语义的常见使用模式是：一个写线程写volatile变量，多个读线程读同一个volatile变量。**当读线程的数量大大超过写线程时，选择在volatile写之后插入StoreLoad屏障将带来可观的执行效率的提升。从这里可以看到JMM在实现上的一个特点：首先确保正确性，然后再去追求执行效率。

下面是在保守策略下，volatile读插入内存屏障后生成的指令序列示意图，如图：![image-20190122170008800](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190122170008800.png)

​	上图中LoadLoad屏障用来禁止处理器把上面的volatile读与下面的普通读重排序。LoadStore屏障用来禁止处理器把上面的volatile读与下面的普通写重排序。

​	**上述volatile写和volatile读的内存屏障插入策略非常保守。在实际执行时，只要不改变volatile写-读的内存语义，编译器可以根据具体情况省略不必要的屏障。**

```java
class VolatileBarrierExample {
	int a;
	volatile int v1 = 1;
	volatile int v2 = 2;
	void readAndWrite() {
		int i = v1; // 第一个volatile读
		int j = v2; // 第二个volatile读
		a = i + j; // 普通写
		v1 = i + 1; // 第一个volatile写
		v2 = j * 2; // 第二个 volatile写
	}
	…
    // 其他方法
    }
```

针对readAndWrite()方法，编译器在生成字节码时可以做如下的优化。![image-20190122170308412](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190122170308412.png)

​	**注意，最后的StoreLoad屏障不能省略。因为第二个volatile写之后，方法立即return。此时编译器可能无法准确断定后面是否会有volatile读或写，为了安全起见，编译器通常会在这里插入一个StoreLoad屏障。**

​	上面的优化针对任意处理器平台，由于不同的处理器有不同“松紧度”的处理器内存模型，内存屏障的插入还可以根据具体的处理器内存模型继续优化。

# 5 锁的内存语义

## 5.1 锁的释放-获取建立的happens-before关系

​	锁是Java并发编程中最重要的同步机制。锁除了让临界区互斥执行外，还可以让释放锁的线程向获取同一个锁的线程发送消息。

## 5.2 锁的释放和获取的内存语义

​	**当线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存中。**

​	**当线程获取锁时，JMM会把该线程对应的本地内存置为无效。从而使得被监视器保护的临界区代码必须从主内存中读取共享变量。**

​	对比锁释放-获取的内存语义与volatile写-读的内存语义可以看出：锁释放与volatile写有相同的内存语义；锁获取与volatile读有相同的内存语义。总结如下：

- 线程A释放一个锁，实质上是线程A向接下来将要获取这个锁的某个线程发出了（线程A对共享变量所做修改的）消息。
- 线程B获取一个锁，实质上是线程B接收了之前某个线程发出的（在释放这个锁之前对共享变量所做修改的）消息。
- 线程A释放锁，随后线程B获取这个锁，这个过程实质上是线程A通过主内存向线程B发送消息。

## 5.3 锁内存语义的实现

​	通过ReentranLock来辅助说明。

​	在ReentrantLock中，调用lock()方法获取锁；调用unlock()方法释放锁。**ReentrantLock的实现依赖于Java同步器框架AbstractQueuedSynchronizer（本文简称之为AQS）。AQS使用一个整型的volatile变量（命名为state）来维护同步状态，**==这个volatile变量是ReentrantLock内存语义实现的关键。==

​	部分类图如下：![image-20190122174025886](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190122174025886.png)

​	ReentrantLock分为公平锁和非公平锁，我们首先分析公平锁。**使用公平锁时，加锁方法lock()调用轨迹如下。**

**1）ReentrantLock:lock()。**

**2）FairSync:lock()。**

**3）AbstractQueuedSynchronizer:acquire(int arg)。**

**4）ReentrantLock:tryAcquire(int acquires)。**

在第4步真正开始加锁，下面是该方法的源代码：

```Java
/**
 * Fair version of tryAcquire.  Don't grant access unless
 * recursive call or no waiters or is first.
 */
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();		// 获取锁的开始，首先读volatile变量state
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
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

​	从上面源代码中我们可以看出，==加锁方法首先读volatile变量state==。在使用公平锁时，解锁方法unlock()调用轨迹如下。

**1）ReentrantLock:unlock()。**

**2）AbstractQueuedSynchronizer:release(int arg)。**

**3）Sync:tryRelease(int releases)。**

在第3步真正开始释放锁，下面是该方法的源代码。	

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);		// 释放锁的最后，写volatile变量state
    return free;
}
```

​	从上面的源代码可以看出，==在释放锁的最后写volatile变量state。公平锁在释放锁的最后写volatile变量state，在获取锁时首先读这个volatile变量。==根据volatile的happens-before规则，**释放锁的线程在写volatile变量之前可见的共享变量，在获取锁的线程读取同一个volatile变量后将立即变得对获取锁的线程可见。**

​	非公平锁的释放和公平锁完全一样，所以这里只分析非公平锁的获取。使用非公平锁时，加锁方法lock()调用轨迹如下。

1）ReentrantLock:lock()。

2）NonfairSync:lock()。

3）AbstractQueuedSynchronizer:compareAndSetState(int expect,int update)。

在第3步真正开始加锁，下面是该方法的源代码。

```Java
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

​	**该方法以原子操作的方式更新state变量，其实就是CAS**，如果当前状态值等于预期值，则以原子方式将同步状态设置为给定的更新值。此操作具有volatile读和写的内存语义。

​	分别从编译器和处理器的角度来分析，CAS如何同时具有volatile读和volatile写的内存语义。编**译器不会对volatile读与volatile读后面的任意内存操作重排序；编译器不会对volatile写与volatile写前面的任意内存操作重排序。**组合这两个条件，意味着**==为了同时实现volatile读和volatile写的内存语义，编译器不能对CAS与CAS前面和后面的任意内存操作重排序。==**

​	下面分析在常见的intel X86处理器中，CAS是如何同时具有volatile读和volatile写的内存语义的。

下面是sun.misc.Unsafe类的compareAndSwapInt()方法的源代码。

```Java
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

​	可以看出这是一个本地方法，即由非Java语言实现的，对应的intel X86处理器的源代码片段如下：

![image-20190122175201387](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190122175201387.png)

​	如上面源代码所示，**程序会根据当前处理器的类型来决定是否为cmpxchg指令添加lock前缀。**==如果程序是在多处理器上运行，就为cmpxchg指令加上lock前缀（Lock Cmpxchg）。反之，如果程序是在单处理器上运行，就省略lock前缀（单处理器自身会维护单处理器内的顺序一致性，不需要lock前缀提供的内存屏障效果）。==

### ​intel的手册对lock前缀的说明如下：

#### 1）确保对内存的读-改-写操作原子执行。

​	**在Pentium及Pentium之前的处理器中，带有lock前缀的指令在执行期间会锁住总线，使得其他处理器暂时无法通过总线访问内存。很显然，这会带来昂贵的开销。**从Pentium 4、Intel Xeon及P6处理器开始，Intel使用缓存锁定（Cache Locking）来保证指令执行的原子性。缓存锁定将大大降低lock前缀指令的执行开销。

#### 2）禁止该指令，与之前和之后的读和写指令重排序。

#### 3）把写缓冲区中的所有数据刷新到内存中。

​	**上面的第2点和第3点所具有的内存屏障效果，足以同时实现volatile读和volatile写的内存语义。**

现在对公平锁和非公平锁的内存语义做个总结：

- 公平锁和非公平锁释放时，最后都要写一个volatile变量state。
- 公平锁获取时，首先会去读volatile变量。
- **非公平锁获取时，首先会用CAS更新volatile变量，这个操作同时具有volatile读和volatile写的内存语义。**

​	由此看出，锁释放-获取的内存语义的实现至少有下面两种方式：

1）利用volatile变量的写-读所具有的内存语义。

2）利用CAS所附带的volatile读和volatile写的内存语义。

## 5.4 concurrent包的实现

##### ​	由于Java的CAS同时具有volatile读和volatile写的内存语义，因此Java线程之间的通信现在有了下面4种方式：

1）A线程写volatile变量，随后B线程读这个volatile变量。

2）A线程写volatile变量，随后B线程用CAS更新这个volatile变量。

3）A线程用CAS更新一个volatile变量，随后B线程用CAS更新这个volatile变量。

4）A线程用CAS更新一个volatile变量，随后B线程读这个volatile变量。

​	**Java的CAS会使用现代处理器上提供的高效机器级别的原子指令，这些原子指令以原子方式对内存执行读-改-写操作，这是在多处理器中实现同步的关键**。同时，**volatile变量的读/写和CAS可以实现线程之间的通信**。concurrent包的源代码有通用化的实现模式：

- 首先，声明共享变量为volatile。
- 然后，使用CAS的原子条件更新来实现线程之间的同步。
- 同时，配合以volatile的读/写和CAS所具有的volatile读和写的内存语义来实现线程之间的通信。

​	AQS，非阻塞数据结构和原子变量类（java.util.concurrent.atomic包中的类），这些concurrent包中的基础类都是使用这种模式来实现的，而concurrent包中的高层类又是依赖于这些基础类来实现的。从整体来看，concurrent包的实现示意图：![image-20190122175907088](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190122175907088.png)

# 6 final域的内存语义

## 6.1 final域的重排序规则

​	对于final域，编译器和处理器要遵守两个重排序规则：

**1）在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。**(即对构造函数的写入要在这个对象被其他线程看到之后)

**2）初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。**

```java
public class FinalExample {
	int i; // 普通变量
	final int j; // final变量
	static FinalExample obj;
	public FinalExample () { // 构造函数
		i = 1; // 写普通域
		j = 2; // 写final域
	}
	public static void writer () { // 写线程A执行
		obj = new FinalExample ();
	}
	public static void reader () { // 读线程B执行
		FinalExample object = obj; // 读对象引用
		int a = object.i; // 读普通域
		int b = object.j; // 读final域
	}
}
```

​	假设线程A执行writer()方法，随后线程B执行reader()方法。

## 6.2 写final域的重排序规则

​	写final域的重排序规则禁止把final域的写重排序到构造函数之外。这个规则的实现包含下面2个方面：

1）JMM禁止编译器把final域的写重排序到构造函数之外。

2）编译器会在final域的写之后，构造函数return之前，插入一个StoreStore屏障。这个屏障禁止处理器把final域的写重排序到构造函数之外。

​	现在让我们分析writer()方法。writer()方法只包含一行代码：finalExample=newFinalExample()。这行代码包含两个步骤，如下：

1）构造一个FinalExample类型的对象。

2）把这个对象的引用赋值给引用变量obj。

​	假设线程B读对象引用与读对象的成员域之间没有重排序，下图是一种可能的执行时序。在图中，写普通域的操作被编译器重排序到了构造函数之外，读线程B错误地读取了普通变量i初始化之前的值。**而写final域的操作，被写final域的重排序规则“限定”在了构造函数之内，读线程B正确地读取了final变量初始化之后的值。**

​	**写final域的重排序规则可以确保：在对象引用为任意线程可见之前，对象的final域已经被正确初始化过了，而普通域不具有这个保障。**如下图，在读线程B“看到”对象引用obj时，很可能obj对象还没有构造完成（对普通域i的写操作被重排序到构造函数外，此时初始值1还没有写入普通域i）。![image-20190123143937339](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123143937339.png)

## 6.3 读final域的重排序规则

​	**读final域的重排序规则是，在一个线程中，初次读对象引用与初次读该对象包含的final域，JMM禁止处理器重排序这两个操作（注意，这个规则仅仅针对处理器）。编译器会在读final域操作的前面插入一个==LoadLoad屏障==。**

​	**初次读对象引用与初次读该对象包含的final域，这两个操作之间存在间接依赖关系。由于编译器遵守间接依赖关系，因此编译器不会重排序这两个操作。**大多数处理器也会遵守间接依赖，也不会重排序这两个操作。但有少数处理器允许对存在间接依赖关系的操作做重排序（比如alpha处理器），这个规则就是专门用来针对这种处理器的。

##### ​	reader()方法包含3个操作：

- 初次读引用变量obj。
- 初次读引用变量obj指向对象的普通域j。
- 初次读引用变量obj指向对象的final域i。

现在假设写线程A没有发生任何重排序，同时程序在不遵守间接依赖的处理器上执行，下图是一种可能的执行时序。![image-20190123144242291](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123144242291.png)

​	在上图中，**读对象的普通域的操作被处理器重排序到读对象引用之前。**读普通域时，该域还没有被写线程A写入，这是一个错误的读取操作。而读final域的重排序规则会把读对象final域的操作“限定”在读对象引用之后，此时该final域已经被A线程初始化过了，这是一个正确的读取操作。

​	**读final域的重排序规则可以确保：在读一个对象的final域之前，一定会先读包含这个final域的对象的引用。**在这个示例程序中，如果该引用不为null，那么引用对象的final域一定已经被A线程初始化过了。

## 6.4 final域为引用类型

![image-20190123144420142](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123144420142.png)	

​	本例final域为一个引用类型，它引用一个int型的数组对象。**对于引用类型，写final域的重排序规则对编译器和处理器增加了如下约束：在构造函数内对一个final引用的对象的成员域的写入，与随后在构造函数外把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。**

​	对上面的示例程序，假设首先线程A执行writerOne()方法，执行完后线程B执行writerTwo()方法，执行完后线程C执行reader()方法。下图是一种可能的线程执行时序。在图中，1是对final域的写入，2是对这个final域引用的对象的成员域的写入，3是把被构造的对象的引用赋值给某个引用变量。这里除了前面提到的1不能和3重排序外，2和3也不能重排序。![image-20190123144543246](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123144543246.png)

​	**JMM可以确保读线程C至少能看到写线程A在构造函数中对final引用对象的成员域的写入。**即C至少能看到数组下标0的值为1。而写线程B对数组元素的写入，读线程C可能看得到，也可能看不到。JMM不保证线程B的写入对读线程C可见，因为写线程B和读线程C之间存在数据竞争，此时的执行结果不可预知。**如果想要确保读线程C看到写线程B对数组元素的写入，写线程B和读线程C之间需要使用同步原语（lock或volatile，防止指令重排）来确保内存可见性。**

## 6.5 为什么final引用不能从构造函数内“溢出”

​	写final域的重排序规则可以确保：在引用变量为任意线程可见之前，该引用变量指向的对象的final域已经在构造函数中被正确初始化过了。其实，要得到这个效果，还需要一个保证：**在构造函数内部，不能让这个被构造对象的引用为其他线程所见，也就是对象引用不能在构造函数中“逸出”。**![image-20190123144826606](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123144826606.png)

​	假设一个线程A执行writer()方法，另一个线程B执行reader()方法。**这里的操作2使得对象还未完成构造前就为线程B可见。**即使这里的操作2是构造函数的最后一步，且在程序中操作2排在操作1后面，执行read()方法的线程仍然可能无法看到final域被初始化后的值，因为这里的操作1和操作2之间可能被重排序。实际的执行时序可能如下图：![image-20190123145428052](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123145428052.png)

​	从上图可以看出：**在构造函数返回前，被构造对象的引用不能为其他线程所见，因为此时的final域可能还没有被初始化。**在构造函数返回后，任意线程都将保证能看到final域正确初始化之后的值。

​	写final域的重排序规则会要求编译器在final域的写之后，构造函数return之前插入一个StoreStore障屏。读final域的重排序规则要求编译器在读final域的操作前面插入一个LoadLoad屏障。

# 7 happens-before

## 7.1 JMM的设计

​	JMM把happens-before要求禁止的重排序分为了下面两类:**会改变程序执行结果的重排序和不会改变程序执行结果的重排序。**

​	JMM对这两种不同性质的重排序，采取了不同的策略：

- 对于会改变程序执行结果的重排序，JMM要求编译器和处理器必须禁止这种重排序。
- 对于不会改变程序执行结果的重排序，JMM对编译器和处理器不做要求（JMM允许这种重排序）。

​	**JMM其实是在遵循一个基本原则：只要不改变程序的执行结果（指的是单线程程序和正确同步的多线程程序），编译器和处理器怎么优化都行。**例如，如果编译器经过细致的分析后，认定一个锁只会被单个线程访问，那么这个锁可以被消除。再如，如果编译器经过细致的分析后，认定一个volatile变量只会被单个线程访问，那么编译器可以把这个volatile变量当作一个普通变量来对待。这些优化既不会改变程序的执行结果，又能提高程序的执行效率。

## 7.2 happens-before的定义

​	happens-before关系的定义如下：

1）如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。

2）两个操作之间存在happens-before关系，并不意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行。如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，那么这种重排序并不非法（也就是说，JMM允许这种重排序）。

​	**as-if-serial语义保证单线程内程序的执行结果不被改变，happens-before关系保证正确同步的多线程程序的执行结果不被改变。**

​	as-if-serial语义给编写单线程程序的程序员创造了一个幻境：单线程程序是按程序的顺序来执行的。happens-before关系给编写正确同步的多线程程序的程序员创造了一个幻境：正确同步的多线程程序是按happens-before指定的顺序来执行的。

## 7.3 happens-before规则

**1）程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。(单线程)**

**2）监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。**

**3）volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。**

**4）传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。**

**5）start()规则：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作。**

**6）join()规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。**	

PS:线程的join方法的意思是使得放弃当前线程的执行，并返回对应的线程，比如t1.join()表示停止当前的线程，并返回t1线程，即执行t1线程。

下面就来看看几个规则。

### 1.volatile变量规则：![image-20190123154021830](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123154021830.png)

​	如上图，由于程序顺序规则，即一个线程里面语句的执行是按照程序的先后顺序执行的，所以1 happens-before 2和3 happens-before 4，然后又由于volatile变量规则，即一个volatile变量的写要在volatile变量的读之前，所以2 happens-before 3 ，又由于传递性，所以1 happens-before 4。**这里的传递性是由volatile的内存屏障插入策略和volatile的编译器重排序规则共同来保证的。**

### 2.start()规则

​	假设线程A在执行的过程中，通过执行ThreadB.start()来启动线程B；同时，假设线程A在执行ThreadB.start()之前修改了一些共享变量，线程B在开始执行后会读这些共享变量。如下图：![image-20190123154517553](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123154517553.png)

​	如上图，同理，1 happens-before 2和3 happens-before 4，2 happens-before 4由start()规则产生。根据传递性，将有1 happens-before 4。这实意味着，线程A在执行ThreadB.start()之前对共享变量所做的修改，接下来在线程B开始执行后都将确保对线程B可见。

### 3.join()规则：

​	假设线程A在执行的过程中，通过执行ThreadB.join()来等待线程B终止；同时，假设线程B在终止之前修改了一些共享变量，线程A从ThreadB.join()返回后会读这些共享变量。

![image-20190123154749210](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123154749210.png)

​	如上图，由于join原则，2 happens-before 4，4 happens-before 5，所以2 happens-before 5，即线程B的所有操作都会被线程A所见。

# 8 双重检查锁定与延迟初始化

## 8.1 双重检查锁定及其存在的问题

​	在单例模式中，普通的单例模式是线程不安全的，如下：

```java
//懒汉式，线程不安全
public static class SingleTon2 {
    public static SingleTon2 instance = null;

    private SingleTon2() {
    }

    public static SingleTon2 getInstance() {
        if (instance == null) {
            instance = new SingleTon2();
        }
        return instance;
    }
}
```

​	理所当然地想到加锁，然后就变成下面这样：

```Java
//懒汉式，加锁实现线程安全，多线程环境下效率不高
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

​	但是这样效率不高，所以有人就提出双重检测：

```Java
public static class SingleTon7 {
    private static SingleTon7 instance = null;

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

​	但是其实这个方法也是线程不安全的，因为在实例化一个对象的时候，可能会发生重排序。先来看看实例化一个对象的过程，即**instance = new SingleTon7();**这一步的过程：

```
memory = allocate(); // 1：分配对象的内存空间
ctorInstance(memory); // 2：初始化对象
instance = memory; // 3：设置instance指向刚分配的内存地址
```

上面的这个过程可能发生想下面这样的重排序：

```
memory = allocate(); // 1：分配对象的内存空间
instance = memory; // 3：设置instance指向刚分配的内存地址
			// 注意，此时对象还没有被初始化！
ctorInstance(memory); // 2：初始化对象
```

​	在Java语言规范中，**所有线程在执行Java程序时必须要遵守intra-thread semantics。intra-thread semantics保证重排序不会改变单线程内的程序执行结果。**换句话说，==intra-thread semantics允许那些在单线程内，不会改变单线程程序执行结果的重排序。==上面3行伪代码的2和3之间虽然被重排序了，但这个重排序并不会违反intra-thread semantics。**这个重排序在没有改变单线程程序执行结果的前提下，可以提高程序的执行性能。**

​	接下来说一下什么是intra-thread semantics：![image-20190123171103150](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123171103150.png)

只要保证2排在4的前面，即使2和3之间重排序了，也不会违反intra-threadsemantics。下面看看多线程访问的情况：![image-20190123171342058](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123171342058.png)

​	由于单线程内要遵守intra-thread semantics，从而能保证A线程的执行结果不会被改变。但是，当线程A和B按上图的时序执行时，B线程将看到一个还没有被初始化的对象。下表是具体多线程执行时序表：![image-20190123171519877](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123171519877.png)

​	这里A2和A3虽然重排序了，但Java内存模型的intra-thread semantics将确保A2一定会排在A4前面执行。因此，线程A的intra-thread semantics没有改变，但A2和A3的重排序，将导致线程B在B1处判断出instance不为空，线程B接下来将访问instance引用的对象。此时，线程B将会访问到一个还未初始化的对象。

## 8.2 基于volatile的解决方案

​	前面说了，在实例化一个对象的过程中，可能发生重排序，所以我们可以利用之前学过的volatile这个关键字禁止重排序。如下面的代码：

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

![image-20190123173102641](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123173102641.png)

## 8.3 基于类初始化的解决方案

​	**JVM在类的初始化阶段（即在Class被加载后，且被线程使用之前），会执行类的初始化。在执行类的初始化期间，JVM会去获取一个锁。这个锁可以同步多个线程对同一个类的初始化。**基于这个特性，可以实现另一种线程安全的延迟初始化方案。

```java
public class InstanceFactory {
    private static class InstanceHolder {
        public static Instance instance = new Instance();
    }

    public static Instance getInstance() {
        return InstanceHolder.instance; // 这里将导致InstanceHolder类被初始化
        }
   }
```

​	假设两个线程并发执行getInstance()方法，下面是执行的示意图:![image-20190123174012155](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123174012155.png)

​	其实在实例化对象的过程中还是发生了重排序，但是这个时候对于线程B是不可见的，而线程A内的重排序并没有什么影响。

​	**初始化一个类，包括执行这个类的静态初始化和初始化在这个类中声明的静态字段。根据Java语言规范，在首次发生下列任意一种情况时，一个类或接口类型T将被立即初始化。**

​	1）T是一个类，而且一个T类型的实例被创建。

​	2）T是一个类，且T中声明的一个静态方法被调用。

​	3）T中声明的一个静态字段被赋值。

​	4）T中声明的一个静态字段被使用，而且这个字段不是一个常量字段。

​	5）T是一个顶级类，而且一个断言语句嵌套在T内部被执行。

​	在上面的代码中，首次执行getInstance()方法的线程将导致InstanceHolder类被初始化（符合情况4）。**由于Java语言是多线程的，多个线程可能在同一时间尝试去初始化同一个类或接口（比如这里多个线程可能在同一时刻调用getInstance()方法来初始化InstanceHolder类）**。因此，在Java中初始化一个类或者接口时，需要做细致的同步处理。

​	**Java语言规范规定，对于每一个类或接口C，都有一个唯一的初始化锁LC与之对应。**从C到LC的映射，由JVM的具体实现去自由实现。**JVM在==类初始化期间==会获取这个初始化锁，并且==每个线程至少获取一次锁==来确保这个类已经被初始化过了。**

Java初始化一个类或接口的处理过程如下：

##### ​	第1阶段：通过在Class对象上同步（即获取Class对象的初始化锁），来控制类或接口的初始化。这个获取锁的线程会一直等待，直到当前线程能够获取到这个初始化锁。

假设Class对象当前还没有被初始化（初始化状态state，此时被标记为state=noInitialization），且有两个线程A和B试图同时初始化这个Class对象。图3-41是对应的示意图：![image-20190123174503040](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123174503040.png)

![image-20190123174518016](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123174518016.png)

##### 第2阶段：线程A执行类的初始化，同时线程B在初始化锁对应的condition上等待。

​							**类初始化第二阶段执行时序表**![image-20190123174650814](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123174650814.png)

![image-20190123174727202](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123174727202.png)

​	**这里线程B得到的是正在初始化中，所以他会释放初始化锁，然后进行等待。**

##### 第3阶段：线程A设置state=initialized，然后唤醒在condition中等待的所有线程。

![image-20190123174934525](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123174934525.png)

![image-20190123174955756](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123174955756.png)

##### 第4阶段：线程B结束类的初始化处理。

![image-20190123175023131](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123175023131.png)

![image-20190123175043184](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123175043184.png)

![image-20190123175127050](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123175127050.png)

​	**线程A在第2阶段的A1执行类的初始化，并在第3阶段的A4释放初始化锁；**线程B在第4阶段的B1获取同一个初始化锁，并在第4阶段的B4之后才开始访问这个类。

​	根据Java内存模型规范的锁规则，这里将存在如下的happens-before关系。**这个happens-before关系将保证：线程A执行类的初始化时的写入操作（执行类的静态初始化和初始化类中声明的静态字段），线程B一定能看到。**

##### 第5阶段：线程C执行类的初始化的处理。

![image-20190123175227311](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123175227311.png)

![image-20190123175236046](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190123175236046.png)

​	在第3阶段之后，类已经完成了初始化。因此线程C在第5阶段的类初始化处理过程相对简单一些（前面的线程A和B的类初始化处理过程都经历了两次锁获取-锁释放，而线程C的类初始化处理只需要经历一次锁获取-锁释放）。

​	线程A在第2阶段的A1执行类的初始化，并在第3阶段的A4释放锁；线程C在第5阶段的C1获取同一个锁，并在在第5阶段的C4之后才开始访问这个类。

​	根据Java内存模型规范的锁规则，将存在如下的happens-before关系。这个happens-before关系将保证：线程A执行类的初始化时的写入操作，线程C一定能看到。

**PS:这里的condition和state标记是虚构出来的。Java语言规范并没有硬性规定一定要使用condition和state标记。JVM的具体实现只要实现类似功能即可。**

​	通过对比，我们会发现基于类初始化的方案的实现代码更简洁，但**基于volatile的双重检查锁定的方案除了可以对静态字段实现延迟初始化外，还可以对实例字段实现延迟初始化。**

​	字段延迟初始化降低了初始化类或创建实例的开销，但增加了访问被延迟初始化的字段的开销。在大多数时候，正常的初始化要优于延迟初始化。如果确实需要对实例字段使用线程安全的延迟初始化，请使用基于volatile的延迟初始化的方案；如果确实需要对静态字段使用线程安全的延迟初始化，请使用基于类初始化的方案。





参照：《Java并发编程的艺术》























































