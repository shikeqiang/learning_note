# Java集合

![img](/Users/jack/Desktop/md/images/java-coll.png)

![img](/Users/jack/Desktop/md/images/jpeg.jpeg)

## 一、集合接口

集合框架定义了一些接口。本节提供了每个接口的概述：

| 序号 | 接口描述                                                     |
| ---- | ------------------------------------------------------------ |
| 1    | Collection 接口        Collection 是最基本的集合接口，一个 Collection 代表一组 Object，即 Collection 的元素, Java不提供直接继承自Collection的类，只提供继承于的子接口(如List和set)。Collection 接口存储一组不唯一，无序的对象。 |
| 2    | List 接口                  List接口是一个有序的 Collection，使用此接口能够精确的控制每个元素插入的位置，能够通过索引(元素在List中位置，类似于数组的下标)来访问List中的元素，第一个元素的索引为 0，而且允许有相同的元素。List 接口存储一组不唯一，有序（插入顺序）的对象。 |
| 3    | Set                          Set 具有与 Collection 完全一样的接口，只是行为上不同，Set 不保存重复的元素。Set 接口存储一组唯一，无序的对象。 |
| 4    | SortedSet  继承于Set保存有序的集合。                         |
| 5    | Map                      Map 接口存储一组键值对象，提供key（键）到value（值）的映射。 |
| 6    | Map.Entry  描述在一个Map中的一个元素（键/值对）。是一个Map的内部类。 |
| 7    | SortedMap 继承于 Map，使 Key 保持在升序排列。                |
| 8    | Enumeration 这是一个传统的接口和定义的方法，通过它可以枚举（一次获得一个）对象集合中的元素。这个传统接口已被迭代器取代。 |

### Set和List的区别

- 1. Set 接口实例存储的是无序的，不重复的数据。List 接口实例存储的是有序的，可以重复的元素。
- 2. Set检索效率低下，删除和插入效率高，插入和删除不会引起元素位置改变 **<实现类有HashSet,TreeSet>**。
- 3. List和数组类似，可以动态增长，根据实际存储的数据的长度自动增长List的长度。查找元素效率高，插入删除效率低，因为会引起其他元素位置改变 **<实现类有ArrayList,LinkedList,Vector>** 。

### 集合实现类（集合类）

​	**Java提供了一套实现了Collection接口的标准集合类。其中一些是具体类，这些类可以直接拿来使用，而另外一些是抽象类，提供了接口的部分实现。**

标准集合类汇总于下表：

| 序号 | 类描述                                                       |
| ---- | ------------------------------------------------------------ |
| 1    | **AbstractCollection**         实现了大部分的集合接口。      |
| 2    | **AbstractList**                     继承于AbstractCollection 并且实现了大部分List接口。 |
| 3    | **AbstractSequentialList**  继承于 AbstractList ，提供了对数据元素的链式访问而不是随机访问。 |
| 4    | LinkedList 该类实现了List接口，允许有null（空）元素。主要用于**创建链表数据结构**，该类没有同步方法，如果多个线程同时访问一个List，则必须自己实现访问同步，解决方法就是在创建List时候构造一个同步的List。例如：`Listlist=Collections.synchronizedList(newLinkedList(...));`LinkedList 查找效率低。 |
| 5    | ArrayList 该类也是实现了List的接口，实现了可变大小的数组，随机访问和遍历元素时，提供更好的性能。该类也是非同步的,在多线程的情况下不要使用。**ArrayList 扩容时，增长当前长度的50%，插入删除效率低。(int newCapacity = oldCapacity + (oldCapacity >> 1);右移一位，相当于除以2)** |
| 6    | **AbstractSet**            继承于AbstractCollection 并且实现了大部分Set接口。 |
| 7    | HashSet         该类实现了Set接口，不允许出现重复元素，不保证集合中元素的顺序，允许包含值为null的元素，但最多只能一个。 |
| 8    | LinkedHashSet 具有可预知迭代顺序的 `Set` 接口的哈希表和链接列表实现。 |
| 9    | TreeSet 该类实现了Set接口，可以实现排序等功能。              |
| 10   | **AbstractMap**  实现了大部分的Map接口。                     |
| 11   | HashMap  HashMap 是一个散列表，它存储的内容是键值对(key-value)映射。 该类实现了Map接口，根据键的HashCode值存储数据，具有很快的访问速度，最多允许一条记录的键为null，不支持线程同步。 |
| 12   | TreeMap  继承了AbstractMap，并且使用一颗树。                 |
| 13   | WeakHashMap  继承AbstractMap类，使用弱密钥的哈希表。         |
| 14   | LinkedHashMap  继承于HashMap，使用元素的自然顺序对元素进行排序. |
| 15   | IdentityHashMap  继承AbstractMap类，比较文档时使用引用相等。 |

