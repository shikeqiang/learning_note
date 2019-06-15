# Redis常用技术

Redis 是存在事务的,Redis的事务是使用 MULTI-EXEC的命令组合，使用它可以提供两个重要的保证 :

- **事务是一个被隔离的操作，事务中的方法都会被 Redis 进行序列化并按顺序执行， 事务在执行的过程中不会被其他客户端发生的命令所打断。** 
- **事务是一个原子性的操作，它要么全部执行，要么就什么都不执行。** 

在 Redis 中使用事务会经过 3 个过程:**开启事务，命令进入队列，执行事务。**

##### redis事务相关命令：

![image-20181216211135228](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216211135228-4965895.png)

## 1.Redis 的基础事务

​	**在 Redis 中 开启事务是 ==multi== 命令，而执行事务是==exec==命令。 multi 到 exec 命令之间的 Redis 命令将采取进入队列的形式，直至 exec 命令的出现，才会一次性发送队列里的命令去执行 ，而在执行这些命令的时候其他客户端就不能再插入任何命令了，这就是 事务机制。**  执行过程如下图：![image-20181216211654133](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216211654133-4966214.png)

​	**由上图可以看出，先用multi开启事务，然后发现输入set和get后并没有马上执行，而是进入了“QUEUED”，进入了存储队列。接着执行exec命令，发现set和get命令都执行并返回结果了。**

​	如果回滚事务，则可以使用 discard 命令，它就会进入在事务队列中的命令，这样事务中的方法就不会被执行了，使用 discard命令取消事务如图：![image-20181216212816641](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216212816641-4966896.png)

​	当我们使用了 discard命令后，再使用 exec命令时就会报错，因为 discard命令已经取消了事务中的命令，而到了 exec命令时，队列里面己经没有命令可以执行了，所以就出现了报错的情况。

在Spring中要使用同一个连接操作Redis命令的场景，这个时候我们借助的是 Spring提供的 **SessionCallback**接口，采用 Spring去实现本节的命令：![image-20181216213832164](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216213832164-4967512.png)

​	**==使用了 SessionCallBack 接口，从而保证所有的命令都是通过同一个Redis 的连接进行操作的。==**在使用 multi命令后， 要特别注意的是，使用 get等返回值的方法一律返回为空，因为在 Redis 中它只是把命令缓存到队列中，而没有去执行 。 使用 exec 后就会执行事务，执行完了事务后，执行 get命令就能正常返回结果了。最后使用 redisTemplate.execute(callBack);就能执行我们在 SessionCallBack 接口定义的Lambda 表达式的业务逻辑，并将获得其返回值。

**执行结果：**事务执行过程中，命令入队列，而没有被执行，所以 value 为 空 : value=null
valuel

如果我们希望得到 Redis 执行事务各个命令的结果，可以用这行代码 :

```
List list = ops.exec () ; //执行事务
```

这段代码将返回之前在事务队列中所有命令的执行结果，并保存在一个 List 中，我们只要在 SessionCallback 接口的 execute 方法中将 list返回，就可以在程序中获得各个命令执行的结果了 。

## 2.探索Redis事务回滚

Redis的事务回滚分为两种情况：一种是命令格式正确而数据类型不符合，一种是命令格式不正确。![image-20181216215644601](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216215644601-4968604.png)

​	如上图，就是数据类型不符合，**使用命令 incr对其自增，但是命令只会进入事务队列，而没有被执行，所以它不会有任何的错误发生，而是等待 exec命令的执行。**当 exec命令执行后，之前进入队列的命令就依次执行，当遇到 incr时发生命令操作的数据类型错误，所以显示出了错误，而其之前和之后的命令都会被正常执行。注意，这里命令格式是正确的，问题在于数据类型，对于命令格式是错误的却是另外一种情形，如图： ![image-20181216215758077](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216215758077-4968678.png)

​	使用的 incr命令格式是错误的，这个时候 Redis会立即检测出来并产生错误，而在此之前我们设置了 keyl， 在此之后我们设置了 key2a 当事务执行的时候，我们发现 keyl 和 key2 的值都为空 ，说明被 Redis 事务回滚了。

