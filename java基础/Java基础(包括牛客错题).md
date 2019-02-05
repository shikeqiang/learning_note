# 	1.继承

​	在java中，**子类构造器会默认调用super()**(无论构造器中是否写有super()),用于初始化父类成员，同时当父类中存在有参构造器时，必须提供无参构造器，**子类构造器中并不会自动继承有参构造器，仍然默认调用super()，使用无参构造器。因此，一个类想要被继承必须提供无参构造器。**

**==PS：方法没有继承一说，只有重载和重写==**

![image-20181226080624787](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226080624787-5782784-7514330-8895518.png)

​	==编译看左边，运行看右边，编译的时候看等号左边定义的是什么类型，运行的时候动态绑定到哪个对象上==,如：Base b = new Sub();**它为多态的一种表现形式，声明是Base,实现是Sub类，** **理解为** **b** **编译时表现为Base类特性，运行时表现为Sub类特性。**

​	**如果父类中的某个方法使用了 synchronized关键字，而子类中也覆盖了这个方法，默认情况下子类中的这个方法并不是同步的，必须显示的在子类的这个方法中加上 synchronized关键字才可。**当然，也可以在子类中调用父类中相应的方法，这样虽然子类中的方法并不是同步的，但子类调用了父类中的同步方法，也就相当子类方法也同步了。

## 接口

​	**接口继承的时候只能继承接口不能继承类**，因为如果类可以存在非抽象的成员，如果接口继承了该类，那么接口必定从类中也继承了这些非抽象成员，这就和接口的定义相互矛盾。

​	**接口只能继承接口，但是可以多继承。类都是单继承，但是继承有传递性。**

​	==java8在接口中引入了默认方法，通过在方法前加上default关键字就可以在接口中写方法的默认实现。==即接口实现类可以不用重写默认方法，可以直接调用。

# 2.函数

#### 调用过程：

​	**调用某个函数实际上是将程序执行顺序转移到该函数所存放在内存中某个地址，将函数的程序内容执行完后，再返回到转去执行该函数前的地方。**这种转移操作要求在转去前要保护现场并记忆执行的地址，转回后先要恢复现场，并按原来保存地址继续执行。也就是通常说的==压栈和出栈。==因此，函数调用要有一定的时间和空间方面的开销。那么对于那些函数体代码不是很大，又频繁调用的函数来说，这个时间和空间的消耗会很大。  

#### 内联函数：

​	**==内联函数就是在程序编译时，编译器将程序中出现的内联函数的调用表达式用内联函数的函数体来直接进行替换。==**显然，这样就不会产生转去转回的问题，但是由于在编译时将函数体中的代码被替代到程序中，因此会增加目标程序代码量，进而增加空间开销，而在时间代销上不象函数调用时  那么大，**可见它是以目标代码的增加为代价来换取时间的节省。**  

Java中的内联函数：在java中使用final关键字来指示一个函数为内联函数，例如：  

```Java
public final void method1() {     
   //TODO something     
}  
```

​    这个指示并不是必需的。final关键字只是告诉编译器，在编译的时候考虑性能的提升，可以将final函数视为内联函数。但最后编译器会怎么处理，编译器会分析将final函数处理为内联和不处理为内联的性能比较了。

# 3.修饰符

![image-20181226080607563](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226080607563-5782767-7514330.png)

![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/9430388_1508736012435_F21604BDD6B5A8912481FAC56612272B.png)

# 4.异常

##### 	如果try...catch捕捉到异常之后，程序会结束，即finally后的语句不会执行。

![image-20181226080651234](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226080651234-5782811-7514330.png)

##### finally一定会在return之前执行，但是如果finally使用了return语句，将会使trycatch中的return或者throw失效。

![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/70-20181227100908360-7514330.png)

##### 	对于运行时异常，java编译器不要求必须进行异常捕获处理或者抛出声明，由程序员自行决定。运行异常，可以通过java虚拟机来自行处理。

捕获到的异常不仅可以在当前方法中处理，**还可以将异常抛给调用它的上一级方法来处理。**

# 5.线程

**在java中线程是有分优先等级的所以优先级不能相同。**

![image-20181226080723036](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226080723036-5782843-7514330.png)

![image-20181226080749306](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226080749306-5782869-7514330.png)

![image-20181227102908360](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181227102908360-5877748-7514330.png)
易知：每个线程对a 均做了两次读写操作，分别是 “ +1 ” 和 “ -2 ”

而题目问了是最终a 的结果，所以 a 的结果取决于各自线程对 a 的先后读写的顺序

结论：a的可能取值为-1、0、-2

![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/995326_1498884304176_23ACE3CCE24ADA857BC64F71F3A561F0-7514330.)

##### 前台线程和后台线程

​	==main()函数即主函数，是一个前台线程==，**前台进程是程序中必须执行完成的**，而后台线程则是java中所有前台结束后结束，不管有没有完成，后台线程主要用于内存分配等方面。                                                                                        
前台线程和后台线程的区别和联系：

1、后台线程不会阻止进程的终止。属于某个进程的所有前台线程都终止后，该进程就会被终止。所有剩余的后台线程都会停止且不会完成。

2、可以在任何时候将前台线程修改为后台线程，方式是设置Thread.IsBackground 属性。

3、不管是前台线程还是后台线程，如果线程内出现了异常，都会导致进程的终止。

4、托管线程池中的线程都是后台线程，使用new Thread方式创建的线程默认都是前台线程。

说明：   

​        ==应用程序的主线程以及使用Thread构造的线程都默认为前台线程==                       
​    使用Thread建立的线程默认情况下是前台线程，在进程中，只要有一个前台线程未退出，进程就不会终止。主线程就是一个前台线程。而后台线程不管线程是否结束，只要所有的前台线程都退出（包括正常退出和异常退出）后，进程就会自动终止。一般后台线程用于处理时间较短的任务，如在一个Web服务器中可以利用后台线程来处理客户端发过来的请求信息。而前台线程一般用于处理需要长时间等待的任务，如在Web服务器中的监听客户端请求的程序，或是定时对某些系统资源进行扫描的程序。

​	**将一个线程标记成daemon线程，意味着当主线程结束，并且没有其它正在运行的非daemon线程时，该daemon线程也会自动结束。**

##### 看下面这道题输出结果：

```Java
public static void main(String[] args) throws InterruptedException {
        Thread t=new Thread(new Runnable() {
            public void run() {
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                System.out.print("2");
            }
        });
        t.start();
        t.join();
        System.out.print("1");
    }
```

​	输出结果为：21，执行顺序是，先创建一个线程t，然后t.start()，此时调用线程t的run方法，然后此时线程进入sleep状态，此时执行t.join方法，把指定的t线程加入到当前线程，即main线程中，此时main线程中的打印1就要等待t线程执行完才会执行。如果没有sleep和join的话，结果可能是12 或 21。

##### 并发和并行的区别：

**一个处理器同时处理多个任务和多个处理器或者是多核的处理器同时处理多个不同的任务。**

并发性(concurrency)，又称共行性，是**指能处理多个同时性活动的能力，并发事件之间不一定要同一时刻发生。**

并行(parallelism)是指**同时发生的两个并发事件，**具有并发的含义，而并发则不一定并行。

