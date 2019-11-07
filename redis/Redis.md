# Redis原理

# 1.获取一个RedisTemplate对象

​	一般都是通过连接池连接Redis的，要在spring中使用Redis，首先，我们需要配置JedisPoolConfig对象，如下图：![image-20181215173333350](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181215173333350.png)

在使用 Spring提供的 RedisTemplate之前需要配置 Spring所提供的连接工厂，在 Spring  Data Redis 方案中它提供了 4 种工厂模型：

​	• **JredisConnectionFactory。**
​	**• JedisConnectionFactory。**
​	**• LettuceConnectionFactory。**
​	• **SrpConnectionFactory**。

![image-20181215173511658](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181215173511658.png)

如果想换成其他的连接工厂，修改bean标签里面的class属性为对应的工厂模型类即可。

​	普通的连接使用没有办法把 Java对象直接存入 Redis，而需要我们自己提供方案，这时往往就是将对象序列化，然后使用 Redis 进行存储，而取回序列化的内容后，在通过转换转变为 Java 对象， Spring 模板中提供了封装的方案，在它内部提供了 **RedisSerializer 接口**(org.spring企amework.data.redis.serializer.RedisSerializer)和一些实现类，如下图：![image-20181215173836896](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181215173836896.png)

下面几种方法实现了RedisSerializer接口：

- **GenericJackson2JsonRedisSerializer，通用的使用 Json2.jar 的包，将 Redis 对象的序 列化器。**

- **Jackson2JsonRedisSerializer<T>，通过 Jackson2.jar包提供的序列化进行转换。** 

- **JdkSerializationRedisSerializer<T>， 使用 JDK 的序列化器进行转化。** 

- **OxmSerializer， 使用 SpringO/X 对象 Object和 XML相互转换。** 

- **StringRedisSerializer， 使用字符串进行序列化 。** 

- **GenericToStringSerializer，通过通用 的字符串序列化进行相互转换。**

  使用它们就能够帮助我们把对象通过序列化存储到 Redis 中，也可以把 Redis存储的内容转换为 Java对象，为此 Spring提供的 RedisTemplate还有两个属性。

  **• keySerializer  	一键序列器 。**
  **• valueSerializer   一值序列器。**![image-20181215202244784](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181215202244784.png)**这样就配置了 一个 RedisTemplate 的对象，并且 spring data redis 知道会用对应 的序列化器去转换 Redis 的键值。**

## 2.Spring 对 Redis API 的基本封装

Java有多种 Redis 的 API，如：Jedis Jredis、Lettuce 等。为了融合这些不同的 APL Spring 给出 一个对底层操作的接口 RedisConnection,通过这个接口就消除了各种连接 API的差异，提供统一的接口规范来简化操作，如图：![image-20181215202657201](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181215202657201.png)

Spring 对 Java 多种 Redis 连接 API 进行封装，而各个连接的实现类都继承抽象类**==AbstractRedisConnection==**，而这个抽象类实现了 **==RedisConnection==** 接口。所以对于使用者而言，只需要知道 RedisConnection 接口的 API 就可以消除各个 API 的差异了。

Spring 会提供创建这个接口对象的工厂----RedisConnectionFactory。其实就是上面第一点的创建工厂连接，如图：![image-20181215202914829](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181215202914829.png)

## 3.Spring 对 Redis 命令的封装

 	Spring对Redis的6种数据类型的命令进行了更深层次封装，封装为6个接口:

**​	• ValueOperations 一键值对操作接口 。**
​	**• HashOperations 一哈希操作接口。**
​	**• ListOperations一一链表操作接口 。**
​	**• SetOperations 一一无序集合操作接口 。**
​	**• ZSetOperations一一有序集合操作接口。**
​	**• HyperLogLogOperations一一基数操作接口 。**	

Spring也会为它们提供默认的实现类， 在大部分情况下只要使用Spring提供的实现类即可，这些实现类是: **DefaultValueOperations、DefaultHashOperations、DefaultListOperations、 DefaultSetOperations、 DefaultZSetOperations和 DefaultHyperLogLogOperations。** 通过名字即可知道这些实现类对应的操作命令是什么。![image-20181215204244164](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181215204244164.png)

如上图，只是列举了键值对和哈希这两种数据类型的命令。 

下面我们来看看spring是怎么保存数据的，来看**==DefaultValueOperations==**类的 set方法：