​	通过上面的例子我们发现，在执行事务命令的时候，**在命令入队的时候， Redis 就会检测事务的命令是否正确，如果不正确则会产生错误。**==无论之前和之后的命令都会被事务所回滚，就变为什么都没有执行。当命令格式正确，而因为操作数据结构引起的错误 ，则 该命令执行出现错误，而其之前和之后的命令都会被正常执行。==这点和数据库很不一样， 这是需要读者注意的地方。对于一些重要的操作 ，我们必须通过程序去检测数据的正确性， 以保证Redis事务的正确执行，避免出现数据不一致的情况。 Redis之所以保持这样简易的 事务 ，完全是为了保证移动互联网的核心问题一一性能。 

## 3.使用 watch 命令监控事务

​	**==在Redis中使用watch命令可以决定事务是执行还是回滚。==** **一般而言，可以在multi命令之前使用 watch 命令监控某些键值对，然后使用 multi 命令开启事务，执行各类对数据结构进行操作的命令，这个时候这些命令就会进入队列。当 Redis使用 exec命令执行事务的时候，它首先会去比对被 watch 命令所监控的键值对，如果没有发生变化，那么它会执行事务队列中的命令，提交事务;如果发生变化，那么它不会执行任何事务中的命令，而去事务回滚。无论事务是否回滚 ， Redis 都会去取消执行事务前的 watch 命令，**这个过程如图：![image-20181216220717589](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181216220717589-4969237.png)

​	**==Redis参考了多线程中使用的 CAS (比较与交换， Compare And Swap) 去执行的。==**其实就是有点像乐观锁那样子的，通过跟原来的值比较，然后判断是否一致。首先，在线程开始读取这些多线程共享的数据，并将其保存到当前进程的副本中，称为旧值，watch命令就是实现这个功能。然后，开启线程业务逻辑，有multi命令提供这个功能。在执行更新前，比较当前线程副本保存的旧值和当前线程共享的值是否一致，如果不一致，表明该数据已经被其他线程操作过，此次更新失败。为了保持一致，线程就不去更新任何值，而将事务回滚；否则就认为它没有被其他线程操作过，执行对应的业务逻辑，exec命令就是执行“类似”这样的一个功能。

​	但是，要注意CAS原理会产生ABA问题。即有一个线程，对原来的值进行了可能多于2次修改，反正最后的值跟原来是一样的，所以是ABA问题。所以要再加上别的方法避免ABA问题。常见的方法有通过版本号version控制，如 Hibernate 对缓存的持久对象( PO)加入宇段 version 值，当每次操作一次该 PO，则 version=version+1， 这样采用 CAS 原理探测 version 宇段， 就能在多线程的环境中，排除 ABA 问题，从而保证数据的一致性。 

​	Redis在执行事务的过程中， 并不会阻塞其他连接的并发，而只是通过比较watch监控的键值对去保证数据的一致性， 所以Redis多个事务完全可以在非阻塞的多线程环境中井发执行，而且Redis的机制是不会产生ABA问题的， 这样就有利于在保证数据一致的基础上 ， 提高高并发系统的数据读/写性能。

##### 例子：![image-20181217100846579](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217100846579-5012526.png)

##### ![image-20181217100909914](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217100909914-5012549.png)

## 4.流水线( pipelined )

​	在事务中 Redis 提供了队列， 这是一个可以批量执行任务的队列，这样性能就比较高，但是使用 multi...exec事务命令是有系统开销的，因为它会检测对应的锁和序列化命令。有时候我们希望在没有任何附加条件的场景下去使用队列批量执行一系列的命令，从而提高系统性能，这就是 Redis 的流水线(pipelined)技术。

 	Redis 执行读/写速度十分快，而系统的瓶颈往往是在网络通信中的延时，即当命令1在T1时刻发送到Redis之后，Redis很快地处理完命令1，命令 2 在 T2 时刻却没有通过网络送达 Redis 服务器，这样就变成了 Redis 服务器在等待命令 2 的到来，当命令2送达，被执行后，而命令 3 又没有送达 Redis, Redis 又要继续等待，依此类推，这样 Redis 的等待时间就会很长，很多时候在空闲的状态，而问题出在网络的延迟中，造成了系统瓶颈。如图：![image-20181217101629999](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217101629999-5012990.png)