在前面的教程中已经讨论通过java.util包中定义的类，如下所示：

| 序号 | 类描述                                                       |
| ---- | ------------------------------------------------------------ |
| 1    | Vector  该类和ArrayList非常相似，但是该类是同步的，可以用在多线程的情况，该类允许设置默认的增长长度，默认扩容方式为原来的2倍。 |
| 2    | Stack  栈是Vector的一个子类，它实现了一个标准的后进先出的栈。 |
| 3    | Dictionary  Dictionary 类是一个抽象类，用来存储键/值对，作用和Map类相似。 |
| 4    | Hashtable  Hashtable 是 Dictionary(字典) 类的子类，位于 java.util 包中。 |
| 5    | Properties  Properties 继承于 Hashtable，表示一个持久的属性集，属性列表中每个键及其对应值都是一个字符串。(==线程安全==) |
| 6    | BitSet   **一个Bitset类创建一种特殊类型的数组来保存位值。BitSet中数组大小会随需要增加。占用内存较小** |

## **集合框架底层数据结构总结**

### 1）List

- ArrayList ：Object 数组。
- Vector ：Object 数组。
- LinkedList ：**双向链表(JDK6 之前为循环链表，JDK7 取消了循环，内部类Node中有next和prev两个指针)。**

### 2）Map

- HashMap ：
  - JDK8 之前，HashMap 由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的。
  - JDK8 以后，在解决哈希冲突时有了较大的变化，**当链表长度大于阈值（默认为 8 ）时，将链表转化为红黑树，以减少搜索时间。**

  > HashMap 最多只允许一条记录的键为 null，允许多条记录的值为 null。 