```Java
public void set(K key, V value) {
		final byte[] rawValue = rawValue(value);
		execute(new ValueDeserializingRedisCallback(key) {
			protected byte[] inRedis(byte[] rawKey, RedisConnection connection) {
				connection.set(rawKey, rawValue);
				return null;
			}
		}, true);
	}
```

​	由上面的代码可以知道，Spring把 **key和 value转化为 rawKey和 rawValue**，然后通过 RedisConnection的set方法发给 Redis 存储。其实就是将key和value转换为字节，由Spring 通过 **AbstractOperations** 抽象类进行转化，它有两个方法 rawKey 和 rawValue，如下面的代码：

```Java
@SuppressWarnings("unchecked")
	byte[] rawKey(Object key) {
		Assert.notNull(key, "non null key required");
		if (keySerializer() == null && key instanceof byte[]) {
			return (byte[]) key;
		}
		return keySerializer().serialize(key);
	}
	
	@SuppressWarnings("unchecked")
	byte[] rawValue(Object value) {
		if (valueSerializer() == null && value instanceof byte[]) {
			return (byte[]) value;
		}
		return valueSerializer().serialize(value);
	}
```

​	Spring使用了 **keySerializer和 valueSerializer两个序列化器**对键值对进行序列化，这个序列化就把键值对转化为了二进制数组(byte[])，通过 RedisConnection传达给 Redis服务器。 

对于这个序列化 Spring 也提供了接口RedisSerializer：

```Java
public interface RedisSerializer<T> {
	/**
	 * Serialize the given object to binary data. 
	 * @param t object to serialize
	 * @return the equivalent binary data
	 */
	byte[] serialize(T t) throws SerializationException;

	/**
	 * Deserialize an object from the given binary data.
	 * @param bytes object binary representation
	 * @return the equivalent object instance
	 */
	T deserialize(byte[] bytes) throws SerializationException;
}
```

​	这个接口只有两个方法， **第一个方法就是是将对象(这个对象要求实现 Serializable接口)通过序列化转化为二进制数组;第二个方法是将已经序列化的二进制数据转化为对应的对象。**只要实现了这个接口，并实现这两个方法，就可以自定义序列化器了，然后通过配置就能在 Spring 的上下文中使用自定义的序列化器。其实这个就是上面第一点提到的序列化与反序列化。

## 4.Spring对 Redis操作的封装

​	在 Spring 中提供了对应的操作接口BoundKeyOperations，这是一个公共接口，它的方法可以给 6 种数据类型操作共享：

```Java
public interface BoundKeyOperations<K> {
	/**
	 * Returns the key associated with this entity.
	 * @return key associated with the implementing entity
	 */
	K getKey();
	/**
	 * Returns the associated Redis type.
	 * @return key type
	 */
	DataType getType();
	/**
	 * Returns the expiration of this key. 
	 * @return expiration value (in seconds)
	 */
	Long getExpire();
	/**
	 * Sets the key time-to-live/expiration. 
	 * @param timeout expiration value
	 * @param unit expiration unit
	 * @return true if expiration was set, false otherwise
	 */
	Boolean expire(long timeout, TimeUnit unit);
	/**
	 * Sets the key time-to-live/expiration.
	 * @param date expiration date
	 * @return true if expiration was set, false otherwise
	 */
	Boolean expireAt(Date date);
	/**
	 * Removes the expiration (if any) of the key.
	 * @return true if expiration was removed, false otherwise
	 */
	Boolean persist();
	/**
	 * Renames the key. <br>
	 * <b>Note:</b> The new name for empty collections will be propagated on add of first element. 
	 * @param newKey new key
	 */
	void rename(K newKey);
}
```

这是一个最基础的接口，为了实现各个数据类型不同的操作，在此接口上 Spring扩展出了其他的接口: 

- **BoundValueOperations一一对于键值对的操作，这是 Redis 最基础的数据类型。** 
- **BoundHashOperations一一对于哈希数据的操作。** 
- **BoundListOperations一一对于链表( List)数据的操作。** 
- **BoundSetOperations一一对于集合( Set)数据的操作。** 
- **BoundZSetOperations一一对于有序集合( ZSet)数据的操作。** 
- **HyperLogLogOperations一一基数统计统计操作。**