​	为了解决这个问题，可以使用Redis的流水线， **Redis的流水线是一种通信协议，**下面通过Java API代码来演示：

```Java
	Jedis jedis = pool.getResource();
    long start = System.currentTimeMillis(); //开启流水线
    Pipeline pipeline = jedis.pipelined(); 
    //这里测试 10万条的读/写 2个操作
	for(int i = O;i<100000;i++){
		int j = i + 1;
        pipeline.set(”pipeline_key_” + j ,”pipeline value ”+j);
        pipeline.get(”pipeline_key ” + j);
    }
	//pipeline .sync ();//这里只执行同步，但是不返回结果
    // pipeline.syncAndReturnAll();将返回执行过的命令返回的 List 列表结果 
    List result = pipeline.syncAndReturnAll();
    long end = System.currentTimeMillis();
	//计算耗时
    System.err.println(”耗时: ”+(end - start) + ”毫秒”);
```

**​	结果显示耗时在 550 毫秒到 700 毫秒之间 ，即不到 1 秒的时间就完成多达 10 万次读/写。**

##### spring操作流水线：![image-20181217102819540](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217102819540-5013699.png)

## 5.发布订阅

​	发布订阅模式首先需要消息源，也就是要有消息发布出来，订阅者就可以收到这个消息进行处理了，观察者模式就是这个模式的典型应用了 。Redis支持这一模式。如下图：

![image-20181217103434506](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217103434506-5014074.png)

首先来注册一个订阅的客户端， 这个时候使用 **==SUBSCRIBE==** 命令。
比如监听一个叫作 chat的渠道， 这个时候我们需要先打开一个客户端，这里记为客户端 1，然后输入命令 : 

```
SUBSCRIBE chat
```

这个时候**客户端 1 就会订阅了一个叫作 chat渠道的消息了**。之后打开另外一个客户端，记为客户端2， 输入命令:

```
publish chat ”let’s go!!”
```

这个时候客户端 2 就向渠道 chat 发送消息 :	”let’s go!!”   如下图：

![image-20181217103808286](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217103808286-5014288.png)

​	Spring 的工作环境中展示如何配置发布订阅模式。首先提供接收消息的类， 它将实现 **org.springframework.data.redis.connection.MessageListener** 接口， 并实现接口定义的方法 **public void onMessage(Message message, byte[] pattern)**, Redis 发布订阅监听类如代码：![image-20181217103949059](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217103949059-5014389.png)

需要配置RedisMessageListener，这样就在 Spring 上下文中定义了监昕类。：

```xml
<bean id=” redisMsgListener”  class=”com.ssm.chapterl9.redis.listener.RedisMessageListener”> <property name=” redisTemplate” ref=” redisTemplate” />
</bean>
```

此外，还需要配置监听容器，在 Spring 中己有类org.springframework.data.redis.listener.RedisMessageListenerContainer。它可以用于监听Redis
的发布订阅消息，

```xml
<bean id=” topicContainer”
class=”org.springframework.data.redis .listener.RedisMessageListenerContainer”
destroy-method=” de stroy” >
< ! --Redis 连接工厂 -->
<property name=” connectionFactory” ref=” connectionFactory” /> 
<!一连接池，这里只要线程池生存，才能继续监听 -->
<property name=” taskExecutor” >
<bean
class = ”org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler ”>
<property name=”poolSize” value=”3”/>
</bean>
</property>
<!--消息监昕 Map-->
<property name=”messageListeners”>
<map>
<!--配置监听者， key-ref和 bean id定义一致-->
<entry key-ref=” redisMsgListener”>
<!--监听类-->
<bean class=”org.springframework.data.redis.listener.ChannelTopic”>
<constructor-arg value=” chat” / >
</bean>
</entry>
</map> 
</property> 
</bean>
```

上面的代码配置了线程池，这个线程池将持续的生存以等待消息传入，而这里配置了容器 用 id为redisMsgListener的 Bean进行对渠道 chat的监昕。当消息通过渠道 chat发送的时候，就会使用 id为 redisMsgListener的 Bean进行处理消息。 

