# Java集合

![img](/Users/jack/Desktop/md/images/java-coll.png)

![img](/Users/jack/Desktop/md/images/jpeg.jpeg)

## 一、集合接口

集合框架定义了一些接口。本节提供了每个接口的概述：

| 序号 | 接口描述                                                     |
| ---- | ------------------------------------------------------------ |
| 1    | Collection 接口 Collection 是最基本的集合接口，一个 Collection 代表一组 Object，即 Collection 的元素, Java不提供直接继承自Collection的类，只提供继承于的子接口(如List和set)。Collection 接口存储一组不唯一，无序的对象。 |
| 2    | List 接口 List接口是一个有序的 Collection，使用此接口能够精确的控制每个元素插入的位置，能够通过索引(元素在List中位置，类似于数组的下标)来访问List中的元素，第一个元素的索引为 0，而且允许有相同的元素。List 接口存储一组不唯一，有序（插入顺序）的对象。 |
| 3    | Set Set 具有与 Collection 完全一样的接口，只是行为上不同，Set 不保存重复的元素。Set 接口存储一组唯一，无序的对象。 |
| 4    | SortedSet  继承于Set保存有序的集合。                         |
| 5    | Map Map 接口存储一组键值对象，提供key（键）到value（值）的映射。 |
| 6    | Map.Entry  描述在一个Map中的一个元素（键/值对）。是一个Map的内部类。 |
| 7    | SortedMap 继承于 Map，使 Key 保持在升序排列。                |
| 8    | Enumeration 这是一个传统的接口和定义的方法，通过它可以枚举（一次获得一个）对象集合中的元素。这个传统接口已被迭代器取代。 |

### Set和List的区别

- 1. Set 接口实例存储的是无序的，不重复的数据。List 接口实例存储的是有序的，可以重复的元素。
- 2. Set检索效率低下，删除和插入效率高，插入和删除不会引起元素位置改变 **<实现类有HashSet,TreeSet>**。
- 3. List和数组类似，可以动态增长，根据实际存储的数据的长度自动增长List的长度。查找元素效率高，插入删除效率低，因为会引起其他元素位置改变 **<实现类有ArrayList,LinkedList,Vector>** 。

### 集合实现类（集合类）

Java提供了一套实现了Collection接口的标准集合类。其中一些是具体类，这些类可以直接拿来使用，而另外一些是抽象类，提供了接口的部分实现。

标准集合类汇总于下表：

| 序号 | 类描述                                                       |
| ---- | ------------------------------------------------------------ |
| 1    | **AbstractCollection**  实现了大部分的集合接口。             |
| 2    | **AbstractList**  继承于AbstractCollection 并且实现了大部分List接口。 |
| 3    | **AbstractSequentialList**  继承于 AbstractList ，提供了对数据元素的链式访问而不是随机访问。 |
| 4    | LinkedList 该类实现了List接口，允许有null（空）元素。主要用于创建链表数据结构，该类没有同步方法，如果多个线程同时访问一个List，则必须自己实现访问同步，解决方法就是在创建List时候构造一个同步的List。例如：`Listlist=Collections.synchronizedList(newLinkedList(...));`LinkedList 查找效率低。 |
| 5    | ArrayList 该类也是实现了List的接口，实现了可变大小的数组，随机访问和遍历元素时，提供更好的性能。该类也是非同步的,在多线程的情况下不要使用。ArrayList 扩容时，增长当前长度的50%，插入删除效率低。(int newCapacity = oldCapacity + (oldCapacity >> 1);右移一位，相当于除以2) |
| 6    | **AbstractSet**  继承于AbstractCollection 并且实现了大部分Set接口。 |
| 7    | HashSet 该类实现了Set接口，不允许出现重复元素，不保证集合中元素的顺序，允许包含值为null的元素，但最多只能一个。 |
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
| 5    | Properties  Properties 继承于 Hashtable，表示一个持久的属性集，属性列表中每个键及其对应值都是一个字符串。 |
| 6    | BitSet 一个Bitset类创建一种特殊类型的数组来保存位值。BitSet中数组大小会随需要增加。 |

### **集合框架底层数据结构总结**

1）List

- ArrayList ：Object 数组。
- Vector ：Object 数组。
- LinkedList ：双向链表(JDK6 之前为循环链表，JDK7 取消了循环)。

2）Map

- HashMap ：
  - JDK8 之前，HashMap 由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。
  - JDK8 以后，在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8 ）时，将链表转化为红黑树，以减少搜索时间。
- LinkedHashMap ：LinkedHashMap 继承自 HashMap，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，LinkedHashMap 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。详细可以查看：[《LinkedHashMap 源码详细分析（JDK1.8）》](https://www.imooc.com/article/22931) 。
- Hashtable ：数组+链表组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的。
- TreeMap ：红黑树（自平衡的排序二叉树）。

3）Set

- HashSet ：无序，唯一，基于 HashMap 实现的，底层采用 HashMap 来保存元素。
- LinkedHashSet ：LinkedHashSet 继承自 HashSet，并且其内部是通过 LinkedHashMap 来实现的。有点类似于我们之前说的LinkedHashMap 其内部是基于 HashMap 实现一样，不过还是有一点点区别的。
- TreeSet ：有序，唯一，红黑树(自平衡的排序二叉树)。

### **快速失败（fail-fast）和安全失败（fail-safe）的区别**

差别在于 ConcurrentModification 异常：

- 快速失败：当你在迭代一个集合的时候，如果有另一个线程正在修改你正在访问的那个集合时，就会抛出一个 ConcurrentModification 异常。 在 `java.util` 包下的都是快速失败。
- 安全失败：你在迭代的时候会去底层集合做一个拷贝，所以你在修改上层集合的时候是不会受影响的，不会抛出 ConcurrentModification 异常。在 `java.util.concurrent` 包下的全是安全失败的。

### Comparable 和 Comparator 的区别

- Comparable 接口，在 `java.lang` 包下，用于当前对象和其它对象的比较，所以它有一个 `#compareTo(Object obj)` 方法用来排序，该方法只有一个参数。
- Comparator 接口，在 `java.util` 包下，用于传入的两个对象的比较，所以它有一个 `#compare(Object obj1, Object obj2)` 方法用来排序，该方法有两个参数。

详细的，可以看看 [《Java 自定义比较器》](https://blog.csdn.net/striveb/article/details/88086451) 文章，重点是如何自己实现 Comparable 和 Comparator 的方法。





[《LinkedHashMap 源码详细分析（JDK1.8）》](https://www.imooc.com/article/22931)



