这样 Spring通过操作就可以把各个数据类型的命令封装到各个操作里面，提供统一的操作接口给调用者使用。 Spring 也对它们提供对应的默认实现类。这里只探讨BoundValueOperations接口的实现类**DefaultBoundValueOperations**来了解 Spring 对 Redis 操作的封装了。  ![image-20181215210723278](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181215210723278.png)

**DefaultBoundValueOperations的构造方法如下：**

```Java
public DefaultBoundValueOperations(K key, RedisOperations<K, V> operations) {
		super(key, operations);
		this.ops = operations.opsForValue();
	}
```

​	key 的传递限制了这个类在操作对应的 Redis 的键值对，而**属性 ops 是一个操作** 。 **==operations.opsForValue()是获取一个 ValueOperations 接口对象==**，可见它最终还是通过对命令 的封装进行操作的。这样它就能够操作命令封装的类对 Redis 的命令操作了。

==注意==， Spring 提供了一个接口**TypedTuple** 来操作有序集合，因为有序集合的元素 是由分数( score)和值( value)组成的，分数是用于排序的 一个双精度数字，这个接口要 求实现两个方法 : 

```Java
public interface TypedTuple<V> extends Comparable<TypedTuple<V>> {
		V getValue();
		Double getScore();
	}
```

​	getValue 方法是获取值 ，而 getScore 方法是返回一个分数，用于排序 。由于它继承了Comparable接口，所以还需要实现一个 compareTo 方法。 

## 5.RedisTemplate

​	Spring 也为 Redis 提供了 一个 RedisTemlate 模板，通过它就可以快速操作 Redis。 在这个类里面定义了很多属性，如下：

```Java
public class RedisTemplate<K, V> extends RedisAccessor implements RedisOperations<K, V>, BeanClassLoaderAware {
	private boolean enableTransactionSupport = false;	//是否支持事务
	private boolean exposeConnection = false;	 //是否暴露连接，用于代理访问的形式时
	private boolean initialized = false;	//初始化状态
	private boolean enableDefaultSerializer = true;	//默认的序列化器是否开启
	private RedisSerializer<?> defaultSerializer;	//默认序列化器
	private ClassLoader classLoader;	//类加载器

	private RedisSerializer keySerializer = null;	//键序列化器
	private RedisSerializer valueSerializer = null;		//值序列化器
	private RedisSerializer hashKeySerializer = null;	//hash数据结构键序列化器
	private RedisSerializer hashValueSerializer = null;	//hash数据结构值序列化器
    //字符串序列化器
	private RedisSerializer<String> stringSerializer = new StringRedisSerializer();
	private ScriptExecutor<K> scriptExecutor;//Lua 脚本执行器

	// cache singleton objects (where possible)
	private ValueOperations<K, V> valueOps;		//字符串操作
	private ListOperations<K, V> listOps;		//链表操作
	private SetOperations<K, V> setOps;			//集合操作
	private ZSetOperations<K, V> zSetOps;		//有序集合操作
	private HyperLogLogOperations<K, V> hllOps;	//HyperLogLog 操作
```

​	通过上面的代码我们知道，可以通过设置 key和 value 的序列化器去控制其序列化，其次也可以获得各种操作来执行各种 Redis 的命令。还有一 个scriptExecutor来支持 Lua脚本的执行。 

##### RedisTemplate 主要有 3 大类的操作 :

- **数据类型的公共命令，比如 expire、 delete、 watch、 unwatch 等命令。** 
- **获取对应的操作类，比如 ValueOperations 对象。** 
- **执行多个命令或者其他用户回调模板。** 

### 5.1 RedisTemplate 公共命令

​	先来看看公共命令spring的封装，以删除键值对的delete为例：

```Java
public void delete(K key) {
		final byte[] rawKey = rawKey(key);
		execute(new RedisCallback<Object>() {
			public Object doInRedis(RedisConnection connection) {
				connection.del(rawKey);
				return null;
			}
		}, true);
	}
```

​	这里面重点主要是**==execute==**方法，参数是RedisCallback接口，这个接口里面只有**doInRedis**这个方法，这里用的是匿名内部类的形式。在 RedisCallback 接口对象里，它采用了自己封装的 RedisConnection 接口操作 Redis 命令 。在 RedisTemplate 中， 有很多个execute 方法，他们参数不同，这里只列举一个：

