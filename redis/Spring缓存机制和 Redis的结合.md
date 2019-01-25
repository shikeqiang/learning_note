# Spring缓存机制和 Redis的结合

## 1.Redis 和数据库的结合

​	Redis和数据库需要保持数据的一致性，两者有可能出现数据不同步的状况，如下图：

![image-20181217174812562](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217174812562-5040092.png)

​	Tl 时刻以键 keyl 保存数据到 Redis, T2 时刻刷新进入数据库，但是 T3时刻发生了其他业务**需要改变数据库同一条记录的数据，但是采用了 key2 保存到 Redis 中，然后又写入了更新数据到数据库中，此时在 Redis 中 keyl 的数据是脏数据，和数据库的数据并不一致。**

### 1.1 Redis 和数据库读操作

​	**数据缓存往往会在Redis上设置超时时间，当设置Redis的数据超时后， Redis就没法读出数据了，这个时候就会触发程序读取数据库， 然后将读取的数据库数据写入 Redis (此时会给 Redis重设超时时间 )，这样程序在读取的过程中就能按一定的时间间隔刷新数据了，**读取数据的流程如图：![image-20181217175355238](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217175355238-5040435.png)

​	**设置过期时间，这样每当读取 Redis数据超过过期时间， Redis就不能读到超时数据了，只能重新从 Redis 中读取，保证了一定的实时性，也避免了多次访问数据库造成的系统性能低下的情况。**

### 1.2 Redis 和数据库写操作

​	**写操作要考虑数据一致 的问题，尤其是那些重要的业务数据，所以首先应该考虑从数据库中读取最新的数据，然后对数据进行操作，最后把数据写入 Redis 缓存中，如图：**![image-20181217175754247](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181217175754247-5040674.png)

​	**==写入业务数据，先从数据库中读取最新数据，然后进行业务操作，更新业务数据到数据库后，再将数据刷新到 Redis 缓存中，这样就完成了一次写操作。==这样的操作就能避免将脏数据写入数据库中，这类问题在操作时要注意。**	

## 2.使用 Spring 缓存机制整合 Redis

### 1.测试环境(纯注解)

​	**POJO：定义一个角色类Role，要实现Serializable接口；**

​	**mapper.xml文件RoleMapper.xml：定义增删改查的方法；**

​	**mybatis角色接口RoleDao：定义一些角色类相关的方法，比如增删改查找等，注解@Repository表示它是一个持久层的接口通过扫描和注解联合定义 DAO 层，就完成了映射器方面的内容。。**

​	**定义角色服务接口RoleService：也是定义一些增删改查找的方法；**

​	**mybatis-config.xml，mybatis配置文件，主要是配置扫描mapper.xml文件所在的路径**：

```xml
<?xml version=” 1. 0 ” eηcoding=”UTF-8 ”?> 
<!DOCTYPE configuration
	PUBLIC "-//mybatis .org//DTD Config 3 .0//EN”
	”http://mybatis.org/dtd/mybatis-3- config.dtd”> 
<configuratiqn>
<mappers>
<mapper resource=” com/ssm/chapter21/mapper/RoleMapper.xml” /> 
</mappers>
</configuration>
```

**​	Java配置类：**

