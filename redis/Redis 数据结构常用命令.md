# Redis 数据结构常用命令

## 1.Redis 数据结构一一字符串

符串是 Redis 最基本的数据结构 ，它将以 一个键和 一个值存储于 Redis 内部，它犹如 Java 的 Map 结构 ，让 Redis 通过键去找到值。![image-20181215234629602](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181215234629602.png)

​	**==常见的命令：==**  

![image-20181215234807167](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181215234807167-4888887.png)

Redis 除了这些之外还提供了对整数和浮点型 数字的功能，如果字符串是数字(整数或者浮点数〉，那么 Redis还能支持简单的运算。不过它的运算能力比较弱 ，![image-20181215234901021](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181215234901021-4888941.png)

##### redis的运算实例：

![image-20181215235006754](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181215235006754-4889006.png)

## 2.Redis 数据结构一一哈希

​	在 Redis 中，**hash是一个 String类型的 field和 value 的映射表**，因此我们存储的数据实际在 Redis 内存中都是一个个字符串而己。==即我们首先会通过key找到hash结构，然后再根据hash结构里面的filed找到对应的value。==

![image-20181216141755147](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216141755147-4941075.png)

​	**由上表可看出，在 Redis 中的哈希结构和字符串有着比较明显的不同。首先，命令都是以 h 开头，代表操作的是 hash 结构。其次，大多数命令多了一个层级 field，这是hash 结构的一个内部键，也就是说 Redis 需要通过 key 索引到对应的 hash 结构，再通过 field来确定使用 hash 结构的哪个键值对 。**

##### 注意：

- 哈希结构的大小，如果哈希结构是个很大的键值对，那么使用它要十分注意，尤其是关于 hkeys、 hgetall、 hvals 等返回所有哈希结构数据的命令，会造成大量数据的读取。这需要考虑性能和读取数据大小对 NM 内存的影响 。 
- 对于**数字的操作命令 hincrby** 而言，要求存储的也是整数型的字符串；对于 hincrbyfloat 而言，则要求使用浮点数或者整数，否则命令会失败。 
- 例子：![image-20181216154807805](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216154807805-4946487.png)

####  Spring操作Redis的hash结构：

![image-20181216163852520](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216163852520-4949532.png)

![image-20181216163909465](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216163909465-4949549.png)



- hmset命令，在Java的API中，是使用map保存多个键值对在先的，**多对filed-value**
- hgetall命令会返回所有的键值对，并保存到一个 map对象中，如果 hash结构很大， 那么要考虑它对 NM 的内存影响。 
- hincrby和 hincrbyFloat命令都采用 increment方法， Spring会识别它具体使用何种方法 
- redisTemplate.opsForHash().values(key)方法相当于 **hvals 命令，它会返回所有的值，并保存到一个 List对象中** ;而 redisTemplate.opsForHash().keys(key)方法相当于 hkeys 命令，它会获取所有的键，保存到一个 Set对象中 。 
- 在 Spring 中使用 **redisTemplate.opsForHash().putAll(key, map)方法相当于执行 了 hmset 命令**，使用了 map，由于配置了默认的序列化器为字符串，所以它也 只会用 字符串进行转化，这样才能执行对应的数值加法，如果使用其他序列化器，则后面 的命令可能会抛出异常。 
- 在使用大的 hash 结构时，需要考虑返回数据的大小，以避免返回太多的数据，引发 NM 内存溢出或者 Redis 的性能问题。 

## 3.Redis 数据结构一一链表( linked-list )

​	**Redis 链表是双向的，因此即可以从左到右，也可以从右到左遍历它存储的节点，链表结构如图:**![image-20181216155937230](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216155937230-4947177.png)

​	链表结构适合插入和删除，而不适合查找：![image-20181216161750117](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216161750117-4948270.png)

##### 图中那个，阿拉伯数字代表新增的步骤 ，而汉 字数字代表删除步骤。

- 新增节点 。 **对插入图中的节点 4 而言，先看从左到右的指向，先让节点 4 指向节点 l 原来的下一个节点，也就是节点 2，然后让节点 l 指向节点 4，这样就完成了从右到左的指向修改**;再看从右到左，先让节点 4 指向 节点 1，然后节点 2 指向节点 4, 这个时候就完成了从右到左的指向，那么节点 l 和节点 2 之间的原有关联关系都已经失效，这样就完成了在链表中新增节点 4 的功能 。
- 删除节点 。 **对删除图中的节点 3 而言 ，首先让节点 2 从左到右指向后续节点，然后让后续节点指向节点 2，**这样节点 3 就脱离了链表， 也就 是断绝了与节点 2 和后继节点的关联关系，然后对节点 3 进行内存回收 ，无须移动任何节点，就完成了 删除 。 

因为是双向链表结构，所以 Redis 链表命令分为左操作和右操作两种命令，左操作就意味着是从左到右，右操作就意味着是从右到左。 Redis关于链表的命令如表：![image-20181216162347938](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216162347938-4948627.png)

##### 	以“l”开头的代表左操作，以“r”开头的代表右操作。

对于大量数据操作的时候，我们需要考虑插入和删除内容的大小，因为这将是十分消耗性能的命令，会导致 Redis 服务器的卡顿 。对于不允许卡顿的一些服务器，**可以进行分批次操作**，以避免出现卡顿。

##### 	例子：![image-20181216162553496](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216162553496-4948753.png)

​	之前这些操作链表的命令都是进程不安全的，因为当我们操作这些命令的时候，其他 Redis 的客户端也可能操作同一个链表，这样就会造成并发数据安全和一致性问题，尤其是当你操作 一个数据 量不 小的链表结构时，常常会遇到这样的问题 。 **为了克服这些问题， Redis提供了链表的阻塞命令，它们在运行的时候， 会给链表加锁，以保证操作链表的命令安全性**，如下表：![image-20181216162732497](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216162732497-4948852.png)

​	当使用这些命令时， Redis就会对对应的链表加锁，加锁的结果就是其他的进程不能再读取或者写入该链表，只能等待命令结束。 加锁的好处可以保证在多线程并发环境中数据的一致性，保证一些重要数据的一致性，比如账户的金额、 商品的数量。不过在保证这些的同时也要付出其他线程等待、线程环境切换等代价，这将使得系统的并发能力下降，关于多线程并发锁， 未来还会提及，这里先看Redis链表阻塞操作命令：![image-20181216163236148](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216163236148-4949156.png)

##### Spring 操作 Redis 链表：

![image-20181216164154432](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216164154432-4949714.png)

![image-20181216164450179](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216164450179-4949890.png)

![image-20181216164541070](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216164541070-4949941.png)

​	有些命令 Spring所提供的 RedisTemplate并不能支持，比如 linsert命令，这个时候可以使用更为底层的方法去操作,像上面的所示。

## 4.Redis 数据结构一一集合

​	**Redis 的集合不是一个线性结构，而是一个哈希表结构**，它的内部会根据 hash 分子来 存储和查找数据，理论上一个集合可以存储 232一 1 (大约 42 亿)个元素，**因为采用哈希表结构，所以对于 Redis集合的插入、删除和查找的复杂度都是 0(1)**，只是我们需要注意 3 点：

- **对于集合而言，它的每一个元素都是不能重复的，当插入相同记录的时候都会失败 。** 
- **集合是无序的**。
- ==集合的每一个元素都是 String数据结构类型。== 

Redis的集合可以对于不同的集合进行操作，比如求出两个或者以上集合的交集、 差集和并集等。集合命令如下表：![image-20181216165008020](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216165008020-4950208.png)

![image-20181216165037196](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216165037196-4950237.png)

​	集合是无序的， 并且支持并集、 交集和差集的运算，下面通过命令行客户端来演示这些命令。

##### 例子： ![image-20181216165536152](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216165536152-4950536.png)

​	**求差集、并集和交集 ， 并保存到新的集合中**

![image-20181216165632328](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216165632328-4950592.png)

##### spring操作Redis集合：redisTemplate.opsForSet().xxx

## 5.Redis 数据结构一一有序集合

​	有序集合和集合类似，只是说它是有序的，**和无序集合的主要区别在于每 一个元素除了值之外，它还会多一个分数。分数是一个浮点数，在 Java 中是使用双精度表示的**，==根据分数， Redis就可以支持对分数从小到大或者从大到小的排序。==这里和无序集合一样，对于每一个元素都是唯一的 ，但是对于不同元素而言，它的分数可以一样。元素也是 String 数据类型，也是一种基于 hash 的存储结构。集合是通过哈希表实现的，所以添加、删除、 查找的复杂度都是 0(1)。集合中最大的成员数为 232一 1 (40 多亿个成员)，有序集合的数据结构如图：![image-20181216165837115](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216165837115-4950717.png)	

​	**==有序集合是依赖 key 标示它是属于哪个集合，依赖分数进行排序，所以值和分数是必须的，而实际上不仅可以对分数进行排序 ，在满足一定的条件下，也可以对值进行排序 。==** 

![image-20181216170141750](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216170141750-4950901.png)

![image-20181216170157402](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216170157402-4950917.png)

![image-20181216170209664](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216170209664-4950929.png)

​	在对有序集合、下标、区间的表示方法进行操作的时候，需要十分小心命令，注意它是操作分数还是值，稍有不慎就会出现问题。	

##### spring操作Redis有序集合：

**在 Spring 中使用 Redis 的有序集合，需要注意的是 Spring 对 Redis 有序 集合的元素的值和分数的范围(Range)和限制(Limit)进行了封装，**主要是==TypedTuple==接口：

```Java
package org.springframework.data.redis.core;
public interface ZSetOperations<K, V> {
	public interface TypedTuple<V> extends Comparable<TypedTuple<V>> {
		V getValue();
		Double getScore();
	}
}
```

​	这里的 getValue()是获取值，而 getScore()是获取分数，但是它只是一个接口，而不是 一个实现类。 spring-data-redis 提供了 一个默认的实现类一-DefaultTypedTuple，同样它会 实现 TypedTuple接口， 在默认的情况下 Spring就会把带有分数的有序集合的值和分数封装 到这个类中 ，这样就可以通过这个类对象读取对应的值和分数了 。 

​	Spring 不仅对有序集合元素封装，而且对范围也进行了封装，方便使用。它是使用接口 org.springframe.work.data.r巳dis.connection.RedisZSetCommands 下的内部类 Range 进行封 装的，它有一个静态的 range()方法，使用它就可以生成一个 Range对象了，只是要清楚 Range 对象的几个方法才行：![image-20181216170646222](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216170646222-4951206.png)

​	这 4个方法就是最常用 的范围方法。下面讨论一下限制，它是接口 org.springframework.
data.redis.connection.RedisZSetCommands 下的内部类，它是一个简单的 POJO，它存在两个属性，它们的 getter和 setter方法， 如下面的代码所示：

```Java
package org.springframework.data.redis.connection ; 
// ..
public interface RedisZSetCommands {
// ..
public class Limit {
	int offset; 
	int count;
	/******setter and getter******/
	}
	//...
}
```

通过属性的名称很容易知道:。他et代表从第几个开始截取，而 count代表限制返回的总数量。

## 6.基数一HyperLogLog

​	基数是一种算法，表示一个集合里面不重复元素的个数。比如数字集合{ l,2,5几9,1,5,9}的基数集合为{ 1,2,5几9}那么基数(不重复元素)就是 5。基数的作用是评估大约需要准备多少个存储单元去存储数据，但是基数的算法一般会存在一定的误差(一般是可控的)。 

​	基数并不是存储元素，存储元素消耗 内存空间比较大，而是给某一个有重复元素的数据集合( 一般是很大的数据集合〉评估需要的空 间单元数 ，所以它 没有办法进行存储 。![image-20181216172554472](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216172554472-4952354.png)

![image-20181216172648499](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216172648499-4952408.png)

##### spring操作基数：

![image-20181216172730964](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216172730964-4952451.png)

​	**增加一个元素到基数中采用 add 方法，它可以是一个或者多 个元素，而求基数大小则是采用了 size 方法，合并基数则采用了 union 方法，其第一个是目标基数的 key，然后可以是一到多个 key。** 





参照：《Java EE互联网轻量级框架整合开发 SSM框架（Spring MVC+Spring+MyBatis）和Redis实现》