测试Redis发布订阅：

![image-20181217104434300](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217104434300-5014674.png)

​	convertAndSend 方法就是向渠道 chat 发送消息的， 当发送后对应的监听者就能监听到消息了。运行它，后台就会打出对应的消息:	

![image-20181217104502527](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217104502527-5014702.png)

​	其实跟消息队列的发布订阅模式是差不多的，都是通过发布消息，接收消息，需要有订阅容器和监听容器。

## 6.超时命令

​	Redis也是基于内存而运行的数据集合，也存在着对内存垃圾的回收和管理的问题。	

​	一般而言，和 Java虚拟机一样，当内存不足时 Redis会触发自动垃圾回收的机制，而程序员可以通过 System.gc()去建议 Java虚拟机回收内存垃圾，它将“可能”(注意， System.gc()并不一定会触发 NM 去执行回收，它仅仅是建议JVM做回收)触发一次 Java虚拟机的回收机制，但是如果这样做可能导致 Java虚拟机在回收大量的内存空间的同时，引发性能低下的情况。

**键值对超时命令：**![image-20181217105254892](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217105254892-5015174.png)	

![image-20181217105717422](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217105717422-5015437.png)

##### spring操作Redis超市命令：

```java 
redisTemplate. execute ((Red工sOperations ops) -> {
	ops.boundValueOps(”keyl”).set(”valuel”);
	String keyValue = (String)ops.boundValueOps(”keyl”).get(); 
	Long expSecond =ops.getExpire(”keyl”);
	System.err.println(expSecond) ;
	boolean b =false ;
	b = ops .expire (”keyl”, 120L, TimeUnit.SECONDS);
	b = ops.persist(”keyl”);
	Long 1 = OL;
	1 = ops.getExpire (”keyl”);
	Long now= System.currentTimeMillis();
	Date date= new Date() ;
	date . setTime (now + 120000) ;
	ops.expireAt(”key”, date) ;
	return null ;
	});
```

 	**==Redis 的 key 超时不会被其自动回收，它只会标识哪些键值对超时了==**。这样做的一个好处在于，如果一个很大的键值对超时，比如一个列表或者哈希结构，存在数以百万个元素，要**对其回收需要很长的时间** 。 如果采用超
时回收，则可能产生停顿。坏处也很明显，这些超时的键值对会浪费比较多的空间。

##### 	Redis 提供两种 方式回收这些超 时键值对， 它们是定时回收和惰性回收。	

- **定时回收是指在确定的某个时间触发一段代码，回收超时的键值对 。**

- **惰性回收则是当一个超时的键，被再次用 get 命令访问时，将触发 Redis 将其从内存中清空。**

  定时回收可以完全回收 些超 时的键值对，但 是缺点也很明显，如果这些键值对比较 多， 则 Redis 需要运行较 的时间，从而导致停顿。所以系统设计者 一般会选择在没有业 务发生的时刻触发 Redis 的定时回收，以便清理超时的键值对。对于惰性回收而言，它的 优势是可以指定回收超时的键值对 i 它的缺点是要执行一个莫名其妙的 get 操作，或者在某些时候 ，我们也难以判断哪 些键值对已经超时。 

## 7.使用Lua语言

​	执行Lua语言是原子性的， 也就说Redis执行 Lua的时候是不会被中断的，具备原子性，这个特性有助于 Redis 对并发数据一致性的支持。	

​	Redis支持两种方法运行脚本， 一种是直接输入一些 Lua语言的程序代码；另外一种是将 Lua 语言编写成文件。对于采用简单脚本的， Redis支持缓存脚本，只是它会使用 SHA-1 算法对脚本进行签名，然后把 SHA-1 标识返回回来，只要通过这个标识运行就可以了。

### 7.1 执行输入 Lua 程序代码

##### 命令格式为：

```lua
eval lua-script key-num [keyl key2 key3 ... ] [valuel value2 value3 ... ]
```