```java
@Configuration
//定义 Spring 扫描的包
@ComponentScan (”com.*” )
//使用事务驱动管理器
@EnableTransactionManagement
//实现接口 TransactionManagementConfigurer，这样可以配置注解驱动事务
public class RootConfig implements TransactionManagementConfigurer {
	private DataSource dataSource = null;
	/**
* 配置数据库
* @return 数据连接池 */
	@Bean(name = ” dataSource ”)
	public DataSource initDataSource () {
		if (dataSource != null) {
			return dataSource;
		}
		Properties props= new Properties() ;
		props .setProperty (”driverClassName”,”com.mysql.jdbc.Driver”);
		props .setProperty (”url”,”jdbc:mysql://localhost:3306/role”); 
		props.setProperty (”username”,”root”);
		props.setProperty (”password”,”123456”);
		try {
			dataSource = BasicDataSourceFactory . createDataSource(props);
		}catch (Exception e) {
			e.printStackTrace() ;
		}
		return dataSource;
	}
	//配置SqlSessionFactoryBean
	@Bean(name = ”sqlSessionFactory”)
	public SqlSessionFactoryBean initSqlSessionFactory() {
		SqlSessionFactoryBean sqlSessionFactory = new SqlSessionFactoryBean();
		sqlSessionFactory.setDataSource(initDataSource()) ;
		//配置 MyBatis 配置文件
		Resource resource =new ClassPathResource("mybatis/mybatis-config.xml”);
		sqlSessionFactory . setConfigLocation(resource) ;
		return sqlSessionFactory;
	}
	//通过自动扫描，发现 MyBatis  Mapper 接口
	@Bean
	public MapperScannerConfigurer initMapperScannerConfigurer() {
		MapperScannerConfigurer msc = new MapperScannerConfigurer();
		//扫描包
		msc.setBasePackage (”com.*”);
		msc.setSqlSessionFactoryBeanName (” sqlSessionFactory” );
		//区分注解扫描
		msc.setAnnotationClass(Repository .class);
		return msc;
	}
	//实现接口方法，注册注解事务 ， 当@Transactional 使用的时候产生数据库事务
	@Override
	@Bean(name = ”annotationDrivenTransactionManager”)
	public PlatformTransactionManager annotationDrivenTransactionManager (){
		DataSourceTransactionManager transactionManager new DataSourceTransactionManager();
		transactionManager.setDataSource(initDataSource());
		return transactionManager;
	}
}
```

### 2.Spring 的缓存管理器

​	在 Spring 项目中它提供了**==接口 CacheManager 来定义缓存管理器==** ， 这样各个不同的缓存就可以实现它来提供管理器的功能了，而在 spring-data-redis.jar包中**==实现 CacheManager接口的则是 RedisCacheManager，==** 因此要定义 RedisCacheManager 的 Bean， 不过在此之前要先定义 RedisTemplate。	

```java 
@Configuration
@EnableCaching
public class RedisConfig {
	@Bean(name = ”redisTemplate”)
		public RedisTemplate initRedisTemplate () {
		JedisPoolConfig poolConfig = new JedisPoolConfig( );
		//最大空闲数
		poolConfig.setMaxidle(50);
		//最大连接数
		poolConfig.setMaxTotal(lOO );
		//最大等待毫秒数
		poolConfig.setMaxWaitMillis(20000) ;
		//创建 Jedis 连接工厂
		JedisConnectionFactory connectionFactory= new JedisConnectionFactory(poolConfig) ;
		connectionFactory.setHostName(”localhost”);
		connectionFactory.setPort (6379);
		//调用后初始化方法，没有它将抛出异常 
		connectionFactory.afterPropertiesSet();
		//自定 Redis 序列化器
		RedisSerializer jdkSerializationRedisSerializer =new JdkSerializationRedisSerializer ();
		RedisSerializer stringRedisSerializer= new
				StringRedisSerializer() ;
		//定义 RedisTemplate， 并设置连接工程
		RedisTemplate redisTemplate = new RedisTemplate ();
		redisTemplate.setConnectionFactory(connectionFactory);
		//设置序列化器
		redisTemplate.setDefaultSerializer(stringRedisSerializer);
		redisTemplate.setKeySerializer(stringRedisSerializer);
		redisTemplate.setValueSerializer(jdkSerializationRedisSerializer);
		redisTemplate.setHashKeySerializer (stringRedisSerializer);
		redisTemplate.setHashValueSerializer(jdkSerializationRedisSerializer);
		return redisTemplate ;
		@Bean(name =”redisCacheManager”)
		public CacheManager initRedisCacheManager(@Autowired RedisTemplate redisTempate) {
			RedisCacheManager cacheManager= new RedisCacheManager(redisTempate) ;
			//设置超时时间为 10 分钟，单位为秒 
            cacheManager.setDefaultExpiration(600) ; 
            //设置缓存名称
			List<String> cacheNames = new ArrayList<String>() ;
			cacheNames.add(”redisCacheManager”);
		}
		cacheManager.setCacheNames(cacheNames);
		return cacheManager;
	}
}
```