- LinkedHashMap ：LinkedHashMap 继承自 HashMap，所以**它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。**另外，LinkedHashMap 在上面结构的基础上，==增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。==同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。详细可以查看：[《LinkedHashMap 源码详细分析（JDK1.8）》](https://www.imooc.com/article/22931) 。

  > ==LinkedHashSet 底层使用 LinkedHashMap 来保存所有元素，它继承与 HashSet，其所有的方法操作上又与 HashSet 相同==，因此 LinkedHashSet 的实现上非常简单，只提供了四个构造方法，并通过传递一个标识参数，调用父类的构造器，底层构造一个 LinkedHashMap 来实现，在相关操作上与父类 HashSet 的操作相同，直接调用父类 HashSet 的方法即可。 

- Hashtable ：数组+链表组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的。

- TreeMap ：红黑树（自平衡的排序二叉树）。

  > **TreeMap 实现 SortedMap 接口，能够把它保存的记录根据键排序，默认是按键值的升序排序， 也可以指定排序的比较器，当用 Iterator 遍历 TreeMap 时，得到的记录是排过序的。** 在使用 TreeMap 时，key 必须实现 Comparable 接口或者在构造 TreeMap 传入自定义的 Comparator，否则会在运行时抛出 java.lang.ClassCastException 类型的异常。 
  >
  > 如果使用排序的映射，建议使用 TreeMap。

### 3）Set

- HashSet ：无序，唯一，基于 HashMap 实现的，底层采用 HashMap 来保存元素。

  > HashSet 首先判断两个元素的哈希值，如果哈希值一样，接着会比较 equals 方法 如果 equls 结果为true ，HashSet 就视为同一个元素。如果 equals 为 false 就不是同一个元素。

- LinkedHashSet ：LinkedHashSet 继承自 HashSet，并且其内部是通过 LinkedHashMap 来实现的。有点类似于我们之前说的LinkedHashMap 其内部是基于 HashMap 实现一样，不过还是有一点点区别的。

- TreeSet ：有序，唯一，红黑树(自平衡的排序二叉树)。

  > 1. TreeSet()是使用二叉树的原理对新 add()的对象按照指定的顺序排序(升序、降序)，每增 加一个对象都会进行排序，将对象插入的二叉树指定的位置。 
  >
  > 2. Integer 和 String 对象都可以进行默认的 TreeSet 排序，而自定义类的对象是不可以的，**自己定义的类必须实现 Comparable 接口，并且覆写相应的 compareTo()函数，才可以正常使 用。** 
  >
  > 3. 在覆写 compare()函数时，要返回相应的值才能使 TreeSet 按照一定的规则来排序 
  >
  > 4. 比较此对象与指定对象的顺序。如果该对象小于、等于或大于指定对象，则分别返回负整 
  >
  >    数、零或正整数。

### **快速失败（fail-fast）和安全失败（fail-safe）的区别**

差别在于 ConcurrentModification 异常：

- 快速失败：当你在迭代一个集合的时候，如果有另一个线程正在修改你正在访问的那个集合时，就会抛出一个 ConcurrentModification 异常。 ==在 `java.util` 包下的都是快速失败==。
- 安全失败：你在迭代的时候会去底层集合做一个拷贝，所以你在修改上层集合的时候是不会受影响的，不会抛出 ConcurrentModification 异常。==在 `java.util.concurrent` 包下的全是安全失败的。==

### Comparable 和 Comparator 的区别

- Comparable 接口，在 `java.lang` 包下，用于当前对象和其它对象的比较，所以它有一个 `compareTo(Object obj)` 方法用来排序，该方法只有一个参数。
- Comparator 接口，在 `java.util` 包下，用于传入的两个对象的比较，所以它有一个 `compare(Object obj1, Object obj2)` 方法用来排序，该方法有两个参数。可以自定义排序

详细的，可以看看 [《Java 自定义比较器》](https://blog.csdn.net/striveb/article/details/88086451) 文章，重点是如何自己实现 Comparable 和 Comparator 的方法。



# 区别

## List 和 Set 区别？

List，Set 都是继承自 Collection 接口。

- List 特点：元素有放入顺序，元素可重复。
- Set 特点：元素无放入顺序，元素不可重复，重复元素会覆盖掉。

> 注意：元素虽然无放入顺序，但是元素在 Set 中的位置是有该元素的 hashcode 决定的，其位置其实是固定的。
>
> 另外 List 支持 `for` 循环，也就是通过下标来遍历，也可以用迭代器，但是 Set 只能用迭代，因为他无序，无法用下标来取得想要的值。

Set 和 List 对比：

- Set：检索元素效率高，删除和插入效率低，插入和删除不会引起元素位置改变。
- List：和数组类似，List 可以动态增长，查找元素效率低，插入删除元素效率，因为可能会引起其他元素位置改变。

## List 和 Map 区别？

- List 是对象集合，允许对象重复。
- Map 是键值对的集合，不允许 key 重复。

## Array 和 ArrayList 有何区别？什么时候更适合用 Array？

- Array 可以容纳基本类型和对象，而 ArrayList 只能容纳对象。
- Array 是指定大小的，而 ArrayList 大小是固定的，可自动扩容。
- Array 没有提供 ArrayList 那么多功能，比如 addAll、removeAll 和 iterator 等。

尽管 ArrayList 明显是更好的选择，但也有些时候 Array 比较好用，比如下面的三种情况。

- 1、如果列表的大小已经指定，大部分情况下是存储和遍历它们
- 2、对于遍历基本数据类型，尽管 Collections 使用自动装箱来减轻编码任务，在指定大小的基本类型的列表上工作也会变得很慢。
- 3、如果你要使用多维数组，使用 `[][]` 比 List 会方便。

## ArrayList 与 LinkedList 区别？

🦅 **ArrayList**

- 优点：ArrayList 是实现了基于动态数组的数据结构，因为地址连续，一旦数据存储好了，查询操作效率会比较高（在内存里是连着放的）。
- 缺点：因为地址连续，ArrayList 要移动数据，所以插入和删除操作效率比较低。即删除或者插入元素的时候，需要移动其他位置的元素，即其他元素在数组中的索引需要改变。

🦅 **LinkedList**

- 优点：LinkedList 基于链表的数据结构，地址是任意的，所以在开辟内存空间的时候不需要等一个连续的地址。对于新增和删除操作 add 和 remove ，LinedList 比较占优势。LinkedList 适用于要头尾操作或插入指定位置的场景。
- 缺点：因为 LinkedList 要移动指针，所以查询操作性能比较低。

🦅 **适用场景分析**：

- 当需要对数据进行对随机访问的情况下，选用 ArrayList 。

- 当需要对数据进行多次增加删除修改时，采用 LinkedList 。

  > 如果容量固定，并且只会添加到尾部，不会引起扩容，优先采用 ArrayList 。

- 当然，绝大数业务的场景下，使用 ArrayList 就够了。主要是，注意好避免 ArrayList 的扩容，以及非顺序的插入。

🦅 **ArrayList 是如何扩容的？**

直接看 [《ArrayList 动态扩容详解》](https://www.cnblogs.com/kuoAT/p/6771653.html) 文章，很详细。主要结论如下：

- 如果通过无参构造的话，初始数组容量为 0 ，当真正对数组进行添加时，才真正分配容量。每次按照 **1.5** 倍（位运算）的比率通过 copeOf 的方式扩容。
- 在 JKD6 中实现是，如果通过无参构造的话，初始数组容量为10，每次通过 copeOf 的方式扩容后容量为原来的 **1.5** 倍。

> 重点是 1.5 倍扩容，这是和 HashMap 2 倍扩容不同的地方。

🦅 **ArrayList 集合加入 1 万条数据，应该怎么提高效率？**

​	ArrayList 的默认初始容量为 10 ，要插入大量数据的时候需要不断扩容，而扩容是非常影响性能的。**因此，现在明确了 10 万条数据了，我们可以直接在初始化的时候就设置 ArrayList 的容量！**

## ArrayList 与 Vector 区别？

ArrayList 和 Vector 都是用数组实现的，主要有这么三个区别：

- 1、Vector 是多线程安全的，线程安全就是说多线程访问同一代码，不会产生不确定的结果，而 ArrayList 不是。这个可以从源码中看出，Vector 类中的方法很多有 `synchronized` 进行修饰，这样就导致了 Vector 在效率上无法与 ArrayList 相比。

  > Vector 是一种老的动态数组，是线程同步的，效率很低，一般不赞成使用。

- 2、两个都是采用的线性连续空间存储元素，但是当空间不足的时候，两个类的增加方式是不同。

- **3、Vector 可以设置增长因子，而 ArrayList 不可以。**

适用场景分析：

- 1、Vector 是线程同步的，所以它也是线程安全的，而 ArrayList 是线程无需同步的，是不安全的。如果不考虑到线程的安全因素，一般用 ArrayList 效率比较高。

  > 实际场景下，如果需要多线程访问安全的数组，使用 CopyOnWriteArrayList 。

- 2、如果集合中的元素的数目大于目前集合数组的长度时，在集合中使用数据量比较大的数据，用 Vector 有一定的优势。

  > 这种情况下，使用 LinkedList 更合适。

## HashMap 和 Hashtable 的区别？

> Hashtable 是在 Java 1.0 的时候创建的，而集合的统一规范命名是在后来的 Java2.0 开始约定的，而当时其他一部分集合类的发布构成了新的集合框架。

- Hashtable 继承 Dictionary ，HashMap 继承的是 Java2 出现的 Map 接口。
- 2、HashMap 去掉了 Hashtable 的 contains 方法，但是加上了 containsValue 和 containsKey 方法。
- 3、HashMap 允许空键值，而 Hashtable 不允许。
- 【重点】**4、HashTable 是同步的，而 HashMap 是非同步的，效率上比 HashTable 要高。也因此，HashMap 更适合于单线程环境，而 HashTable 适合于多线程环境。**
- 5、HashMap 的迭代器（Iterator）是 fail-fast 迭代器，HashTable的 enumerator 迭代器不是 fail-fast 的。
- 6、HashTable 中数组默认大小是 11 ，扩容方法是 `old * 2 + 1` ，HashMap 默认大小是 16 ，扩容每次为 2 的指数大小。

一般现在不建议用 HashTable 。主要原因是两点：

- 一是，HashTable 是遗留类，内部实现很多没优化和冗余。
- 二是，即使在多线程环境下，现在也有同步的 ConcurrentHashMap 替代，没有必要因为是多线程而用 Hashtable 。

🦅 **Hashtable 的 size() 方法中明明只有一条语句 "return count;" ，为什么还要做同步？**

​	同一时间只能有一条线程执行固定类的同步方法，但是对于类的非同步方法，可以多条线程同时访问。所以，这样就有问题了，可能线程 A 在执行 Hashtable 的 put 方法添加数据，线程 B 则可以正常调用 `size()` 方法读取 Hashtable 中当前元素的个数，那读取到的值可能不是最新的，可能线程 A 添加了完了数据，但是没有对 `count++` ，线程 B 就已经读取 `count` 了，那么对于线程 B 来说读取到的 `count` 一定是不准确的。

**而给 size() 方法加了同步之后，意味着线程 B 调用 #size() 方法只有在线程 A 调用 put 方法完毕之后才可以调用，这样就保证了线程安全性**。

## HashSet 和 HashMap 的区别？

- Set 是线性结构，值不能重复。HashSet 是 Set 的 hash 实现，HashSet 中值不能重复是用 HashMap 的 key 来实现的。

- Map 是键值对映射，可以空键空值。**HashMap 是 Map 的 hash 实现，key 的唯一性是通过 key 值 hashcode 的唯一来确定，value 值是则是链表结构。**

  > 因为不同的 key 值，可能有相同的 hashcode ，所以 value 值需要是链表结构。

他们的共同点都是 hash 算法实现的唯一性，他们都不能持有基本类型，只能持有对象。

> 为了更好的性能，Netty 自己实现了 key 为基本类型的 HashMap ，例如 [IntObjectHashMap](https://netty.io/4.1/api/io/netty/util/collection/IntObjectHashMap.html) 。

## HashSet 和 TreeSet 的区别？

- HashSet 是用一个 hash 表来实现的，因此，它的元素是无序的。添加，删除和 HashSet 包括的方法的持续时间复杂度是 `O(1)` 。
- TreeSet 是用一个树形结构实现的，因此，它是有序的。添加，删除和 TreeSet 包含的方法的持续时间复杂度是 `O(logn)` 。

🦅 **如何决定选用 HashMap 还是 TreeMap？**

- 对于在 Map 中插入、删除和定位元素这类操作，HashMap 是最好的选择。
- 然而，假如你需要对一个有序的 key 集合进行遍历， TreeMap 是更好的选择。

基于你的 collection 的大小，也许向 HashMap 中添加元素会更快，再将 HashMap 换为 TreeMap 进行有序 key 的遍历。

## HashMap 和 ConcurrentHashMap 的区别？

ConcurrentHashMap 是线程安全的 HashMap 的实现。主要区别如下：

- 1、**ConcurrentHashMap 对整个桶数组进行了分割分段(Segment)，然后在每一个分段上都用 lock 锁进行保护，相对 于Hashtable 的 syn 关键字锁的粒度更精细了一些，并发性能更好。而 HashMap 没有锁机制，不是线程安全的。**

  > JDK8 之后，ConcurrentHashMap 启用了一种全新的方式实现,利用 CAS 算法，并且也引入了红黑树。
  >
  > ==ConcurrentHashMap 是一个 Segment 数组，Segment 通过继承ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。==
  >
  > concurrencyLevel:并行级别，默认是 16，也就是说 ConcurrentHashMap 有 16 个 Segments，所以理论上，这个时候，**最多可以同时支持 16 个线程并发写，只要它们的操作分别分布在不同的 Segment 上。**这个值可以在初始化的时候设置为其他值，但是一旦初始化以后，它是不可以扩容的。再具体到每个 Segment 内部，其实每个 Segment 很像之前介绍的 HashMap，不过它要保证线程安全，所以处理起来要麻烦些。

- 2、HashMap 的键值对允许有 `null` ，但是 ConCurrentHashMap 都不允许。

## 队列和栈是什么，列出它们的区别？

栈和队列两者都被用来预存储数据。

- java.util.Queue是一个接口，它的实现类在Java并发包中。
  - 队列允许先进先出（FIFO）检索元素，但并非总是这样。
  - Deque 接口允许从两端检索元素。
- 栈与队列很相似，但它允许对元素进行后进先出（LIFO）进行检索。
  - Stack 是一个扩展自 Vector 的类，而 Queue 是一个接口。

## 为什么HashMap允许key，value都可以为空，而ConCurrentHashMap和Hashtable不允许？

![image-20190414112633320](/Users/jack/Desktop/md/images/image-20190414112633320.png)



[《LinkedHashMap 源码详细分析（JDK1.8）》](https://www.imooc.com/article/22931)



