- **eval代表执行 Lua语言的命令。** 
- **Lua-script代表 Lua 语言脚本 。** 
- **key-num 整数代表参数中有多少个 key，==需要注意的是 Redis 中 key 是从 1 开始的，如果没有 key 的参数，那么写 0。==** 
- **[key1key2key3...]是key作为参数传递给Lua语言，也可以不填它是key的参数， 但 是需要和 key-num 的个数对应起来。** 
- **[valuel value2 value3 ...]这些参数传递给 Lua 语言，它们是可填可不填的。** 

##### 例子：![image-20181217113026018](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217113026018-5017426.png)

```
EVAL "return 'hello'" 0
```

这个脚本只是返回一个字符串，并不需要任何参数，所以 key-num 填写了 0，代表着没有任何 key参数。按照脚本的结果就是返回了 hello，所以执行后 Redis也是这样返回的 。 这个例子很简单，只是返回 一个字符串。

```
EVAL "redis.call('set',KEYS[1],ARGV[1])" 1 lua-key lua-value
```

**设置一个键值对，可以在 Lua 语言中采用 redis.call(command, key[paraml , param2...]) 进行操作，其中:** 		

- command是命令，包括 set、 get、 del等。
- Key是被操作的键。
- paraml,param2...代表给 key 的参数 。

**脚本中的KEYS[l]代表读取传递给 Lua脚本的第一个 key参数，而 ARGV[l]代表第一 个非 key参数。**这里共有一个 key参数，所以填写的 key-num为 l，这样 Redis就知道 key-value 是 key参数，而 lua-value是其他参数，它起到的是一种间隔的作用。最后我们可以看到使 用 get 命令获取数据是成功的，所以 Lua 脚本运行成功了 。 

​	有时可能需要多次执行同样一段脚本，这个时候可以使用 Redis 缓存脚本的功能，在Redis 中脚本会通过 SHA-1 签名算法加密脚本，然后返回一个标识字符串，可以通过这个字符串 执行加密后的脚本。这样的 一个好处在于，如果脚本很长，从 客户端传输可能需要很长的时间，那么使用标识字符串，则只需要传递 32 位字符串即可，这样就能提高传输的效率，从而提高性能。

​	先使用**script load script**命令加载脚本，这个脚本的返回值是一个 SHA-1 签名过后的标识字符串，通过这个字符串可以使用命令执行签名后的脚本，命令的格式是:

```
evalsha shastring keynum [keyl key2 key3 . .. ] [paraml param2 param3. .. ]
```

例子如下图，对脚本签名后就可以使用 SHA-1 签名标识运行脚本了。：

![image-20181217113434054](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217113434054-5017674.png)

##### spring操作lua脚本：![image-20181217113833851](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217113833851-5017913.png)

​	如果要存储对象，可以使用Spring提供的 RedisScript接口，它还是提供了一个实现类DefaultRedisScript。![image-20181217114716851](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217114716851-5018436.png)

### 7.2 执行Lua文件

​	当逻辑比较复杂的时候，就需要通过执行一个Lua文件来实现功能，如下面的Lua文件test.lua：

```Lua
redis.call('set',KEYS[1],ARGV[1])
redis.call('set',KEYS[2],ARGV[2])
local n1=tonumber(reids.call('get',KEYS[1]))
local n2=tonumber(reids.call('get',KEYS[2]))
if n1 > n2 then
    return 1
end
if n1 == n2 then
    return 0
end
if n1 < n2 then
    return 2
end
```

​	上面的脚本表示：输入两个数字 n1,n2，先按键保存两个数字，然后去比较这两个数字的大小，不同的比较结果返回不同的值，下面是测试代码：

```lua
redis-cli --eval test.lua keyl key2 , 2 4
```

要注意文件和命令位置，此外，还要注意命令，**执行的命令键和参数是使用逗号分隔的，而键之间用空格分开。**在本例中 key2和参数之间是用逗号分隔的，而==这个逗号前后的空格是不能省略的==，这是要非常注意 的地方， 一旦左 边的空格被省略了，那么 Redis就会认为“key2,”是一个键， 一旦右边的空格被省略了， Redis就会认为“,2”是一个键。



参照：《Java EE互联网轻量级框架整合开发 SSM框架（Spring MVC+Spring+MyBatis）和Redis实现》