​	**@EnableCaching 表示 Spring IoC 容器启动了缓存机制。**对于 RedisTemplate 的定义实 例和 XML 的方式差不多。**==注意，在创建 Jedis 连接工厂 (JedisConnectionFactory)后，要自己调用Jedis连接工厂的afterPropertiesSet方法，因为这里不是单独自定义一个 SpringBean，而是在 XML 方式中是单独 自定义的。==**这个类实现了 InitializingBean接口，按照 SpringBean 的生命周期， 我们知道它会被 SpringIoC容器自己调用，而这里的注解方式没有定义 SpringBean，因此需要自己调用 ，这也是使用注解方式的不便之处一一需要了解其内部的实现。 

​	字符串定义了 key (包括 hash 数据结构)，而值则使用了序列化，这样就能够保存 Java 对象了。**缓存管理器 RedisCacheManager 定义了默认的超时时间为 10 分钟，这样就可以在一定的时间间隔后重新从数据库中读取数据了，而名称则定义为 redisCacheManager，** 名称 是为了方便后面注解 引用的 。

​	下面是使用xml文件的方式来配置缓存管理器：![image-20181218095039117](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181218095039117-5097839.png)

### 3.缓存注解简介

##### 	缓存相关注解：![image-20181218100352912](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181218100352912-5098632.png) 

​	**注解@Cacheable 和 @CachePut 都可以保存缓存键值对**，只是它们的方式略有不同，请注意二者的区别 ，它们只能运用于有返回值的方法中，而**删除缓存 key 的@CacheEvict可以用在 void 的方法上 ，因为它 并不需要去保存任何值 。**

​	**上述注解都能标注到类或者方法之上，如果放到类上，则对所有的方法都有效;如果放到方法上，则只是对方法有效。在大部分情况下，会放置到方法上。因为@Cacheable和@CachePut可以配置的属性接近，所以把它们归为一类去介绍，@Caching 因为不常用。==一般而言，对于查询，我们会考虑使用@Cacheable;对于插入和修改，我们会考虑使用@ CachePut;对于删除操作，我们会考虑使用@CacheEvict。==**

### 4.注解@Cacheable 和@ CachePut	

##### 	@Cacheable 和@ CachePut相关属性

![image-20181218101459661](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181218101459661-5099299.png)

​	由上表可见，value 是 一个数组，可以引用多个缓存管理器，而对于 key则是缓存中的键，它支持 Spring表达式，通过 Spring表达式就可以自定义缓存的 key。

##### 	Spring表达式值的引用：

![image-20181218102237450](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181218102237450-5099757.png)![image-20181218102251830](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181218102251830-5099771.png)

RoleService 接口的实现类--RoleServicelmpl：

```java 
@Service
public class RoleServiceimpl implements RoleService {
	//角色 DAO，方便执行 SQL
	@Autowired
	private RoleDao roleDao = null ;
/**
*使用@Cacheable 定义缓存策略 
*当缓存中有值，则返回缓存数据，否则访问方法得到数据 
*通过 value 引用缓存管理器，通过 key 定义键
* @param id 角色编号
* @return 角色
*/
	@Override
	@Transactional(isolation =Isolation.READ.COMMITTED,propagation = Propagation.REQUIRED)
	@Cacheable(value = ”redisCacheManager”, key = ”’redis_role_’+#id”) 
    public Role getRole(long id) (
		return roleDao .getRole(id);
}
/**
* 使用@CachePut 则表示无论如何都会执行方法，最后将方法的返回值再保存到缓存中 
* 使用在插入数据的地方，则表示保存到数据库后，会同期插入 Redis 缓存中
* @param role 角色对象
* @return 角色对象(会回填主键)
*/
@Override
@Transactional(isolation =Isolation.READ_COMMITTED,propagation = Propagation.REQUIRED)
@CachePut(value =”redisCacheManager”, key=”’redis_role_’+#result.id”)
public Role insertRole(Role role) {
	roleDao.InsertRole(role) ;
	return role ;
}
/**
*使用@CachePut，表示更新数据库数据的同时，也会同步更新缓存 
* @param role 角色对象
* @return 影响条数
*/
@Override
@Transactional(isolation =Isolation.READ_COMMITTED,propagation= Propagation.REQUIRED)
@CachePut(value = ”redisCacheManager”, key = ”’redis role ’+#role.id”) 
publiC Role updateRole(Role role) {
	roleDao.updateRole(role) ; 
  	return role ;
	}
}
```