​	==银行家算法==是一种最有代表性的避免[死锁](http://baike.baidu.com/view/121723.htm)的算法。又被称为“资源分配拒绝”法。在避免[死锁](http://baike.baidu.com/view/121723.htm)方法中允许进程动态地申请资源，但系统在进行资源分配之前，应先计算此次分配资源的安全性，若分配不会导致系统进入不安全状态，则分配，否则等待。为实现银行家算法，系统必须设置若干[数据结构](http://baike.baidu.com/view/9900.htm)。

**轮询调度：**每一次把来自用户的请求轮流分配给内部中的服务器，从1开始，直到N(内部服务器个数)，然后重新开始循环。**只有在当前任务主动放弃CPU控制权的情况下（比如任务挂起），才允许其他任务（包括高优先级的任务）控制CPU。**其优点是其简洁性，它无需记录当前所有连接的状态，所以它是一种无状态调度。但不利于后面的请求及时得到响应。

**抢占式调度：**允许高优先级的任务打断当前执行的任务，抢占CPU的控制权。这有利于后面的高优先级的任务也能及时得到响应。但实现相对较复杂且可能出现低优先级的任务长期得不到调度。

#### 产生死锁的原因主要是： 

（1） 因为系统资源不足。 
（2） 进程运行推进的顺序不合适。 
（3） 资源分配不当等。 

#### 产生死锁的四个必要条件： 

（1）互斥条件：一个资源每次只能被一个进程使用。 
（2）请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。 
（3）不可剥夺条件：进程已获得的资源，在末使用完之前，不能强行剥夺。 
（4）循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系。

### sleep和wait方法区别：

相同点：

​	wait()和sleep()都可以通过interrupt()方法 打断线程的暂停状态 ，从而使线程立刻抛出InterruptedException。 
​	**如果线程A希望立即结束线程B，则可以对线程B对应的Thread实例调用interrupt方法。**如果此刻线程B正在wait/sleep/join，则线程B会立刻抛出InterruptedException，在catch() {} 中直接return即可安全地结束线程。 
需要注意的是，InterruptedException是线程自己从内部抛出的，并不是interrupt()方法抛出的。对某一线程调用 interrupt()时，如果该线程正在执行普通的代码，那么该线程根本就不会抛出InterruptedException。但是，一旦该线程进入到 wait()/sleep()/join()后，就会立刻抛出InterruptedException 。 	

不同点：

​	1.每个对象都有一个锁来控制同步访问。Synchronized关键字可以和对象的锁交互，来实现线程的同步。 
**sleep方法没有释放锁，而wait方法释放了锁，使得其他线程可以使用同步控制块或者方法。** 
​	2.wait，notify和notifyAll只能在同步控制方法或者同步控制块里面使用，而sleep可以在任何地方使用 

​	3.sleep必须捕获异常，而wait，notify和notifyAll不需要捕获异常 

​	**4.sleep是线程类（Thread）的方法，导致此线程暂停执行指定时间，给执行机会给其他线程，但是监控状态依然保持，到时后会自动恢复。调用sleep不会释放对象锁。**

​	5.wait是Object类的方法，对此对象调用wait方法导致本线程放弃对象锁，进入等待此对象的等待锁定池，**==只有针对此对象发出notify方法（或notifyAll）后本线程才进入对象锁定池准备获得对象锁进入运行状态。==**

# 6.原始jdbc执行语句

## 6.1 结果集(ResultSet)

##### 	结果集(ResultSet)是数据中查询结果返回的一种对象，可以说结果集是一个存储查询结果的对象，但是结果集并不仅仅具有存储的功能，他同时还具有操纵数据的功能，可能完成对数据的更新等。 

​	结果集读取数据的方法主要是getXXX() ，他的参数可以是整型，表示第几列（是从1开始的），还可以是列名，返回的是对应的XXX类型的值。如果对应那列是空值，XXX是对象的话返回XXX型的空值，如果XXX是数字类型，如Float等则返回0，boolean返回false。

​	**使用getString()可以返回所有的列的值，不过返回的都是字符串类型的。**XXX可以代表的类型有：基本的数据类型如整型(int)，布尔型(Boolean)，浮点型(Float,Double)等，比特型（byte），还包括一些特殊的类型，如：日期类型（java.sql.Date），时间类型(java.sql.Time)，时间戳类型 (java.sql.Timestamp)，大数型(BigDecimal和BigInteger等)等。还可以使用getArray(int colindex/String columnname)，通过这个方法获得当前行中，colindex所在列的元素组成的对象的数组。使用getAsciiStream(int colindex/String colname)可以获得该列对应的当前行的ascii流。**也就是说所有的getXXX方法都是对当前行进行操作。** 

 	结果集从其使用的特点上可以分为四类，这四类的结果集的所具备的特点都是和Statement语句的创建有关，因为结果集是通过Statement语句执行后产生的，所以可以说，**结果集具备何种特点，完全决定于Statement**，首先是无参数类型的，他对应的就是下面要介绍的基本的ResultSet对应的Statement。下面的代码中用到的Connection并没有对其初始化，变量conn代表的就是Connection对应的对象。SqlStr代表的是响应的SQL语句。 

### 1、最基本的ResultSet。 

​	其中，CreateStatement属于Connection接口

​	之所以说是最基本的ResultSet，是因为**这个ResultSet他起到的作用就是完成了查询结果的存储功能，而且只能读取一次，不能够来回的滚动读取**。这种结果集的创建方式如下： 

```Java
Statement st = conn.CreateStatement() //不带参数的CreateStatement方法
ResultSet rs = Statement.excuteQuery(sqlStr); //执行sql语句
```

​	由于这种结果集不支持滚动的读取功能，**所以如果获得这样一个结果集，只能使用它里面的==next()方法==，逐个的读去数据。** 

### 2、可滚动的ResultSet类型。 

​	这个类型支持前后滚动取得记录next（）、previous()，回到第一行first()，同时还支持要去的ResultSet中的第几行 absolute（int n），以及移动到相对当前行的第几行relative(int n)，要实现这样的ResultSet在创建Statement时用如下的方法。 

```
Statement st = conn.createStatement (int resultSetType, int resultSetConcurrency) 
ResultSet rs = st.executeQuery(sqlStr) 
```

其中两个参数的意义是： 

##### resultSetType 是设置 ResultSet 对象的类型可滚动，或者是不可滚动。取值如下： 

​       ResultSet.TYPE_FORWARD_ONLY 	只能向前滚动 

​       **ResultSet.TYPE_SCROLL_INSENSITIVE 和 Result.TYPE_SCROLL_SENSITIVE 这两个取值都能够实现任意的前后滚动，使用各种移动的 ResultSet 指针的方法。二者的区别在于前者对于修改不敏感，而后者对于修改敏感。** 

##### resultSetConcurency 是设置 ResultSet 对象能够修改的，取值如下： 

​       ResultSet.CONCUR_READ_ONLY 	设置为只读类型的参数。 

​       ResultSet.CONCUR_UPDATABLE		设置为可修改类型的参数。 

**这些参数是ResultSet接口里面封装好的一些变量，都是用int类型的数值表示。**

所以如果只是想要可以滚动的类型的 Result 只要把 Statement 如下赋值就行了。 

```
Statement st = conn.createStatement(Result.TYPE_SCROLL_INSENITIVE, ResultSet.CONCUR_READ_ONLY); 
ResultSet rs = st.executeQuery(sqlStr) ； 
```

用这个 Statement 执行的查询语句得到的就是可滚动的 ResultSet 。 

### 3、可更新的ResultSet 

​	这样的ResultSet对象可以完成对数据库中表的修改，但是我知道ResultSet只是相当于数据库中表的视图，所以并不是所有的ResultSet只要设置了可更新就能够完成更新的，**能够完成更新的ResultSet的SQL语句必须要具备如下的属性：** 

​    **a 、只引用了单个表。** 

​    **b 、不含有join或者group by子句。** 

​    **c 、那些列中要包含主关键字。** 

​    具有上述条件的，可更新的ResultSet可以完成对数据的修改，可更新的结果集的创建方法是： 

```
Statement st = createstatement(Result.TYPE_SCROLL_INSENSITIVE,Result.CONCUR_UPDATABLE) 
```

### 4、可保持的ResultSet 

​	正常情况下如果使用Statement执行完一个查询，又去执行另一个查询的时候，第一个查询的结果集就会被关闭，也就是说，**所有的Statement的查询对应的结果集是一个**，如果调用Connection的commit()方法也会关闭结果集。可保持性就是指当ResultSet的结果被提交时，是被关闭还是不被关闭。JDBC2.0和1.0提供的都是提交后ResultSet就会被关闭。不过在JDBC3.0中，我们可以设置ResultSet是否关闭。要完成这样的ResultSet的对象的创建，要使用的Statement的创建要具有三个参数，这个Statement的创建方式也就是上面的Statement的第三种创建方式。 

​	当使用ResultSet的时候，当查询出来的数据集记录很多，有一千万条的时候，那rs所指的对象是否会占用很多内存，如果记录过多，那程序会不会把系统的内存用光呢 ？不会的，**ResultSet表面看起来是一个记录集，其实这个对象中只是记录了结果集的相关信息，具体的记录并没有存放在对象中，具体的记录内容直到你通过next方法提取的时候，再通过相关的getXXXXX方法提取字段内容的时候才能从数据库中得到**，这些并不会占用内存，==具体消耗内存是由于你将记录集中的数据提取出来加入到你自己的集合中的时候才会发生，如果你没有使用集合记录所有的记录就不会发生消耗内存厉害的情况。==

## 6.2 相关执行sql语句的方法

**1、executeUpdate(sql)**

**用于执行给定 SQL 语句，该语句可能为 INSERT、UPDATE 或 DELETE 语句，或者不返回任何内容的 SQL 语句。返回值是更新的条数。**

**2、executeQuery（sql）**

**这个方法被用来执行 SELECT 语句，返回代表查询结果的ResultSet对象。**

## 6.3 Statement

​	Statement是sql语句的载体，是标准的Statement类，通过字符串对sql语句进行拼接，但是它存在sql注入的危险。

PreparedStatement对sql语句进行了预编译，可以防止SQL注入

CallableStatement用来调用存储过程的

BatchedStatement用于批量操作数据库，BatchedStatement不是标准的Statement类

# 7.编码

##### 标准ASCII只使用7个bit，扩展的ASCII使用8个bit。

​	ANSI通常使用 0x00~0x7f 范围的1 个[字节](https://baike.baidu.com/item/%E5%AD%97%E8%8A%82)来表示 1 个英文字符。超出此范围的使用0x80~0xFFFF来编码，即扩展的ASCII编码。不同 ANSI 编码之间互不兼容。在简体中文Windows操作系统中，ANSI 编码代表 GBK 编码；在繁体中文Windows操作系统中，ANSI编码代表Big5；在日文Windows操作系统中，ANSI 编码代表 Shift_JIS 编码。

ANSI通常使用 0x00~0x7f 范围的1 个[字节](https://baike.baidu.com/item/%E5%AD%97%E8%8A%82)来表示 1 个英文字符，即ASCII码。ASCII码是ANSI码的子集

ASCII码包含一些特殊空字符，所以ASCII码不都是可打印字符

##### Java一律采用Unicode编码方式，每个字符无论中文还是英文字符都占用2个字节。

​	**Java语言中，中文字符所占的字节数取决于字符的编码方式，一般情况下，采用ISO8859-1编码方式时，一个中文字符与一个英文字符一样只占1个字节；采用GB2312或GBK编码方式时，一个中文字符占2个字节；而采用UTF-8编码方式时，一个中文字符会占3个字节。**char占两个字节。

# 8.子父类关系及执行顺序

##### ==父类静态代码块->子类静态代码块->父类非静态代码块->父类构造函数->子类非静态代码块->子类构造函数==

​	当实例化子类对象时，首先要加载父类的class文件进内存，静态代码块是随着类的创建而执行，所以父类静态代码块最先被执行，子类class文件再被加载，同理静态代码块被先执行；实例化子类对象要先调用父类的构造方法，而调用父类构造方法前会先执行父类的非静态代码块。这里的非静态代码块其实就是构造块。

**静态块：用static申明，JVM加载类时执行，仅执行一次；构造块：类中直接用{}定义，每一次创建对象时执行 **![image-20190101113345126](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190101113345126-6313625-7514330.png)

​	实例化子类的时候，如果子类重写了父类的方法，那么在父类的构造方法中调用这个方法的时候，实际上调用的是子类重写之后的方法。如下面这个，：父类构造函数中执行的test（）方法，因子类是重写了test（）方法的，因此父类构造函数中的test（）方法实际执行的是子类的test（）方法，所以

![image-20190101152437670](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190101152437670-6327477-7514330.png)

**子类的构造器第一行默认都是super()，默认调用直接父类的无参构造，一旦直接父类没有无参构造，那么子类必须显式的声明要调用父类或者自己的哪一个构造器。**

​	==如果子类的方法重写了父类的方法，那么子类中该方法的访问级别不允许低于父类的访问级别。==这是为了确保可以使用父类实例的地方都可以使用子类实例，也就是确保满足里氏替换原则。

# 9.Spring的事务管理

![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226081100232-5783060-7514330-8860046.png)

![image-20181226081118993](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226081118993-5783079-7514330.png)

# 10.垃圾回收

![image-20181226081136798](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226081136798-5783096-7514330.png)

### 堆内存设置原理：

##### JVM堆内存分为2块：Permanent Space 和 Heap Space。

- Permanent      即 持久代（Permanent Generation），**主要存放的是Java类定义信息，与垃圾收集器要收集的Java对象关系不大。**
- Heap = { Old + NEW      = {Eden, from, to} }，Old 即 年老代（Old Generation），New 即 年轻代（Young      Generation）。

年老代和年轻代的划分对垃圾收集影响比较大。

年轻代：所有新生成的对象首先都是放在年轻代。年轻代的目标就是尽可能快速的收集掉那些生命周期短的对象。年轻代一般分3个区，1个Eden区，2个Survivor区（from 和 to）。大部分对象在Eden区中生成。当Eden区满时，还存活的对象将被复制到Survivor区（两个中的一个），当一个Survivor区满时，此区的存活对象将被复制到另外一个Survivor区，当另一个Survivor区也满了的时候，从前一个Survivor区复制过来的并且此时还存活的对象，将可能被复制到年老代。

​	2个Survivor区是对称的，没有先后关系，所以同一个Survivor区中可能同时存在从Eden区复制过来对象，和从另一个Survivor区复制过来的对象；而复制到年老区的只有从另一个Survivor区过来的对象。而且，因为需要交换的原因，Survivor区至少有一个是空的。特殊的情况下，根据程序需要，Survivor区是可以配置为多个的（多于2个），这样可以增加对象在年轻代中的存在时间，减少被放到年老代的可能。

​	**针对年轻代的垃圾回收即 Young GC。**

年老代：在年轻代中经历了N次（可配置）垃圾回收后仍然存活的对象，就会被复制到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象。

​	**针对年老代的垃圾回收即 Full GC。**

持久代：用于存放静态类型数据，如 Java Class, Method 等。持久代对垃圾回收没有显著影响。但是有些应用可能动态生成或调用一些Class，例如 Hibernate CGLib 等，在这种时候往往需要设置一个比较大的持久代空间来存放这些运行过程中动态增加的类型。

##### 所以，当一组对象生成时，内存申请过程如下：

1. JVM会试图为相关Java对象在年轻代的Eden区中初始化一块内存区域。
2. 当Eden区空间足够时，内存申请结束。否则执行下一步。
3. JVM试图释放在Eden区中所有不活跃的对象（Young      GC）。释放后若Eden空间仍然不足以放入新对象，JVM则试图将部分Eden区中活跃对象放入Survivor区。
4. Survivor区被用来作为Eden区及年老代的中间交换区域。当年老代空间足够时，Survivor区中存活了一定次数的对象会被移到年老代。
5. 当年老代空间不够时，JVM会在年老代进行完全的垃圾回收（Full      GC）。
6. Full  GC后，若Survivor区及年老代仍然无法存放从Eden区复制过来的对象，则会导致JVM无法在Eden区为新生成的对象申请内存，即出现“Out of Memory”。

#### OOM（“Out of Memory”）异常一般主要有如下2种原因：

1. ##### 年老代溢出，表现为：java.lang.OutOfMemoryError:Javaheapspace

   这是最常见的情况，产生的原因可能是：设置的内存参数Xmx过小或程序的内存泄露及使用不当问题。

例如循环上万次的字符串处理、创建上千万个对象、在一段代码内申请上百M甚至上G的内存。还有的时候虽然不会报内存溢出，却会使系统不间断的垃圾回收，也无法处理其它请求。这种情况下除了检查程序、打印堆内存等方法排查，还可以借助一些内存分析工具，比如MAT就很不错。

##### 2. 持久代溢出，表现为：java.lang.OutOfMemoryError:PermGenspace

​	通常由于持久代设置过小，动态加载了大量Java类而导致溢出 ，解决办法唯有将参数 -XX:MaxPermSize 调大（一般256m能满足绝大多数应用程序需求）。将部分Java类放到容器共享区（例如Tomcat share lib）去加载的办法也是一个思路，但前提是容器里部署了多个应用，且这些应用有大量的共享类库。

​	JAVA程序不能依赖于垃圾回收的时间或者顺序，局部变量存放在栈上，栈上的垃圾回收，由finalize()来实现，而gc用来释放堆上的内存。

# 12.框架

## 1.SpringMVC和Struts2的区别

![image-20181226081244517](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226081244517-5783164-7514330.png)

### 1.  机制：spring mvc的入口是servlet，而struts2是filter。

​    **1.Filter 实现javax.servlet.Filter接口，在web.xml中配置与标签指定使用哪个Filter实现类过滤哪些URL链接**。只在web启动时进行初始化操作。 filter 流程是线性的， url传来之后，检查之后，可保持原来的流程继续向下执行，被下一个filter, servlet接收等，而servlet 处理之后，不会继续向下传递。filter功能可用来保持流程继续按照原来的方式进行下去，或者主导流程，而servlet的功能主要用来主导流程。 

特点：可以在响应之前修改Request和Response的头部，只能转发请求，不能直接发出响应。filter可用来进行字符编码的过滤，检测用户 是否登陆的过滤，禁止页面缓存等

​     2.Servlet， servlet 流程是短的，url传来之后，就对其进行处理，之后返回或转向到某一自己指定的页面。它主要用来在业务处理之前进行控制。Listener呢？我们知道 servlet、filter都是针对url之类的，而listener是针对对象的操作的，如session的创建，session.setAttribute的发生，在这样的事件发生时做一些事情。 

### 2.性能：              																			

​	**spring会稍微比struts快。 spring mvc是基于方法的设计 ， 而sturts是基于类 ，每次发一次请求都会实例一个action，每个action都会被注入属性，而spring基于方法，粒度更细(粒度级别的东西比较sychronized和lock)，但要小心把握像在servlet控制数据一样。** 

​	**spring3 mvc是方法级别的拦截，拦截到方法后根据参数上的注解，把request数据注入进去，在spring3 mvc中，一个方法对应一个request上下文。** 而struts2框架是类级别的拦截，每次来了请求就创建一个Action，然后调用setter getter方法把request中的数据注入；struts2实际上是通过setter getter方法与request打交道的；struts2中，一个Action对象对应一个request上下文。

### 3.参数传递

​	**struts是在接受参数的时候，可以用属性来接受参数，这就说明参数是让多个方法共享的。**

### 4.设计思想上

​	**struts更加符合oop的编程思想 ， spring就比较谨慎，在servlet上扩展。**

### 5.intercepter(拦截器)的实现机制

​	**struts有以自己的interceptor机制， spring mvc用的是独立的AOP方式 。**这样导致struts的配置文件量还是比spring mvc大，虽然struts的配置能继承，所以我觉得，就拿使用上来讲，spring mvc使用更加简洁， 开发效率Spring MVC确实比struts2高 。 **spring mvc是方法级别的拦截，一个方法对应一个request上下文，而方法同时又跟一个url对应，所以说从架构本身上 spring3 mvc就容易实现restful url 。** struts2是类级别的拦截，一个类对应一个request上下文；实现restful url要费劲，因为struts2 action的一个方法可以对应一个url；而其类属性却被所有方法共享，这也就无法用注解或其他方式标识其所属方法了。 spring3 mvc的方法之间基本上独立的，独享request response数据，请求数据通过参数获取，处理结果通过ModelMap交回给框架方法之间不共享变量， 而struts2搞的就比较乱，虽然方法之间也是独立的，但其所有Action变量是共享的，这不会影响程序运行，却给我们编码，读程序时带来麻烦。

**6. spring3 mvc的验证也是一个亮点，支持JSR303， 处理ajax的请求更是方便 ，只需一个注解 @ResponseBody  ，然后直接返回响应文本即可**。 代码：

```java
@RequestMapping (value= "/whitelists" )  
public  String index(ModelMap map) {  
    Account account = accountManager.getByDigitId(SecurityContextHolder.get().getDigitId());  
    List<Group> groupList = groupManager.findAllGroup(account.getId());  
    map.put("account" , account);  
    map.put("groupList" , groupList);  
    return   "/group/group-index" ;  
}  
// @ResponseBody ajax响应，处理Ajax请求也很方便   
@RequestMapping (value= "/whitelist/{whiteListId}/del" )  
@ResponseBody   
public  String delete( @PathVariable  Integer whiteListId) {  
    whiteListManager.deleteWhiteList(whiteListId);  
    return "success" ;  
} 

```

## 2.spring七大模块：

### 1.Spring Core：

​	Core封装包是框架的最基础部分，提供IOC和依赖注入特性。这里的基础概念是**BeanFactory，它提供对Factory模式的经典实现来消除对程序性单例模式的需要，并真正地允许你从程序逻辑中分离出依赖关系和配置。**

### 2.Spring Context: 

​	构建于[Core](http://www.mianwww.com/html/2014/03/19750.html#beans-introduction)封装包基础上的 [Context](http://blog.chinaunix.net/u/9295/ch03s08.html)封装包，提供了一种框架式的对象访问方法，有些象JNDI注册器。Context封装包的特性得自于Beans封装包，并添加了对国际化（I18N）的支持（例如资源绑定），事件传播，资源装载的方式和Context的透明创建，比如说通过Servlet容器。

### 3.Spring DAO: 

​	 [DAO](http://www.mianwww.com/html/2014/03/19750.html#dao-introduction) (Data Access Object)提供了JDBC的抽象层，它可消除冗长的JDBC编码和解析数据库厂商特有的错误代码。 并且，JDBC封装包还提供了一种比编程性更好的声明性事务管理方法，不仅仅是实现了特定接口，而且对所有的POJOs（plain old Java objects）都适用。

### 4.Spring ORM: 

​	[ORM](http://www.mianwww.com/html/2014/03/19750.html#orm-introduction) 封装包提供了常用的“对象/关系”映射APIs的集成层。 其中包括[JPA](http://blog.chinaunix.net/u/9295/ch12s07.html)、[JDO](http://blog.chinaunix.net/u/9295/ch12s03.html)、[Hibernate](http://blog.chinaunix.net/u/9295/ch12s02.html) 和 [iBatis](http://blog.chinaunix.net/u/9295/ch12s06.html) 。利用ORM封装包，可以混合使用所有Spring提供的特性进行“对象/关系”映射，如前边提到的简单声明性事务管理。

### 5.Spring AOP: 

​	Spring的 [AOP](http://www.mianwww.com/html/2014/03/19750.html#aop-introduction) 封装包提供了符合AOP Alliance规范的面向方面的编程实现，让你可以定义，例如方法拦截器（method-interceptors）和切点（pointcuts），从逻辑上讲，从而减弱代码的功能耦合，清晰的被分离开。而且，利用source-level的元数据功能，还可以将各种行为信息合并到你的代码中。

### 6.Spring Web: 

​	Spring中的 Web 包提供了基础的针对Web开发的集成特性，例如多方文件上传，利用Servlet listeners进行IOC容器初始化和针对Web的ApplicationContext。当与WebWork或Struts一起使用Spring时，这个包使Spring可与其他框架结合。

### 7.Spring Web MVC: 

​	Spring中的[MVC](http://www.mianwww.com/html/2014/03/19750.html#mvc-introduction)封装包提供了Web应用的Model-View-Controller（MVC）实现。Spring的MVC框架并不是仅仅提供一种传统的实现，它提供了一种清晰的分离模型，在领域模型代码和Web Form之间。并且，还可以借助Spring框架的其他特性。

## 3.struts1和Struts2的区别

  1.Struts1要求Action类继承一个抽象基类。Struts1的一个普遍问题是使用抽象类编程而不是接口。 
2. Struts 2 Action类可以实现一个Action接口，也可实现其他接口，使可选和定制的服务成为可能。Struts2提供一个ActionSupport基类去实现常用的接口。Action接口不是必须的，任何有execute标识的POJO对象都可以用作Struts2的Action对象。
  从Servlet 依赖分析: 
3. Struts1 Action 依赖于Servlet API ,因为当一个Action被调用时HttpServletRequest 和 HttpServletResponse 被传递给execute方法。 
4. Struts 2 Action不依赖于容器，允许Action脱离容器单独被测试。如果需要，Struts2 Action仍然可以访问初始的request和response。但是，其他的元素减少或者消除了直接访问HttpServetRequest 和 HttpServletResponse的必要性。
  从action线程模式分析: 
5. Struts1 Action是单例模式并且必须是线程安全的，因为仅有Action的一个实例来处理所有的请求。单例策略限制了Struts1 Action能作的事，并且要在开发时特别小心。Action资源必须是线程安全的或同步的。 
6. Struts2 Action对象为每一个请求产生一个实例，因此没有线程安全问题。（实际上，servlet容器给每个请求产生许多可丢弃的对象，并且不会导致性能和垃圾回收问题）
7. ![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/6740262_1502892230619_8AA0BB8C0EEED931C8EE12011A5E8E1B.png)

# 13.运算及运算符

### 1.==当字符型与整型运算时会自动转换成整型==，所以'a' % 3会变成97%3。而’a’ = 1/3是错误的，因为常量不能被赋值

![image-20181226081429859](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226081429859-5783269-7514330.png)

### 2.<< : 左移运算符，num << 1,相当于num乘以2，即2的一次方

\>\>: 右移运算符，num >> 1,相当于num除以2，2的一次方

\>\>\>: 无符号右移，忽略符号位，空位都以0补齐。

如：x=5>>2;    5的二进制是：0101，右移两位即在左边补两个0，变成：0001，然后y=x>>>2;即1>>>2，即在0001的左边补两个0

### 3.位与运算符（&）

运算规则：两个数都转为二进制，然后从高位开始比较，如果两个数都为1则为1，否则为0。

比如：129&128，129转换成二进制就是10000001，128转换成二进制就是10000000。从高位开始比较得到，得到10000000，即128.

### 3.a++，表示先用后+，即先返回a的值，再执行a+1，++a表示先+后用，即先执行a+1，返回返回结果。

​	如果是用 System.out.print(a++)  ;则先输出原来 a 的值。

count = count++  原理是 temp = count； count = count+1 ； count = temp；

### 4.Math.ceil(d1) ：

​	如果参数小于0且大于-1.0，结果为 -0

   Math.floor(d1)：如果参数是 NaN、无穷、正 0、负 0，那么结果与参数相同，如果是 -0.0，那么其结果是 -0.0

### 5."|"与"||"的区别

用法：condition 1 | condition 2、condition 1 || condition 2

"|"是按位或：先判断条件1，不管条件1是否为true，都会执行条件2

"||"是逻辑或：先判断条件1，如果条件1为true，那么就不会执行条件2；若为false，则会执行条件2

### 6.参数传递

​	在将一个参数传入一个方法时，**本质上是将对象的地址以值的方式传递到形参中。**因此在方法中使指针引用其它对象，那么==这两个指针此时指向的是完全不同的对象，在一方改变其所指向对象的内容时对另一方没有影响。==

### 7.多进制运算

​	7.1变量a是一个64位有符号的整数，初始值用16进制表示为：0Xf000000000000000； 变量b是一个64位有符号的整数，初始值用16进制表示为：0x7FFFFFFFFFFFFFFF。 则a-b的结果用10进制表示为多少？

​	0x7FFFFFFFFFFFFFFF+1=0X8000000000000000，那么

a-b=0Xf000000000000000-0X8000000000000000+1

=0X7000000000000001

=16^15*7+16^0*1

=2^60*7+1

=2^60*(2^2+2^1+2^0)+1

=2^62+2^61+2^60+1

# 14.抽象类与接口

​	含有abstract修饰符的class即为抽象类，abstract类不能创建的实例对象。**含有abstract方法的类必须定义为abstract class，abstract class类中的方法不必是抽象的。**abstract class类中定义抽象方法必须在具体(Concrete)子类中实现，所以，不能有抽象构造方法或抽象静态方法。**如果的子类没有实现抽象父类中的所有抽象方法，那么子类也必须定义为abstract类型。**  子类不一定要全部实现父类的所有abstract方法，子类可以定义为abstract，然后交由它的子类实现。

​	接口（interface）可以说成是抽象类的一种特例，接口中的所有方法都必须是抽象的。**接口中的方法定义默认为public abstract类型，接口中的成员变量类型默认为public static final。**  

##### 抽象类和接口的语法区别：  

1.抽象类可以有构造方法，接口中不能有构造方法。  

2.抽象类中可以有普通成员变量，==接口中没有普通成员变量，接口中只有常量==  

3.抽象类中可以包含非抽象的普通方法，==接口中的所有方法必须都是抽象的，不能有非抽象的普通方法。==  

4.抽象类中的抽象方法的访问类型可以是public和protected，但接口中的抽象方法只能是public类型的，并且默认即为public abstract类型。  

==5.抽象类中可以包含静态方法，接口中不能包含静态方法== 

6.抽象类和接口中都可以包含静态成员变量，抽象类中的静态成员变量的访问类型可以任意，==但接口中定义的变量只能是public static final类型，并且默认即为public static final类型。==  

7.一个类可以实现多个接口，但只能继承一个抽象类。  

8.接口可以有default、static方法。

9.接口的字段只能是 static 和 final 类型的，而抽象类的字段没有这种限制。

##### 抽象类和接口在应用上的区别：  

​	接口更多的是在系统架构设计方法发挥作用，主要用于定义模块之间的通信契约。而抽象类在代码实现方面发挥作用，可以实现代码的重用，  

例如，模板方法设计模式是抽象类的一个典型应用，假设某个项目的所有Servlet类都要用相同的方式进行权限判断、记录访问日志和处理异常，那么就可以定义一个抽象的基类，让所有的Servlet都继承这个抽象基类，在抽象基类的service方法中完成权限判断、记录访问日志和处理异常的代码，在各个子类中只是完成各自的业务逻辑代码，伪代码如下： 

```Java
package com.lei; 
import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
public abstract class BaseServlet extends HttpServlet {
	/**
	* serialVersionUID属性概述
	*/
	private static final long serialVersionUID = 1L;
	public final void service(HttpServletRequest request,
		 HttpServletResponse response) throws IOException, ServletException {
		 // 记录访问日志
		 // 进行权限判断
		 if (true){// if条件里写的是“具有权限”
		 	try {
				 doService(request, response);
		 		} catch (IOException e) {
		 			// 记录异常信息
		 		}
		 	}		 
		 }
	protected abstract void doService(HttpServletRequest request,
		 HttpServletResponse response) throws IOException, ServletException}

//实现类如下(父类方法中间的某段代码不确定，留给子类干，就用模板方法设计模式。)： 
package com.lei;
import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
public class MyServlet extends BaseServlet{
	/**
	* serialVersionUID属性概述
	*/
	private static final long serialVersionUID = 1L;
		@Override
		protected void doService(HttpServletRequest request,
		 HttpServletResponse response) throws IOException, ServletException {
		 // TODO Auto-generated method stub 
		 }
}
```

# 15.会话跟踪

​	会话跟踪是一种灵活、轻便的机制，它使Web上的状态编程变为可能。

HTTP是一种无状态协议，每当用户发出请求时，服务器就会做出响应，客户端与服务器之间的联系是离散的、非连续的。当用户在同一网站的多个页面之间转换时，根本无法确定是否是同一个客户，会话跟踪技术就可以解决这个问题。当一个客户在多个页面间切换时，服务器会保存该用户的信息。

##### 有四种方法可以实现会话跟踪技术：URL重写、隐藏表单域、Cookie、Session。

1）.隐藏表单域：<input type="hidden">，非常适合步需要大量数据存储的会话应用。

2）.URL 重写:URL 可以在后面附加参数，和服务器的请求一起发送，这些参数为名字/值对。

3）.Cookie:一个 Cookie 是一个小的，已命名数据元素。服务器使用 SET-Cookie 头标将它作为 HTTP响应的一部分传送到客户端，客户端被请求保存 Cookie 值，在对同一服务器的后续请求使用一个Cookie 头标将之返回到服务器。与其它技术比较，Cookie 的一个优点是在浏览器会话结束后，甚至在客户端计算机重启后它仍可以保留其值

4）.Session：使用 setAttribute(String str,Object obj)方法将对象捆绑到一个会话

# 16.Java相关命令软件

**javac.exe是编译.java文件**

**java.exe是执行编译好的.class文件**

**javadoc.exe是生成Java说明文档**

**jdb.exe是Java调试器**

**javaprof.exe是剖析工具**

命令javac-d参数的用途是：

![image-20181226081617169](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226081617169-5783377-7514330.png)

![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/4550868_1517736571041_FB5C81ED3A220004B71069645F112867.png)

# 17.Socket

​	**==ServerSocket(int port) 是服务端绑定port端口，调accept()监听等待客户端连接，它返回一个连接队列中的一个socket。创建socket，监听TCP端口9000    ：new ServerSocket(9000);==**

**Socket(InetAddress address , int port)是创建==客户端==连接主机的socket流，其中InetAddress是用来记录主机的类，port指定端口。**

##### socket和servletSocket的交互如下图所示：

![image-20181226081644297](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226081644297-5783404-7514330.png)

服务器端：ServerSocket提供的实例 ServerSocket server = new ServerSocket(端口号) ，创建TCP连接对象

客户端：	   Socket提供的实例 Socket client = new Socket(IP地址，端口号)![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/8955099_1521189690989_0BB28C2A1ECCC47EC020E89E8A554BBC-7514330.png)

# 18.Java类加载器

![image-20181226081708829](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226081708829-5783428-7514330.png)

# ![image-20181226081729848](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226081729848-5783449-7514330.png)

# 19.基本类型

![image-20181226081748225](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226081748225-5783468-7514330.png)

##### 	基本类型又被称为原始数据类型。

基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成。

# 20.Unicode编码

​	因为Unicode兼容了大多数老版本的编码规范例如 ASCII

Unicode编码定义了这个世界上几乎所有字符（就是你眼睛看到的长那个样子的符号）的数字表示，也就是说Unicode为每个字符发了一张身份证，这张身份证上有一串唯一的数字ID确定了这个字符。

Unicode给这串数字ID起了个名字叫［码点］（Code Point），而很多人说的编码其实是想表达［Unicode转换格式］（即UTF，Unicode Transformation Formats）这是UTF-8/UTF-16/UTF-32的前缀来源

这个［Unicode转换格式］的存在是为了解决［码点］在计算机中的二进制表现形式而设计的

毕竟我们的机内表示涉及存储位宽，兼容古老编码格式，码点是数值过大的罕见字符等问题

［码点］经过映射后得到的二进制串的转换格式单位称之为［码元］（Code Unit）。也就是说如果有一种UTF的码点二进制表示有n字节，其码元为8位（1个byte），那么其拥有码元n个。每种UTF的码元都不同，其宽度被作为区分写在了UTF的后缀——这就是UTF-8/UTF-16/UTF-32的由来。UTF-8的码元是8位的，UTF-16的码元是16位的。大部分的编程语言采用16位的码元作为机内表示。这就是我们在各种语言中调用获取一个字符串中character的数量时会出现这么多混乱的原因。事实上我们调用这些方法时取得的不是字符个数，而是码元个数！一旦我们的字符串中包含了位于基本平面之外的码点，那么就会需要更多的码元来表示，这个时候就会出现测试时常见的困惑——为何return的字符数比实际字符数要多？所以实际写代码时要特别注意这个问题。

采取不同的映射方式可以得到不同格式的二进制串，但是他们背后所表示的［码点］永远是一致的就好像你换身份证但是身份证号不变一样。由于平时人们误把［转换格式］也称为［编码］，所以造成今天Unicode／UTF傻傻分不清楚且遣词造句运用混乱的悲桑局面。

Unicode 编码 发展到今天 扩展到了 21 位（从 U+0000 到 U+10FFFF ）。这一点很重要： Unicode 不是 16 位的编码， 它是 21 位的。这 21 位提供了 1,114,112 个码点，其中，只有大概 10% 正在使用，所以还有相当大的扩充空间。

编码空间被分成 17 个平面（plane），每个平面有 65,536 个字符（正好填充2个字节，16位）。0 号平面叫做「基本多文种平面」（BMP, Basic Multilingual Plane ），涵盖了几乎所有你能遇到的字符，除了 emoji（emoji位于1号平面 - -）。其它平面叫做补充平面，大多是空的。

总结一下各种编码格式的特质：

UTF-32

最清楚明了的一个 UTF 就是 UTF[-](http://en.wikipedia.org/wiki/UTF-32)32 ：它在每个码点上使用整 32 位。32 大于 21，因此每一个 UTF-32 值都可以直接表示对应的码点。尽管简单，UTF-32却几乎从来不在实际中使用，因为每个字符占用 4 字节太浪费空间了。

UTF-16 以及「代理对」（ Surrogate Pairs ）的概念

UTF[-](http://en.wikipedia.org/wiki/UTF-16)16要常见得多，它是根据有 16 位固定长度的码元（ code units ）定义的。UTF-16 本身是一种长度可变的编码。基本多文种平面（BMP）中的每一个码点都直接与一个码元相映射。鉴于 BMP 几乎囊括了所有常见字符，UTF-16 一般只需要 UTF-32 一半的空间。其它平面里很少使用的码点都是用两个 16 位的码元来编码的，这两个合起来表示一个码点的码元就叫做代理对（ surrogate pair ）。

UTF-8

UTF-8 使用一到四个字节来编码一个码点。从 0 到 127 的这些码点直接映射成 1 个字节（对于只包含这个范围字符的文本来说，这一点使得 UTF-8 和 ASCII 完全相同）。接下来的 1,920 个码点映射成 2 个字节，在 BMP 里所有剩下的码点需要 3 个字节。Unicode 的其他平面里的码点则需要 4 个字节。UTF-8 是基于 8 位的码元的，因此它并不需要关心字节顺序（不过仍有一些程序会在 UTF-8 文件里加上多余的 BOM）。

有效率的空间使用（仅就西方语言来讲），以及不需要操心字节顺序问题使得 UTF-8 成为存储和交流 Unicode 文本方面的最佳编码。它也已经是文件格式、网络协议以及 Web API 领域里事实上的标准了。

我们的JVM中保存码点是UTF16的转换格式，从char的位宽为16位也可以看得出来。由于绝大部分编码的码点位于基本平面，所以使用16位可以几乎表示所有常用字符。这就是许多语言编译器或运行时都使用UTF16的原因。英文在使用UTF16时也是2字节表示的。当我们想要使用其他平面的字符时，码元超过2个字节，就需要使用代理对在语言中的特定表示方式，譬如‘\U112233’之类的。

使用UTF8时，常用的Alphabet和Numeric都在前127字节，被有效率地用一个字节表示。而我们的中文由于排在1920个码点之后，所以使用3个字节表示，这方面就比UTF16转换格式耗费更多空间。

最后，不论使用哪种UTF转换格式，都是程序员自己可以选择的一种表达方式而已。我们可以通过Java方便的API进行自如转换。

# 21.重写和重载

![image-20181226081818928](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226081818928-5783499-7514330.png)

# 22.JVM

![image-20181226081840098](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226081840098-5783520-7514330.png)

-Xmx：最大堆大小

-Xms：初始堆大小

-Xmn:年轻代大小

-XXSurvivorRatio：年轻代中Eden区与Survivor区的大小比值

年轻代5120m， Eden：Survivor=3，Survivor区大小=1024m（Survivor区有两个，即将年轻代分为5份，每个Survivor区占一份），总大小为2048m。

-Xms初始堆大小即最小内存值为10240m

![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/5032673_1539139922699_59B2900AA03CB2182A51CDB520B535B6.png)

# 23.数据库管理系统

#### 数据库系统的特点：

1.数据结构化

2.数据的共享性高，[冗余度](https://baike.baidu.com/item/%E5%86%97%E4%BD%99%E5%BA%A6/5555254)低，易扩充

3.[数据独立性](https://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E7%8B%AC%E7%AB%8B%E6%80%A7)高

4.数据由[DBMS](https://baike.baidu.com/item/DBMS)统一管理和控制

##### 原子性-事务管理子系统。一致性-完整子系统。隔离性-并发控制子系统。持久性-恢复管理子系统

DBS（数据库系统）包含DB（数据库）和 DBNS（数据库管理系统）

##### 三级模式结构

​	**数据库系统由外模式、模式和内模式构成。**==外模式是数据库用户能够看见和使用的局部数据的逻辑结构和特征的描述，是数据库用户的数据视图；模式也称逻辑模式，是数据库中全体数据的逻辑结构和特征的描述，是所有用户的公共数据视图。内模式也称存储模式，是数据物理结构和存储方式的描述。==

**外模式/模式映像保证了数据库系统中的数据能具有较高的逻辑独立性；模式/内模式映像保证了数据库系统中的数据具有较高的物理独立性。**



##### 数据库系统特点：

​	数据库系统的存储模式如有改变，概念模式无需改动。因为概念模式是数据库中全体数据的逻辑结构和特征的描述，它隐藏了物理存储的细节，注重于描述实体、数据类型、联系、用户操作和约束。



​	活锁产生的原因：当一系列封锁不能按照其先后顺序执行时，就可能导致一些事务无限期等待某个封锁，从而导致活锁。 避免活锁的简单方法是采用==先来先服务==的策略。当多个事务请求封锁同一数据对象时，封锁子系统按请求封锁的先后次序对事务排队，数据对象上的锁一旦释放就批准申请队列中第一个事务获得锁。

#####  一、层次模型

​	层次模型是数据库系统中最早出现的数据模型，层次数据库系统采用层次模型作为数据的组织方式。它采用**树形结构来表示各类实体以及实体间的联系。**

1）优点：数据结构比较简单清晰，数据库的查询效率高，提供了良好的完整性支持。 
2）缺点：现实世界中很多联系是非层次性的，它不适用于结点之间具有多对多联系；查询子女结点必须通过双亲结点；由于结构严密，层次命令趋于程序化。

##### 二、网状模型

​	网状数据模型的典型代表是DBTG（CODASYL）系统。

1）优点：能够更为直接地描述现实世界，如一个结点可以有多个双亲，结点直接可以有多种联系；具有良好的性能，存取效率较高。 
2）缺点：结构比较复杂，随应用环境的扩大，数据库的结构就变得越来越复杂，不利于最终用户掌握；网状模型的DDL、DML复杂，并且要嵌入某一种高级语言（C、COBOL）中，用户不容易掌握和使用；由于记录之间的联系是通过存取路径实现的，应用程序在访问数据时必须选择适当的存取路径，因此用户必须了解系统结构的细节，加重了编写应用程序的负担。

##### 三、关系模型

​	关系模型是最重要的一种数据模型。

1）优点：建立在严格的数学概念的基础上；概念单一，无论实体还是实体之间的联系都是用关系来表示。对数据的检索和更新结构也是关系（也就是我们常说的表）；它的存取路径对用户透明，从而具有更高的独立性、更好的安全保密性，简化了程序员的工作个数据库开发建立的工作。 
2）缺点：存取路径的隐蔽导致查询效率不如格式化数据模型。

**一个关系都对应于一个二维表，表的每一行对应一个元组，一个二维表中，要求不同行之间元素不能完全相同**

##### 四、ER模型

​	实体-联系图(Entity-RelationDiagram)用来建立数据模型,在数据库系统概论中属于概念设计阶段，形成一个独立于机器，独立于DBMS的ER图模型。通常将它简称为ER图，相应地可把用ER图描绘的数据模型称为ER模型。

  ER模型被广泛应用在概年模型设计中，在软件开发的需求分析阶段，我们从用户的需求出发，对数据进行抽象，绘制ER图，这样一方面有效控制设计的复杂度，一方面可以更容易与用户交流，**可以反映现实世界中实体及实体间联系。**

ER图的要素：实体型，有向边和属性。

##### 范式

![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/5994168_1491996858898_A01A48B530942A78ACFC847D2B087CC7.png)

**第一范式（1NF）：属性不可拆分 或 无重复的列**:一个属性不允许再分成多个属性来建立列

**第二范式（2NF）：完全函数依赖**

​	部分函数依赖，就是多个属性决定另一个属性，但事实上，这多个属性是有冗余的。例如，（学号，班级）->姓名，事实上，只需要学号就能决定姓名，因此班级是冗余的，应该去掉。满足第二范式的数据库设计必须先满足第一范式。**因此第二范式的目标就是消除函数依赖关系中左边存在的冗余属性。**第二范式需要确保数据库表中的每一列都和主键相关，而不能只与主键的某一部分相关（主要针对联合主键而言）。也就是说在一个数据库表中，一个表中只能保存一种数据，不可以把多种数据保存在同一张数据库表中。

**第三范式（3NF）：消除传递依赖**

不依赖于其他非主属性（消除传递依赖）。满足第三范式的数据库必须先满足第二范式。也就是，数据库中的属性依赖仅能依赖于主属性，不存在于其他非主属性的关联。

例如，图书，图书室的关系。图书包括编号、出版商、页码等信息，图书室包括图书室编号、所存图书（外键）。其中，图书室的表中不应该存储任何图书的具体信息（例如，出版商。。），而只能通过主键图书编号来获得对应图书的信息。**第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关。**

**BC范式（BCNF）：**

（1）所有非主属性对每一个码都是完全函数依赖；

（2）所有的主属性对于每一个不包含它的码，也是完全函数依赖；

（3）没有任何属性完全函数依赖于非码的任意一个组合。

**R属于3NF，不一定属于BCNF，如果R属于BCNF，一定属于3NF。说明3NF>BCNF**

**第四范式（4NF）：**

对于每一个X->Y，X都能找到一个候选码（ 若关系中的某一属性组的值能唯一地表示一个元组,而其真子集不行,则称该属性组为候选码）。

https://www.cnblogs.com/linjiqin/archive/2012/04/01/2428695.html

​	**丢失修改、不可重复读、脏读等情况会导致数据不一致性。**

#####  E－R 模型向关系模型转换

​	一般的转换原则为：一个实体型转换为一个关系模式，关系的属性就是实体的属性，关系的码就是实体的码。

对于实体型间的联系有以下不同的情况：

**1.一个1：1联系可以转换成一个独立的关系模式，也可以与任意一端对应的关系模式合并。**如果转换为一个独立的关系模式，则与该联系相连的各实体的码以及联系本身的属性均转换为关系的属性，==每个实体的码均是该关系的候选码。==如果与某一端实体对应的关系模式合并，则需要在该关系模式的属性中加入另一个关系模式的码和联系本身的属性。

**2.一个1：n联系可以转换为一个独立的关系模式，也可以与n端对应的关系模式合并。**如果转换为一个独立的关系模式，则与该联系相连的各实体的码以及联系本身的属性均转换为关系的属性，而关系的码为n端实体的码。

**3.一个m：n联系转换为一个关系模式，与该联系相连的各实体的码以及联系本身的属性均转换为关系的属性，各实体的码组成关系的码或关系码的一部分。**

**4.三个或三个以上实体间的一个多元联系可以转换为一个关系模式。**与该多元联系相连的各实体的码以及联系本身的属性均转换为关系的属性，各实体的码组成关系的码或关系码的一部分。

**5.具有相同码的关系可以合并。**

==数据模型的三要素：数据结构，数据操作，数据的约束条件。==

# 23.数据库

### sql语句

增加列：**alter table** tableName **add** columnName **varchar** **(**30**)**

删除列：**alter table** tableName **drop column** columnName

modify:修改字段类型和长度的（即修改字段的属性）。
alter:修改表的数据结构（modify是alter的一种用法）。
update：修改数据内容的。

### 索引

```sql
全文索引创建：FULLTEXT KEY `title` (`title`,`content`)
selete与match()和against()一起使用：match()：指定被搜素的列；against()：指定要使用的搜索表达式，即：
selecte * from product where match(detail) against('rabbit');
```

# 24.String字符串

![image-20181226081859339](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226081859339-5783539.png)

##### String内部是用char[]数组实现的，不过结尾不用\0。char存储的unicode码，不仅可以存储ascII码，汉字也可以。

##### 串中任意个连续的字符组成的子序列称为该串的子串。

==一个字符常量表示为一个字符或一个转义序列，被一对ASCII**单引号**关闭。==

在 Java 8 中，String 内部使用 char 数组存储数据。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

**在 Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 `coder` 来标识使用了哪种编码。**

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte[] value;

    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
```

value 数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。

## 不可变的好处

**1. 可以缓存 hash 值**

​	因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

**2. String Pool 的需要**

​	如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/f76067a5-7d5f-4135-9549-8199c77d8f1c.jpg)

**3. 安全性**

​	String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 对象的那一方以为现在连接的是其它主机，而实际情况却不一定是。

**4. 线程安全**

​	==String 不可变性天生具备线程安全==，可以在多个线程中安全地使用。

# 25.文件

​	**==文件分为文本文件和二进制文件，计算机只认识二进制，所以实际上都是二进制的不同解释方式。==**文本文件是以不同编码格式显示的字符，例如Ascii、Unicode等，window中文本文件的后缀名有".txt",".log",各种编程语言的源码文件等；二进制文件就是用文本文档打开是看不懂乱码，只要能用文本打开的文件都可以算是文本文件，只是显示的结果不是你想要的，==二进制文件只有用特殊的应用才能读懂的文件，例如".png",".bmp"等==，计算机中大部分的文件还是二进制文件。

​	File类是对文件整体或者文件属性操作的类，例如创建文件、删除文件、查看文件是否存在等功能，不能操作文件内容；文件内容是用IO流操作的。

​	当输入过程中意外到达文件或流的末尾时，抛出EOFException异常,正常情况下读取到文件末尾时，返回一个特殊值表示文件读取完成，例如read()返回-1表示文件读取完成。

​	**一个文件中的字符要写到另一个文件中，首先需要建立文件字符输入流。**我们要把一个文件的字符写进另一个文件中，首先应该把当前文件的字符写进计算机内存中，所以是输入流。输入输出是相对于内存而言的。

# 26.ant和maven

![image-20181226081946241](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226081946241-5783586.png)

# 27.静态方法

##### 	非静态变量不能够被静态方法引用

##### 	被static修饰的变量称为静态变量，静态变量属于整个类，而局部变量属于方法，只在该方法内有效，所以static不能修饰局部变量

==只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字。==

# 28.jsp

![image-20181226081959871](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181226081959871-5783599.png)

# 29.构造函数

​	构造函数可以私有化，比如单例模式中的构造器

 	**子类的构造方法总是先调用父类的构造方法，如果子类的构造方法没有明显地指明使用父类的哪个构造方法，子类就调用父类不带参数的构造方法。如果父类没有无参的构造函数，子类则需要在自己的构造函数中显示的调用父类的构造函数。用super()调用带有参数的父类构造函数。**

# 30.处理Unicode字符

字符流是字节流根据字节流所要求的编码集解析获得的

可以理解为字符流=字节流+编码集

字符流有关的类都拥有操作编码集(unicode)的能力。

# 31.内部类

**1. 静态内部类：**

​    **1. 静态内部类本身可以访问外部的静态资源，包括静态私有资源。但是不能访问非静态资源，可以不依赖外部类实例而实例化。**

​	==静态内部类不能访问外部类的非静态的变量和方法。==

**2. 成员内部类：**

​    **1. 成员内部类本身可以访问外部的所有资源，但是自身不能定义静态资源，因为其实例化本身就还依赖着外部类。**

**3. 局部内部类：**

​    **1. 局部内部类就像一个局部方法，==不能被访问修饰符(四种修饰符)修饰，也不能被static修饰。==**

​    **2. 局部内部类只能访问所在代码块或者方法中被定义为final的局部变量。**

**4. 匿名内部类：**

​    **1. 没有类名的内部类，不能使用class，extends和implements，没有构造方法。**

​    **2. 多用于GUI中的事件处理。**

​    **3. 不能定义静态资源**

​    **4. 只能创建一个匿名内部类实例。**

​    **5. 一个匿名内部类一定是在new后面的，这个匿名类必须继承一个父类或者实现一个接口。**

​    **6. 匿名内部类是局部内部类的特殊形式，所以局部内部类的所有限制对匿名内部类也有效。**

# 32.JDK各个包的作用

java.awt： 包含构成抽象窗口工具集的多个类，用来构建和管理应用程序的图形用户界面

java.lang： 提供java编成语言的程序设计的基础类

java.io：　 包含提供多种输出输入功能的类，

java.net：　 包含执行与网络有关的类，如URL，SCOKET，SEVERSOCKET，

java.applet： 包含java小应用程序的类

java.util：　 包含一些实用性的类

# 33.内部类

#### 1.一个.java文件中定义多个类，注意一下几点：

(1) public权限类只能有一个（也可以一个都没有，但最多只有一个）；

(2)这个.java文件名只能是public 权限的类的类名；

(3)倘若这个文件中没有public 类，则它的.java文件的名字是随便的一个类名；

(4)当用javac命令生成编译这个.java 文件的时候，则会针对每一个类生成一个.class文件；

所以，一个Java源程序文件中定义几个类和接口，则编译该文件后生成几个以.class为后缀的字节码文件。

#### 2.外部类和内部类的访问权限

​	==public>defalut>protected>private==

（1）对于外部类而言，它也可以使用访问控制符修饰，但**外部类只能有两种访问控制级别： public 和默认。**因为外部类没有处于任何类的内部，也就没有其所在类的内部、所在类的子类两个范围，因此 private 和 protected 访问控制符对外部类没有意义。

**（2）内部类的上一级程序单元是外部类，它具有 4 个作用域：同一个类（ private ）、同一个包（ protected ）默认(default)和任何位置（ public）。**

（3）因为局部成员的作用域是所在方法，其他程序单元永远不可能访问另一个方法中的局部变量，所以所有的局部成员都不能使用访问控制修饰符修饰。局部内部类就相当于局部成员。

# 34.hashcode和equals方法

​	hashCode()方法和equals()方法的作用其实是一样的，在Java里都是用来对比两个对象是否相等一致。

**1.equals()相等的两个对象他们的hashCode()肯定相等，也就是用equals()对比是绝对可靠的。**

**2.hashCode()相等的两个对象他们的equal()不一定相等，也就是hashCode()不是绝对可靠的。**

​	每当需要对比的时候，首先用hashCode()去对比，如果hashCode()不一样，则表示这两个对象肯定不相等（也就是不必再用equal()去再对比了）,如果hashCode()相同，此时再对比他们的equals()，如果equals()也相同，则表示这两个对象是真的相同了，这样既能大大提高了效率也保证了对比的绝对正确性！

# 35.形式参数和实际参数

形式参数：就是函数定义时设定的参数。例如函数头 int min(int x,int y,int z) 中 x,y,z 就是形参，可被视为local variable。形参可以是对象,是对象的时候传递引用。对于形式参数只能用final修饰符，其它任何修饰符都会引起编译器错误 。但是用这个修饰符也有一定的限制，就是在方法中不能对参数做任何修改。 不过一般情况下，一个方法的形参不用final修饰。只有在特殊情况下，那就是：方法内部类。   

实际参数：调用函数时所使用的实际的参数，真正被传递的是实参

# 36.时间复杂度

**一、时间复杂度**  

（1）时间频度 

​	一个算法执行所耗费的时间，从理论上是不能算出来的，必须上机运行[测试](http://lib.csdn.net/base/softwaretest)才能知道。但我们不可能也没有必要对每个算法都上机测试，只需知道哪个算法花费的时间多，哪个算法花费的时间少就可以了。并且一个算法花费的时间与算法中语句的执行次数成正比例，哪个算法中语句执行次数多，它花费时间就多。一个算法中的语句执行次数称为语句频度或时间频度。记为T(n)。 

（2）时间复杂度 

​	在刚才提到的时间频度中，n称为问题的规模，当n不断变化时，时间频度T(n)也会不断变化。但有时我们想知道它变化时呈现什么规律。为此，我们引入时间复杂度概念。 

​	一般情况下，算法中基本操作重复执行的次数是问题规模n的某个函数，用T(n)表示，若有某个辅助函数f(n),使得当n趋近于无穷大时，T(n)/f(n)的极限值为不等于零的常数，则称f(n)是T(n)的同数量级函数。记作T(n)=O(f(n)),称O(f(n)) 为算法的渐进时间复杂度，简称时间复杂度。 

在各种不同算法中，若算法中语句执行次数为一个常数，则时间复杂度为O(1),另外，在时间频度不相同时，时间复杂度有可能相同，如T(n)=n2+3n+4与T(n)=4n2+2n+1它们的频度不同，但时间复杂度相同，都为O(n2)。 

按数量级递增排列，常见的时间复杂度有： 

**常数阶O(1),对数阶O(log2n),线性阶O(n),** 

**线性对数阶O(nlog2n),平方阶O(n2)，立方阶O(n3),...，** 

**k次方阶O(nk),指数阶O(2n)。**随着问题规模n的不断增大，上述时间复杂度不断增大，算法的执行效率越低。 

2、空间复杂度 

与时间复杂度类似，空间复杂度是指算法在计算机内执行时所需存储空间的度量。记作: 

S(n)=O(f(n)) 

我们一般所讨论的是除正常占用内存开销外的辅助存储单元规模

 **二、常见算法时间复杂度：**

O(1): 表示算法的运行时间为常量

O(n): 表示该算法是线性算法

O(㏒2n): 二分查找算法

O(n2): 对数组进行排序的各种简单算法，例如直接插入排序的算法。

O(n3): 做两个n阶矩阵的乘法运算

O(2n): 求具有n个元素集合的所有子集的算法

O(n!): 求具有N个元素的全排列的算法

优<---------------------------<劣

O(1)<O(㏒2n)<O(n)<O(n2)<O(2n)

时间复杂度按数量级递增排列依次为：常数阶O(1)、对数阶O(log2n)、线性阶O(n)、线性对数阶O(nlog2n)、平方阶O(n2)、立方阶O(n3)、……k次方阶O(nk)、指数阶O(2n)。 

**三、算** **法的时间复杂度（计算实例）**

定义：如果一个问题的规模是n，解这一问题的某一算法所需要的时间为T(n)，它是n的某一函数 T(n)称为这一算法的“时间复杂性”。

当输入量n逐渐加大时，时间复杂性的极限情形称为算法的“渐近时间复杂性”。

我们常用大O表示法表示时间复杂性，注意它是某一个算法的时间复杂性。大O表示只是说有上界，由定义如果f(n)=O(n)，那显然成立f(n)=O(n^2)，它给你一个上界，但并不是上确界，但人们在表示的时候一般都习惯表示前者。

此外，一个问题本身也有它的复杂性，如果某个算法的复杂性到达了这个问题复杂性的下界，那就称这样的算法是最佳算法。

“大O记法”：在这种描述中使用的基本参数是 n，即问题实例的规模，把复杂性或运行时间表达为n的函数。这里的“O”表示量级 (order)，比如说“二分检索是 O(logn)的”,也就是说它需要“通过logn量级的步骤去检索一个规模为n的数组”记法 O ( f(n) )表示当 n增大时，运行时间至多将以正比于 f(n)的速度增长。

这种渐进估计对算法的理论分析和大致比较是非常有价值的，但在实践中细节也可能造成差异。例如，一个低附加代价的O(n2)算法在n较小的情况下可能比一个高附加代价的 O(nlogn)算法运行得更快。当然，随着n足够大以后，具有较慢上升函数的算法必然工作得更快。

O(1)

Temp=i;i=j;j=temp;                    

以上三条单个语句的频度均为1，该程序段的执行时间是一个与问题规模n无关的常数。算法的时间复杂度为常数阶，记作T(n)=O(1)。如果算法的执行时 间不随着问题规模n的增加而增长，即使算法中有上千条语句，其执行时间也不过是一个较大的常数。此类算法的时间复杂度是O(1)。

O(n^2)

2.1. 交换i和j的内容

```
sum=0；                 （一次）

     for(i=1;i<=n;i++)       （n次 ）

        for(j=1;j<=n;j++) （n^2次 ）

         sum++；       （n^2次 ）

解：T(n)=2n^2+n+1 =O(n^2)

```

2.2.   

```
  for (i=1;i<n;i++)

    {

        y=y+1;         ①   

        for (j=0;j<=(2*n);j++)    

           x++;        ②      

    }         

解： 语句1的频度是n-1

          语句2的频度是(n-1)*(2n+1)=2n^2-n-1

          f(n)=2n^2-n-1+(n-1)=2n^2-2

          该程序的时间复杂度T(n)=O(n^2).         

O(n)                                                           
```

2.3.

```
 a=0;

    b=1;                      ①

    for (i=1;i<=n;i++) ②

    {  

       s=a+b;　　　　③

       b=a;　　　　　④  

       a=s;　　　　　⑤

    }

解： 语句1的频度：2,        

           语句2的频度： n,        

          语句3的频度： n-1,        

          语句4的频度：n-1,    

          语句5的频度：n-1,                                  

          T(n)=2+n+3(n-1)=4n-1=O(n).                                                                                               O(log2n )
```

2.4.

```
i=1;       ①

    while (i<=n)

       i=i*2; ②

解： 语句1的频度是1,  

          设语句2的频度是f(n),   则：2^f(n)<=n;f(n)<=log2n    

          取最大值f(n)= log2n,

          T(n)=O(log2n )

O(n^3)
```

2.5.

```
for(i=0;i<n;i++){  
      for(j=0;j<i;j++)  
       {
          for(k=0;k<j;k++)
             x=x+2;  
       }
    }
解：当i=m, j=k的时候,内层循环的次数为k当i=m时, j 可以取 0,1,...,m-1 , 所以这里最内循环共进行了0+1+...+m-1=(m-1)m/2次所以,i从0取到n, 则循环共进行了: 0+(1-1)*1/2+...+(n-1)n/2=n(n+1)(n-1)/6所以时间复杂度为O(n^3).
```

我们还应该区分算法的最坏情况的行为和期望行为。如快速排序的最 坏情况运行时间是 O(n^2)，但期望时间是 O(nlogn)。通过每次都仔细 地选择基准值，我们有可能把平方情况 (即O(n^2)情况)的概率减小到几乎等于 0。在实际中，精心实现的快速排序一般都能以 (O(nlogn)时间运行。

下面是一些常用的记法：

访问数组中的元素是常数时间操作，或说O(1)操作。一个算法如 果能在每个步骤去掉一半数据元素，如二分检索，通常它就取 O(logn)时间。用strcmp比较两个具有n个字符的串需要O(n)时间 。常规的矩阵乘算法是O(n^3)，因为算出每个元素都需要将n对 元素相乘并加到一起，所有元素的个数是n^2。

指数时间算法通常来源于需要求出所有可能结果。例如，n个元 素的集合共有2n个子集,所以要求出所有子集的算法将是O(2n)的 。指数算法一般说来是太复杂了，除非n的值非常小，因为，在 这个问题中增加一个元素就导致运行时间加倍。不幸的是，确实有许多问题 (如著名 的“巡回售货员问题” )，到目前为止找到的算法都是指数的。如果我们真的遇到这种情况， 通常应该用寻找近似最佳结果的算法替代之。

# 37.switch

​	switch结构中若没有break就会从第一个匹配上的case一直执行到整个结构结束。

​	==switch语句后的控制表达式只能是short、char、int、long整数类型和枚举类型，不能是float，double和boolean类型。**String类型是java7开始支持。**==

# 38.集合

![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/2193220_1487038875892_072774B6B658B3603E1AA7198722775C.png)

# 39.值类型和引用类型

​	值类型的变量赋值只是进行数据复制，创建一个同值的新对象，而引用类型变量赋值，仅仅是把对象的引用的指针赋值给变量，使它们共用一个内存地址。

# 40.字节、位及各类型所占字节数

![image-20190121104754130](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190121104754130.png)

​	**一个字节(byte)等于8位(bit)，数组要先固定大小，才可以说某个元素占了多少字节，所占的字节数要根据声明的元素类型乘几个元素去算，比如：arr[16],那么arr[0]-arr[7]占32字节，因为一个int占4个字节，总共有8个元素，所以是32。**

# 41.null

## **一、null是代表不确定的对象**

Java中，null是一个关键字，用来标识一个不确定的对象。**因此可以将null赋给引用类型变量，但不可以将null赋给基本类型变量。**

比如：int a = null;是错误的。Ojbect o = null是正确的。

Java中，变量的适用都遵循一个原则，先定义，并且初始化后，才可以使用。我们不能int a后，不给a指定值，就去打印a的值。这条对于引用类型变量也是适用的。

有时候，我们定义一个引用类型变量，在刚开始的时候，无法给出一个确定的值，但是不指定值，程序可能会在try语句块中初始化值。这时候，我们下面使用变量的时候就会报错。这时候，可以先给变量指定一个null值，问题就解决了。例如：

```java
Connection conn = null;
try {
	conn = DriverManager.getConnection("url", "user", "password");
} catch (SQLException e) {
	e.printStackTrace();
}
String catalog = conn.getCatalog();
```

如果刚开始的时候不指定conn = null，则最后一句就会报错。

## **二、null本身不是对象，也不是Objcet的实例**

​	null本身虽然能代表一个不确定的对象，但就null本身来说，它不是对象，也不知道什么类型，也不是java.lang.Object的实例。null instanceof java.lang.Object会返回false

## **三、Java默认给变量赋值**

​	在定义变量的时候，如果定义后没有给变量赋值，则Java在运行时会自动给变量赋值。**赋值原则是整数类型int、byte、short、long的自动赋值为0，带小数点的float、double自动赋值为0.0，boolean的自动赋值为false，其他==各供引用类型变量自动赋值为null==。**

## **四、容器类型与null**

List：允许重复元素，可以加入任意多个null。

Set：不允许重复元素，最多可以加入一个null。

Map：Map的key最多可以加入一个null，value字段没有限制。

数组：基本类型数组，定义后，如果不给定初始值，则java运行时会自动给定值。引用类型数组，不给定初始值，则所有的元素值为null。

## **五、null的其他作用**

1、判断一个引用类型数据是否null。 用==来判断。

2、**释放内存，让一个非null的引用类型变量指向null。这样这个对象就不再被任何对象应用了。等待JVM垃圾回收机制去回收。**

![image-20190122100211403](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190122100211403.png)

# 42.final

1.final修饰变量，则等同于常量

2.final修饰方法中的参数，称为最终参数。

3.final修饰类，则类不能被继承

4.final修饰方法，则方法不能被重写，但可以被重载

# 43.ArrayList

## 1.构造方法

（1）ArrayList()构造一个初始容量为 10 的空列表。

（2）ArrayList(Collection<? extends E> c)构造一个包含指定 collection 的元素的列表，这些元素是按照该 collection 的迭代器返回它们的顺序排列的。

（3）ArrayList(int initialCapacity)构造一个具有指定初始容量的空列表。

如：ArrayList list = new ArrayList(20);	表示构造了初始容量为20的ArrayList

# 44.Servlet

## 1.Servlet过滤器的配置

​	第一部分是过滤器在Web应用中的定义，由<filter>元素表示，包括<filter-name>和<filter-class>两个必需的子元素
​	第二部分是过滤器映射的定义，由<filter-mapping>元素表示,可以将一个过滤器映射到一个或者多个Servlet或JSP文件，也可以采用url-pattern将过滤器映射到任意特征的URL。

![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/6316247_1469628859864_A8BB53E66CC9A072C0448DDDBDF4C3B2.png)

## 2.servlet是什么

​	Servlet是JavaEE规范的一种，主要是为了扩展Java作为Web服务的功能，统一接口。由其他内部厂商如tomcat，jetty内部实现web的功能。如一个http请求到来：
 容器将请求封装为servlet中的HttpServletRequest对象，调用init（），service（）等方法输出response,由容器包装为httpresponse返回给客户端的过程。

![image-20190201104605382](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190201104605382.png)

## 3、servlet的生命周期

加载和实例化：Servlet容器负责加载和实例化Servlet。当Servlet容器启动时，或者在容器检测到需要这个Servlet来响应第一个请求时，创建Servlet实例。
初始化：init方法是在servlet实例创建时调用的方法，用于创建或打开任何与servlet相的资源和初始 化servlet的状态，Servlet规范保证调用init方法前不会处理任何请求 

请求处理：service方法是servlet真正处理客户端传过来的请求的方法，由web容器调用， 根据HTTP请求方法（GET、POST等），将请求分发到doGet、doPost等方法 

服务终止：destory方法是在servlet实例被销毁时由web容器调用。Servlet规范确保在destroy方法调用之 前所有请求的处理均完成，需要覆盖destroy方法的情况：释放任何在init方法中 打开的与servlet相关的资源存储servlet的状态

https://www.cnblogs.com/lgk8023/p/6427977.html

==servlet在多线程下其本身并不是线程安全的。==

# 45.数组

1.在java 中，声明一个数组时，不能直接限定数组长度，只有在创建实例化对象时，才能对给定数组长度。因为数组是**一个引用类型变量** ，因此**使用它定义一个变量时，仅仅定义了一个变量** **，这个引用变量还未指向任何有效的内存** **，因此定义数组不能指定数组的长度**。只有在实例化时才可以指定长度，如：String[] arr=new String[50];

2.在定义 int a\[3]\[4][2]; 后，第 20 个元素是

​	首先，总共有三层(0~2)，然后每层都是一个二维数组\[4\][2]，所以总共有3*4\*2=24个元素，所以20个元素就是第三层开始，然后2--16=4，差4个元素，所以是a[2]\[1][1]，后面二维数组是[1]\[1]表示2\*2=4

# 46.成员变量与局部变量

​	成员变量有初始值，而局部变量(方法里面)没有初始值得。所以如果方法里面声明的变量没有赋值的话会报错，编译通不过。

# 47.进制计算

​	可以使用方程式计算，比如：13*14=204，假设是X进制，即：(3+X\*1)\*(4+X\*1)=204，得X=8/X=-1

# 48.数据结构

​	**线性表的顺序存储结构是一种随机存取的存储结构。**每一个数据元素的存储位置都和线性表的起始位置相差一个和数据元素在线性表中的位序成正比的常数。由此，只要确定了存储线性表的起始位置，线性表中任一元素都可随机存取，所以线性表的顺序存储结构是一种随机存取的存储结构。

​	**假设线性表的每个数据元素需占用K个存储单元，并以元素所占的第一个存储单元的地址作为数据元素的存储地址。**则线性表中序号为i的数据元素的存储地址LOC(ai)与序号为i+1的数据元素的存储地址LOC(ai+1)之间的关系为： 
　　LOC(ai+1)=LOC(ai)+K 
　　通常来说，线性表的i号元素的存储地址为： 
　　LOC(ai+1)=LOC(a0)+i×K 
　　其中LOC(a0)为0号元素的存储地址，通常称为线性表的起始地址。 

　　**线性表的这种机内表示称作线性表的顺序存储。**它的特点是，以数据元素在机内存储地址相邻来表示线性表中数据元素之间的逻辑关机。每一个数据元素的存储地址都和线性表的起始地址相差一个与数据据元素在线性表中的序号成正比的常数。由此，只要确定了线性表的起始地址，线性表中的任何一个数据元素都可以随机存取，因此线性表的顺序存储结构时一种随机的存储结构。

![çº¿æ§è¡¨çé¡ºåºå­å¨](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/SouthEast.png)

## 48.1 数组与链表

1：数组内存空间比链表少

2：数组支持随机访问，链表不具有随机访问的特性

3：插入和删除是链表优于数组，数组需要移动被删除或者插入位置之后的元素

​	静态链表是用数组存储节点数据，模拟链表的实现，但是没有用到指针。每个数组节点包括两部分：data域和cursor（游标）域。data存储数据，cursor指明下个元素在数组中的下标。

（1）存取第i个元素时，需要从头遍历到i-1和元素，由第i-1个节点的cursor，才能知道第i个元素存储的位置，因此和i是相关的。

（2）使用数组对元素进行存储，在定义时大小已经确定。

（3）插入和删除操作无需移动元素，只需要修改cursor游标的值即可，就像修改动态链表中的指针一样。

4.广义表有如下三个特性：
1.层次性：广义表的元素可以是子表，而子表的元素还可以是子表，由此，广义表是一个多层次的结构；
2.共享性：广义表可为其他表所共享。
3.递归表：广义表可以是其自身的一个子表。

# 49.泛型与JVM

① 虚拟机中没有泛型，只有普通类和方法。 

② 在编译阶段，所有泛型类的类型参数都会被Object或者它们的限定边界来替换。(类型擦除) 

③ 在继承泛型类型的时候，桥方法的合成是为了避免类型变量擦除所带来的多态灾难。 无论我们如何定义一个泛型类型，相应的都会有一个原始类型被自动提供。原始类型的名字就是擦除类型参数的泛型类型的名字。

​	**创建泛型对象时请指明类型，让编译器尽早的做参数检查**

# 50.redirect和forward差别：

转发过程：

​	客户浏览器发送http请求----》web服务器接受此请求--》调用内部的一个方法在容器内部完成请求处理和转发动作----》将目标资源 发送给客户；在这里，转发的路径必须是同一个web容器下的url，其不能转向到其他的web路径上去，中间传递的是自己的容器内的request。在客 户浏览器路径栏显示的仍然是其第一次访问的路径，也就是说客户是感觉不到服务器做了转发的。转发行为是浏览器只做了一次访问请求。 

重定向过程：

​	客户浏览器发送http请求----》web服务器接受后发送**302**状态码响应及对应新的location给客户浏览器--》**客户浏览器发现 是302响应**，则自动再发送一个新的http请求，请求url是新的location地址----》服务器根据此请求寻找资源并发送给客户。在这里 location可以重定向到任意URL，既然是浏览器重新发出了请求，则就没有什么request传递的概念了。在客户浏览器路径栏显示的是其重定向的 路径，客户可以观察到地址的变化的。重定向行为是浏览器做了至少两次的访问请求的。 

# 51.缓存池

new Integer(123) 与 Integer.valueOf(123) 的区别在于：

- new Integer(123) 每次都会新建一个对象；
- Integer.valueOf(123) 会使用缓存池中的对象，多次调用会取得同一个对象的引用。

valueOf() 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓存池的内容。

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

在 Java 8 中，Integer 缓存池的大小默认为 -128~127。

```java
static final int low = -128;
static final int high;
static final Integer cache[];

static {
    // high value may be configured by property
    int h = 127;
    String integerCacheHighPropValue =
        sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
    if (integerCacheHighPropValue != null) {
        try {
            int i = parseInt(integerCacheHighPropValue);
            i = Math.max(i, 127);
            // Maximum array size is Integer.MAX_VALUE
            h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
        } catch( NumberFormatException nfe) {
            // If the property cannot be parsed into an int, ignore it.
        }
    }
    high = h;

    cache = new Integer[(high - low) + 1];
    int j = low;
    for(int k = 0; k < cache.length; k++)
        cache[k] = new Integer(j++);

    // range [-128, 127] must be interned (JLS7 5.1.7)
    assert IntegerCache.high >= 127;
}
```

编译器会在自动装箱过程调用 valueOf() 方法，因此多个 Integer 实例使用自动装箱来创建并且值相同，那么就会引用相同的对象。

基本类型对应的缓冲池如下：

- boolean values true and false
- all byte values
- short values between -128 and 127
- int values between -128 and 127
- char in the range \u0000 to \u007F

在使用这些基本类型对应的包装类型时，就可以直接使用缓冲池中的对象。

# 52.字符串常量池

​	字符串常量池（String Pool）保存着所有字符串字面量（literal strings），**这些字面量在编译时期就确定。**不仅如此，还可以==使用 String 的 intern() 方法在运行过程中将字符串添加到 String Pool 中。==

​	当一个字符串调用 intern() 方法时，**如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；**否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

例子：

```java
//创建两个字符串对象s1,s2
String s1 = new String("aaa");			
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
//s3,s4通过 s1.intern() 方法取得一个字符串引用
String s3 = s1.intern();
String s4 = s1.intern();
System.out.println(s3 == s4);           // true
```

​	**intern() 首先把 s1 引用的字符串放到 String Pool 中，然后返回这个字符串引用。因此 s3 和 s4 引用的是同一个字符串。**

​	在 Java 7 之前，**String Pool 被放在运行时常量池中，它属于永久代。而在 Java 7，String Pool 被移到堆中。**这是因为永久代的空间有限，在大量使用字符串的场景下会导致 OutOfMemoryError 错误。

以下是String 构造函数的源码，可以看到，**在将一个字符串对象作为另一个字符串对象的构造函数参数时，并不会完全复制 value 数组内容，而是都会指向同一个 value 数组。**

```java
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```



























































