```Java
	public <T> T execute(RedisCallback<T> action, boolean exposeConnection, boolean pipeline) {
		Assert.isTrue(initialized, "template not initialized; call afterPropertiesSet() before using it");
		Assert.notNull(action, "Callback object must not be null");
        //获取 Redis 连接池
		RedisConnectionFactory factory = getConnectionFactory();
		RedisConnection conn = null;
		try {
            //获取连接，判断是否存在/支持事务
			if (enableTransactionSupport) {
				// only bind resources in case of potential transaction synchronization
				conn = RedisConnectionUtils.bindConnection(factory, enableTransactionSupport);
			} else {
				conn = RedisConnectionUtils.getConnection(factory);
			}
			boolean existingConnection = TransactionSynchronizationManager.hasResource(factory);
			RedisConnection connToUse = preProcessConnection(conn, existingConnection);
			boolean pipelineStatus = connToUse.isPipelined();
            //是否采用流水线
			if (pipeline && !pipelineStatus) {
				connToUse.openPipeline();
			}
            //是否采用代理访问
			RedisConnection connToExpose = (exposeConnection ? connToUse : createRedisConnectionProxy(connToUse));
            //调用 RedisCallback 接口 的 doinRedis 方法
			T result = action.doInRedis(connToExpose);
			// close pipeline
			if (pipeline && !pipelineStatus) {
				connToUse.closePipeline();
			}
			// TODO: any other connection processing?即事后方法
			return postProcessResult(result, connToUse, existingConnection);
		} finally {
            //是否支持事务，关闭事务
			if (!enableTransactionSupport) {
				RedisConnectionUtils.releaseConnection(conn, factory);
			}
		}
	}
```

​	**当用一个公共命令的时候， Spring会从 Redis 的连接池里获取连接，**所以在一个方法里面使用 RedisTemplate操作，比如下面的代码，在大部分情况下都会抛出异常 ：

```Java
redisTemplate .multi();
redisTemlate.exec();
```

**这是因为使用公共命令，==每次 RedisTemplate 都会尝试在连接池里面拿到一条空闲的连接==，而 redisTemplate.multi()和 redisTemlate.exec();执行的时候，在大部分的情况下都不是同一条连接，因此会在 redisTemlate.exec();执行过程中发生异常，因为它内部的 Redis 连接没有执行事务的开启(该连接在此之前没有执行 multi命令)。**

### 5.2 获取操作类

​	RedisTemplate 的属性定义了 Redis 的 6种数据结构的操作类，只要**==通过opsForXXX这样的方法就可以得到各种命令的封装类==**。比如要对字符串操作，就可以使用: 

```Java
redisTemplate.opsForValue();	//得到操作字符串的封装类
```

​	得到命令**封装类**。**如果要对某个键值对操作，那么也可以通过 boundXXXOps 来获取操作类，比如对字符串的 boundValueOps 方法，这样就可以得到一个字符串的操作类。**它们的底层也是通过使用 RedisTemplate 的 execute方法去执行的，所以对于它们的操作还是类似 RedisTemplate 那样，执行一次命令就尝试在连接池里获取新的连接。 即其实通过 boundXXXOps 来获取操作类，最终还是调用xxxOperations的方法，如下代码：

```java
//第二行代码是例子，下面是一步步调用的方法
redisTemplate.boundValueOps("name").set("baichen");
public BoundValueOperations<K, V> boundValueOps(K key) {
		return new DefaultBoundValueOperations<K, V>(key, this);
	}

public DefaultBoundValueOperations(K key, RedisOperations<K, V> operations) {
		super(key, operations);
		this.ops = operations.opsForValue();
	}
public ValueOperations<K, V> opsForValue() {
		if (valueOps == null) {
			valueOps = new DefaultValueOperations<K, V>(this);
		}
		return valueOps;
	}

	private ValueOperations<K, V> valueOps;
```

​	第一行代码的执行过程是：先通过boundValueOps获取字符串的操作类，然后这个操作类会再调用ValueOperations这个类，然后这个类又返回RedisTemplate里面定义好的一些单例模式的操作类，然后这些类里面封装了相应的方法，如上面说的，最后都是调用DefaultValueOperations的方法。



具体的操作实例参见：https://mp.csdn.net/postedit/83745680





参照：《Java EE互联网轻量级框架整合开发 SSM框架（Spring MVC+Spring+MyBatis）和Redis实现》