​	**这里先说一下@Transactional事务注解中的隔离级别isolation：Isolation.READ_COMMITTED表示读已提交。解决脏读，存在不可重复读与幻读。还有传播行为Propagation：REQUIRED表示指定的方法必须在事务内执行。若当前存在事务，就加入到当前事务中;若当前没有事务，则创建一个新事务。这种传播行为是最常见的选择，也是 Spring 默认的事 务传播行为。**

**因为getRole 方法是一个查询方法，所以使用@Cacheable 注解，这样在 Spring 的调用 中，它就会先查询 Redis， 看看是否存在对应的值。==它是通过注解中的 key属性，它配置的是’redis_role_’+#id，这样 Spring EL就会计算返回一个 key，比如参数 id为 IL， 其 key计算结果就为 redis_role_I。==以一个 key去访问 Redis，如果有返回值，则不再执行方法，如果没有则访问方法 ，返回角色信息，然后通过key去保存数据到 Redis 中 。**

​	先执行 insertRole 方法才能把对应的信息保存到 Redis 中，所以采用的是注解@CachePut。**由于主键是由数据库生成，所以无法从参数中读取，但是可以从结果中读取， 那么#result.id 的写法就会返回方法返回的角色 id。**而这个角色 id是通过数据库生成，**然后由MyBatis进行回填得到的 ，这样就可以在 Redis 中新增一个 key， 然后保存对应的对象了 。** 

​	对于 updateRole 方法而言，采用的是注解@CachePut，**由于对象有所更新，所以要在方法之后更新 Redis 的数据，以保证数据的一致性。**==这里直接读取参数的 id，所以表达式写为#role.id，这样就可以引入角色参数的 id 了。在方法结束后，它就会去更新 Redis对应的 key 的值了==。

### 5.注解@ CacheEvict

##### 	注解@CacheEvict 主要是为了移除缓存对应的键值对，主要对于那些删除的操作，相关属性：![image-20181218112126226](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181218112126226-5103286.png)

​	value 和 key 与之前的@Cacheable 和@CachePut 是一致的。**==属性 allEntries 要求删除缓存服务器中所有的缓存，这个时候指定的 key 将不会生效，所以这个属性要慎用 。==** beforeInvocation 属性指定缓存在方法前或者方法后移除。beforelnvocation的名字暴露了 Spring的实现方式一一反射方法， 它是通过AOP去实现的，数据库事务 的方式也是如此 。和@Transactional 一样， beforelnvocation 提供注解和配置项，进一步简化了开发 。使用@CacheEvict移除缓存，如下面的代码：

```java
/**
*使用@CacheEvict删除缓存对应的 key 
@param id 角色编号
@return 返回删除记录数
*/
@Override
@Transactional(isolation = Isolation.READCOMMITTED ,propagation = Propagation.REQUIRED)
@CacheEvict(value = ”redisCacheManager”, key = ”’redis_role_’+#id") 
public int deleteRole(Long id) {
	return roleDao.deleteRole(id) ;
}
```

​	**在方法执行完成后会移除对应的缓存，也就是还可以从方法内读取到缓存服务器中的数据。如果属性 beforelnvocation 声明为 true，则在方法前删除缓存数据 ，这样就不能在方法中读取缓存数据了，只是这个值 的默认值为 false，所以默认的情况下只会在方法后执行删除缓存。**

### 6.不适用续存的方法

​	RoleServicelmpl类里面还有个查找角色的方法,findRoles:

```Java
@Override
@Transactional(isolation =Isolation.READ_COMMITTED, propagation = Propagation.REQUIRED)
public List<Role> findRoles(String roleName, String note) {
	return roleDao.findRoles(roleName,note) ;
}
```

​	注意，使用缓存的前提一一高命中率，由于这里根据角色名称和备注查找角色信息，该方法的返回值会根据查询条件而多样化，导致其不确定和命中率低下，对于这样的场景，使用缓存并不能有效提高性能，所以
这样的场景，就不再使用缓存了。**即对于命令中率不高的场景，就不建议使用缓存。**



参照：《Java EE互联网轻量级框架整合开发 SSM框架（Spring MVC+Spring+MyBatis）和Redis实现》