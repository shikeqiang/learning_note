# 一、Mybatis编程步骤

1. 创建 SqlSessionFactory 对象。
2. 通过 SqlSessionFactory 获取 SqlSession 对象。
3. 通过 SqlSession 获得 Mapper 代理对象。
4. 通过 Mapper 代理对象，执行数据库操作。
5. 执行成功，则使用 SqlSession 提交事务。
6. 执行失败，则使用 SqlSession 回滚事务。
7. 最终，关闭会话。

> 当 Java 代码通过 JDBC 的 PreparedStatement 向 DB 发送一条 SQL 语句时，DBMS 会首先编译 SQL 语句，然后**将编译好的 SQL 放入到 DBMS 的数据库缓存池中再执行**。当 DBMS 再次接收到对该数据库操作的 SQL 时，先从 DB 缓存池中查找该语句是否被编译过，若被编译过，则直接执行，否则先编译后将编译结果放入 DB 缓存池，再执行。

# 二、`#{}` 和 `${}` 的区别

`${}` 是 Properties 文件中的变量占位符，它可以用于 XML 标签属性值和 SQL 内部，属于**字符串替换**。例如将 `${driver}`会被静态替换为 `com.mysql.jdbc.Driver` ：

```xml
<dataSource type="UNPOOLED">
    <property name="driver" value="${driver}"/>
    <property name="url" value="${url}"/>
    <property name="username" value="${username}"/>
</dataSource>
```

`${}` 也可以对传递进来的参数**原样拼接**在 SQL 中(有sql注入的风险)。字符串拼接是将参数值以硬编码的方式直接拼接到了 SQL 语句中。**字符串拼接就会引发两个问题:SQL 注入问题与没有使用预编译所导致的执行效率低下问题。**

代码如下：

```xml
<select id="getSubject3" parameterType="Integer" resultType="Subject">
    SELECT * FROM subject
    WHERE id = ${id}
</select>
```

> ==resultType 属性并非指查询结果集最后的类型，而是每查出 DB 中的一条记录，将该记录封装成为的对象的类型。==resultMap 是对 resultType的增强。

**#{}是 SQL 的参数占位符，Mybatis 会将 SQL 中的 `#{}` 替换为 `?` 号，在 SQL 执行前会使用 PreparedStatement 的参数设置方法，按序给 SQL 的 `?` 号占位符设置参数值**，比如 `ps.setInt(0, parameterValue)` 。 所以，==#{}是**预编译处理**，==可以有效防止 SQL 注入，提高系统安全性。

另外，`#{}` 和 `${}` 的取值方式非常方便。例如：`#{item.name}` 的取值方式，为使用反射从参数对象中，获取 `item` 对象的 `name` 属性值，相当于 `param.getItem().getName()` 。

> #{ }:对指定参数类型属性值的引用。其底层是通过反射机制，调用实体类相关属性的 get 方法来获取值的。因为底层使用的是反射，所以这里使用的是类的属性名，而非表的字段名。

# 三、实体类中的属性名与表中的字段名绑定

第一种， 通过在查询的 SQL 语句中定义字段名的别名，让字段名的别名和实体类的属性名一致。代码如下：

```xml
<select id="selectOrder" parameterType="Integer" resultType="Order"> 
    SELECT order_id AS id, order_no AS orderno, order_price AS price 
    FROM orders 
    WHERE order_id = #{id}
</select>
```

- 1、数据库的关键字，统一使用大写，例如：`SELECT`、`AS`、`FROM`、`WHERE` 。
- 2、每 5 个查询字段换一行，保持整齐。
- 3、`,` 的后面，和 `=` 的前后，需要有空格，更加清晰。
- 4、`SELECT`、`FROM`、`WHERE` 等，单独一行，高端大气。

第二种，是第一种的特殊情况。大多数场景下，数据库字段名和实体类中的属性名差，主要是前者为**下划线风格**，后者为**驼峰风格**。在这种情况下，可以直接配置如下，实现自动的下划线转驼峰的功能。

```xml
<setting name="logImpl" value="LOG4J"/>
    <setting name="mapUnderscoreToCamelCase" value="true" />
</settings>
```

第三种，通过 `<resultMap>` 来映射字段名和实体类属性名的一一对应的关系。代码如下：

```xml
<resultMap type="me.gacl.domain.Order" id=”OrderResultMap”> 
    <!–- 用 id 属性来映射主键字段 -–> 
    <id property="id" column="order_id"> 
    <!–- 用 result 属性来映射非主键字段，property 为实体类属性名，column 为数据表中的属性 -–> 
    <result property="orderNo" column ="order_no" /> 
    <result property="price" column="order_price" /> 
</resultMap>
<select id="getOrder" parameterType="Integer" resultMap="OrderResultMap">
    SELECT  order_no
    FROM orders 
    WHERE order_id = #{id}
</select>
```

# 四、XML 映射文件中常见的标签

SQL 映射文件只有很少的几个顶级元素（按照应被定义的顺序列出）：

- `cache` – 对给定命名空间的缓存配置。
- `cache-ref` – 对其他命名空间缓存配置的引用。
- `resultMap` – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。
- `parameterMap` – 已被废弃！老式风格的参数映射。更好的办法是使用内联参数，此元素可能在将来被移除。
- `sql` – 可被其他语句引用的可重用语句块。
- `insert` – 映射插入语句
- `update` – 映射更新语句
- `delete` – 映射删除语句
- `select` – 映射查询语句

## select

```xml
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
```

这个语句被称作 selectPerson，接受一个 int（或 Integer）类型的参数，并返回一个 HashMap 类型的对象，其中的键是列名，值便是结果行中的对应值。

注意参数符号：#{id}

这就告诉 MyBatis 创建一个预处理语句（PreparedStatement）参数，在 JDBC 中，这样的一个参数在 SQL 中会由一个“?”来标识，并被传递到一个新的预处理语句中，就像这样：

```Java
// 近似的 JDBC 代码，非 MyBatis 代码...
String selectPerson = "SELECT * FROM PERSON WHERE ID=?";
PreparedStatement ps = conn.prepareStatement(selectPerson);
ps.setInt(1,id);
```

```xml
<select
  id="selectPerson"
  parameterType="int"
  parameterMap="deprecated"
  resultType="hashmap"
  resultMap="personResultMap"
  flushCache="false"
  useCache="true"
  timeout="10"
  fetchSize="256"
  statementType="PREPARED"
  resultSetType="FORWARD_ONLY">
```

| 属性            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| `id`            | 在命名空间中唯一的标识符，可以被用来引用这条语句。           |
| `parameterType` | 将会传入这条语句的参数类的完全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器（TypeHandler） 推断出具体传入语句的参数，默认值为未设置（unset）。 |
| parameterMap    | 这是引用外部 parameterMap 的已经**被废弃**的方法。请使用内联参数映射和 parameterType 属性。 |
| `resultType`    | 从这条语句中返回的期望类型的类的完全限定名或别名。 注意如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身。可以使用 resultType 或 resultMap，但不能同时使用。 |
| `resultMap`     | 外部 resultMap 的命名引用。结果集的映射是 MyBatis 最强大的特性，如果你对其理解透彻，许多复杂映射的情形都能迎刃而解。可以使用 resultMap 或 resultType，但不能同时使用。 |
| `flushCache`    | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：false。 |
| `useCache`      | 将其设置为 true 后，将会导致本条语句的结果被二级缓存缓存起来，默认值：对 select 元素为 true。 |
| `timeout`       | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖驱动）。 |
| `fetchSize`     | 这是一个给驱动的提示，尝试让驱动程序每次批量返回的结果行数和这个设置值相等。 默认值为未设置（unset）（依赖驱动）。 |
| `statementType` | STATEMENT，PREPARED 或 CALLABLE 中的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| `resultSetType` | FORWARD_ONLY，SCROLL_SENSITIVE, SCROLL_INSENSITIVE 或 DEFAULT（等价于 unset） 中的一个，默认值为 unset （依赖驱动）。 |
| `databaseId`    | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。 |
| `resultOrdered` | 这个设置仅针对嵌套结果 select 语句适用：如果为 true，就是假设包含了嵌套结果集或是分组，这样的话当返回一个主结果行的时候，就不会发生有对前面结果集的引用的情况。 这就使得在获取嵌套的结果集的时候不至于导致内存不够用。默认值：`false`。 |
| `resultSets`    | 这个设置仅对多结果集的情况适用。它将列出语句执行后返回的结果集并给每个结果集一个名称，名称是逗号分隔的。 |

## insert, update 和 delete

数据变更语句 insert，update 和 delete 的实现非常接近：

```xml
<insert
  id="insertAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  keyProperty=""
  keyColumn=""
  useGeneratedKeys=""
  timeout="20">

<update
  id="updateAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">

<delete
  id="deleteAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">
```

| 属性               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| `id`               | 命名空间中的唯一标识符，可被用来代表这条语句。               |
| `parameterType`    | 将要传入语句的参数的完全限定类名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器推断出具体传入语句的参数，默认值为未设置（unset）。 |
| `parameterMap`     | 这是引用外部 parameterMap 的已经被废弃的方法。请使用内联参数映射和 parameterType 属性。 |
| `flushCache`       | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：true（对于 insert、update 和 delete 语句）。 |
| `timeout`          | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖驱动）。 |
| `statementType`    | STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| `useGeneratedKeys` | （仅对 insert 和 update 有用）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系数据库管理系统的自动递增字段），默认值：false。 |
| `keyProperty`      | （仅对 insert 和 update 有用）唯一标记一个属性，MyBatis 会通过 getGeneratedKeys 的返回值或者通过 insert 语句的 selectKey 子元素设置它的键值，默认值：未设置（`unset`）。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| `keyColumn`        | （仅对 insert 和 update 有用）通过生成的键值设置表中的列名，这个设置仅在某些数据库（像 PostgreSQL）是必须的，当主键列不是表中的第一列的时候需要设置。如果希望使用多个生成的列，也可以设置为逗号分隔的属性名称列表。 |
| `databaseId`       | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。 |

### 生成主键

```xml
<selectKey
  keyProperty="id"
  resultType="int"
  order="BEFORE"
  statementType="PREPARED">
```

| 属性            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| `keyProperty`   | selectKey 语句结果应该被设置的目标属性。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| `keyColumn`     | 匹配属性的返回结果集中的列名称。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| `resultType`    | 结果的类型。MyBatis 通常可以推断出来，但是为了更加精确，写上也不会有什么问题。MyBatis 允许将任何简单类型用作主键的类型，包括字符串。如果希望作用于多个生成的列，则可以使用一个包含期望属性的 Object 或一个 Map。 |
| `order`         | 这可以被设置为 BEFORE 或 AFTER。如果设置为 BEFORE，那么它会首先生成主键，设置 keyProperty 然后执行插入语句。如果设置为 AFTER，那么先执行插入语句，然后是 selectKey 中的语句 - 这和 Oracle 数据库的行为相似，在插入语句内部可能有嵌入索引调用。 |
| `statementType` | 与前面相同，MyBatis 支持 STATEMENT，PREPARED 和 CALLABLE 语句的映射类型，分别代表 PreparedStatement 和 CallableStatement 类型。 |

```xml
<selectKey resultType="java.lang.Long" order="AFTER" keyProperty="id">
  SELECT LAST_INSERT_ID() AS id
</selectKey>
```

## sql

这个元素可以被用来定义可重用的 SQL 代码段，这些 SQL 代码可以被包含在其他语句中。它可以（在加载的时候）被静态地设置参数。 在不同的包含语句中可以设置不同的值到参数占位符上。比如：

```xml
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>
<select id="selectUsers" resultType="map">
  select
    <include refid="userColumns"><property name="alias" value="t1"/></include>,
    <include refid="userColumns"><property name="alias" value="t2"/></include>
  from some_table t1
    cross join some_table t2
</select>
```

## 结果映射

`resultMap` 元素是 MyBatis 中最重要最强大的元素，在一些情形下允许你进行一些 JDBC 不支持的操作。实际上，在为一些比如连接的复杂语句编写映射代码的时候，一份 `resultMap` 能够代替实现同等功能的长达数千行的代码。**==ResultMap 的设计思想是，对于简单的语句根本不需要配置显式的结果映射，而对于复杂一点的语句只需要描述它们的关系就行了。==**

在JavaBean中定义一个有 3 个属性：id，username 和 hashedPassword的类，然后在mapper.xml中，这些属性会对应到 select 语句中的列名，这样的一个 JavaBean 可以被映射到 `ResultSet`，就像映射到 `HashMap` 一样简单。

```xml
<select id="selectUsers" resultType="com.someapp.model.User">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```

像第三中的第三个例子，也可以定义外部resultMap，这也是解决列名不匹配的另外一种方式。

```xml
<resultMap id="userResultMap" type="User">
  <id property="id" column="user_id" />
  <result property="username" column="user_name"/>
  <result property="password" column="hashed_password"/>
</resultMap>

<select id="selectUsers" resultMap="userResultMap">
  select user_id, user_name, hashed_password
  from some_table
  where id = #{id}
</select>
```

### 结果映射（resultMap）

- constructor-用于在实例化类时，注入结果到构造方法中
  - `idArg` - ID 参数；标记出作为 ID 的结果可以帮助提高整体性能
  - `arg` - 将被注入到构造方法的一个普通结果
- `id` – 一个 ID 结果；标记出作为 ID 的结果可以帮助提高整体性能
- `result` – 注入到字段或 JavaBean 属性的普通结果
- association– 一个复杂类型的关联；许多结果将包装成这种类型
  - 嵌套结果映射 – 关联本身可以是一个 `resultMap` 元素，或者从别处引用一个
- collection– 一个复杂类型的集合
  - 嵌套结果映射 – 集合本身可以是一个 `resultMap` 元素，或者从别处引用一个
- discriminator– 使用结果值来决定使用哪个resultMap
  - case– 基于某些值的结果映射
    - 嵌套结果映射 – `case` 本身可以是一个 `resultMap` 元素，因此可以具有相同的结构和元素，或者从别处引用一个。

| 属性          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| `id`          | 当前命名空间中的一个唯一标识，用于标识一个结果映射。         |
| `type`        | 类的完全限定名, 或者一个类型别名（关于内置的类型别名，可以参考上面的表格）。 |
| `autoMapping` | 如果设置这个属性，MyBatis将会为本结果映射开启或者关闭自动映射。 这个属性会覆盖全局的属性 autoMappingBehavior。默认值：未设置（unset）。 |

#### id & result

```xml
<id property="id" column="post_id"/>
<result property="subject" column="post_subject"/>
```

​	这些是结果映射最基本的内容。*id* 和 *result* 元素都将一个列的值映射到一个简单数据类型（String, int, double, Date 等）的属性或字段。

​	**这两者之间的唯一不同是，*id* 元素表示的结果将是对象的标识属性，这会在比较对象实例时用到。 这样可以提高整体的性能，尤其是进行缓存和嵌套结果映射（也就是连接映射）的时候。**

| 属性          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| `property`    | 映射到列结果的字段或属性。如果用来匹配的 JavaBean 存在给定名字的属性，那么它将会被使用。否则 MyBatis 将会寻找给定名称的字段。 无论是哪一种情形，你都可以使用通常的点式分隔形式进行复杂属性导航。 比如，你可以这样映射一些简单的东西：“username”，或者映射到一些复杂的东西上：“address.street.number”。 |
| `column`      | 数据库中的列名，或者是列的别名。一般情况下，这和传递给 `resultSet.getString(columnName)` 方法的参数一样。 |
| `javaType`    | 一个 Java 类的完全限定名，或一个类型别名（关于内置的类型别名，可以参考上面的表格）。 如果你映射到一个 JavaBean，MyBatis 通常可以推断类型。然而，如果你映射到的是 HashMap，那么你应该明确地指定 javaType 来保证行为与期望的相一致。 |
| `jdbcType`    | JDBC 类型，所支持的 JDBC 类型参见这个表格之后的“支持的 JDBC 类型”。 只需要在可能执行插入、更新和删除的且允许空值的列上指定 JDBC 类型。这是 JDBC 的要求而非 MyBatis 的要求。如果你直接面向 JDBC 编程，你需要对可能存在空值的列指定这个类型。 |
| `typeHandler` | 我们在前面讨论过默认的类型处理器。使用这个属性，你可以覆盖默认的类型处理器。 这个属性值是一个类型处理器实现类的完全限定名，或者是类型别名。 |

## 缓存

默认情况下，只启用了本地的会话缓存，它仅仅对一个会话中的数据进行缓存。 要启用全局的二级缓存，只需要在你的 SQL 映射文件中添加一行：\<cache/>

基本上就是这样。**这个简单语句的效果如下:**

- 映射语句文件中的所有 select 语句的结果将会被缓存。
- 映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。
- 缓存会使用最近最少使用算法（LRU, Least Recently Used）算法来清除不需要的缓存。
- 缓存不会定时进行刷新（也就是说，没有刷新间隔）。
- 缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用。
- 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。

==**提示** 缓存只作用于 cache 标签所在的映射文件中的语句。如果你混合使用 Java API 和 XML 映射文件，在共用接口中的语句将不会被默认缓存。你需要使用 @CacheNamespaceRef 注解指定缓存作用域。==

这些属性可以通过 cache 元素的属性来修改。比如：

```xml
<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
```

这个更高级的配置创建了一个 FIFO 缓存，每隔 60 秒刷新，最多可以存储结果对象或列表的 512 个引用，而且返回的对象被认为是只读的，因此对它们进行修改可能会在不同线程中的调用者产生冲突。

可用的清除策略有：

- `LRU` – 最近最少使用：移除最长时间不被使用的对象。
- `FIFO` – 先进先出：按对象进入缓存的顺序来移除它们。
- `SOFT` – 软引用：基于垃圾回收器状态和软引用规则移除对象。
- `WEAK` – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象。

默认的清除策略是 LRU。

flushInterval（刷新间隔）属性可以被设置为任意的正整数，设置的值应该是一个以毫秒为单位的合理时间量。 默认情况是不设置，也就是没有刷新间隔，缓存仅仅会在调用语句时刷新。

size（引用数目）属性可以被设置为任意正整数，要注意欲缓存对象的大小和运行环境中可用的内存资源。默认值是 1024。

readOnly（只读）属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓存对象的相同实例。 因此这些对象不能被修改。这就提供了可观的性能提升。而可读写的缓存会（通过序列化）返回缓存对象的拷贝。 速度上会慢一些，但是更安全，因此默认值是 false。

==**提示** 二级缓存是事务性的。这意味着，当 SqlSession 完成并提交时，或是完成并回滚，但没有执行 flushCache=true 的 insert/delete/update 语句时，缓存会获得更新。==

### 使用自定义缓存

除了上述自定义缓存的方式，你也可以通过实现你自己的缓存，或为其他第三方缓存方案创建适配器，来完全覆盖缓存行为。

```xml
<cache type="com.domain.something.MyCustomCache"/>
```

例子：

type 属性指定的类必须实现 org.mybatis.cache.Cache 接口，且提供一个接受 String 参数作为 id 的构造器。 这个接口是 MyBatis 框架中许多复杂的接口之一，但是行为却非常简单。

```Java
public interface Cache {
  String getId();
  int getSize();
  void putObject(Object key, Object value);
  Object getObject(Object key);
  boolean hasKey(Object key);
  Object removeObject(Object key);
  void clear();
}
```

## 动态SQL

- Mybatis 动态 SQL ，可以让我们在 XML 映射文件内，**以 XML 标签的形式编写动态 SQL ，完成逻辑判断和动态拼接 SQL 的功能。**
- Mybatis 提供了 9 种动态 SQL 标签：`<if />`、`<choose />`、`<when />`、`<otherwise />`、`<trim />`、`<where />`、`<set />`、`<foreach />`、`<bind />` 。
- 其执行原理为，使用 **OGNL** 的表达式，从 SQL 参数对象中计算表达式的值，根据表达式的值动态拼接 SQL ，以此来完成动态 SQL 的功能。

### if

动态 SQL 通常要做的事情是根据条件包含 where 子句的一部分。比如：

```xml
<select id="findActiveBlogWithTitleLike"
     resultType="Blog">
  SELECT * FROM BLOG
  WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
</select>
```

​	这条语句提供了一种可选的查找文本功能。**如果没有传入“title”，那么所有处于“ACTIVE”状态的BLOG都会返回；反之若传入了“title”，那么就会对“title”一列进行模糊查找并返回 BLOG 结果**（“title”参数值是可以包含一些掩码或通配符的）。

通过“title”和“author”两个参数进行可选搜索:

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```

### choose, when, otherwise

​	有时我们不想应用到所有的条件语句，而只想从中择其一项。针对这种情况，MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句。 

​	这次变为提供了“title”就按“title”查找，提供了“author”就按“author”查找的情形，若两者都没有提供，就返回所有符合条件的 BLOG（实际情况可能是由管理员按一定策略选出 BLOG 列表，而不是返回大量无意义的随机结果）。

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

### trim, where, set

​	上面的几个例子，如果条件不满足的话，会拼凑成不成一条sql语句，导致无法查询，如：SELECT * FROM BLOG  WHERE

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG
  <where>
    <if test="state != null">
         state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```

​	==*where* 元素只会在至少有一个子元素的条件返回 SQL 子句的情况下才去插入“WHERE”子句。而且，若语句的开头为“AND”或“OR”，*where* 元素也会将它们去除。==

​	如果 *where* 元素没有按正常套路出牌，我们可以**通过自定义 trim 元素来定制 *where* 元素的功能**。比如，和 *where* 元素等价的自定义 trim 元素为：

```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ...
</trim>
```

​	*prefixOverrides* 属性会忽略通过管道分隔的文本序列（注意此例中的空格也是必要的）。它的作用是移除所有指定在 *prefixOverrides* 属性中的内容，并且插入 *prefix* 属性中指定的内容。

类似的用于动态更新语句的解决方案叫做 *set*。*set* 元素可以用于动态包含需要更新的列，而舍去其它的。比如：

```xml
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
```

​	这里，*set* 元素会动态前置 SET 关键字，同时也会删掉无关的逗号，因为用了条件语句之后很可能就会在生成的 SQL 语句的后面留下这些逗号。（译者注：**因为用的是“if”元素，若最后一个“if”没有匹配上而前面的匹配上，SQL 语句的最后就会有一个逗号遗留**）

若你对 *set* 元素等价的自定义 trim 元素的代码感兴趣，那这就是它的真面目：

```xml
<trim prefix="SET" suffixOverrides=",">
  ...
</trim>
```

​	**注意这里我们删去的是后缀值，同时添加了前缀值。**

### foreach

​	动态 SQL 的另外一个常用的操作需求是对一个集合进行遍历，通常是在构建 IN 条件语句的时候。比如：

```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

​	*foreach* 元素的功能非常强大，它允许你指定一个集合，声明可以在元素体内使用的集合项（item）和索引（index）变量。它也允许你指定开头与结尾的字符串以及在迭代结果之间放置分隔符。这个元素是很智能的，因此它不会偶然地附加多余的分隔符。

> **注意** 
>
> 可以将任何可迭代对象（如 List、Set 等）、Map 对象或者数组对象传递给 *foreach* 作为集合参数。**==当使用可迭代对象或者数组时，index 是当前迭代的次数，item 的值是本次迭代获取的元素。当使用 Map 对象（或者 Map.Entry 对象的集合）时，index 是键，item 是值。==**

### bind

`bind` 元素可以从 OGNL 表达式中创建一个变量并将其绑定到上下文。比如：

```xml
<select id="selectBlogsLike" resultType="Blog">
  <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
  SELECT * FROM BLOG
  WHERE title LIKE #{pattern}
</select>
```

# 五、Mybatis工作原理如下：

![image-20190513182031415](/Users/jack/Desktop/md/images/image-20190513182031415.png)

**Dao 实现类：**

![image-20190515000712587](/Users/jack/Desktop/md/images/image-20190515000712587.png)

# 六、Mapper接口的工作原理

Mapper 接口，对应的关系如下：

- 接口的全限名，就是映射文件中的 `"namespace"` 的值。
- 接口的方法名，就是映射文件中 MappedStatement 的 `"id"` 值。
- 接口方法内的参数，就是传递给 SQL 的参数。

Mapper 接口是没有实现类的，当调用接口方法时，接口全限名 + 方法名拼接字符串作为 key 值，可唯一定位一个对应的 MappedStatement 。举例：`com.mybatis3.mappers.StudentDao.findStudentById` ，可以唯一找到 `"namespace"` 为 `com.mybatis3.mappers.StudentDao` 下面 `"id"` 为 `findStudentById` 的 MappedStatement 。

总结来说，在 Mybatis 中，每一个 `<select />`、`<insert />`、`<update />`、`<delete />` 标签，都会被解析为一个 MappedStatement 对象。

Mapper 接口的实现类，通过 MyBatis 使用 **JDK Proxy** 自动生成其代理对象 Proxy ，而代理对象 Proxy 会拦截接口方法，从而“调用”对应的 MappedStatement 方法，最终执行 SQL ，返回执行结果。整体流程如下图：![流程](/Users/jack/Desktop/md/images/02-7742725.png)

​	其中，SqlSession 在调用 Executor 之前，会获得对应的 MappedStatement 方法。例如：`DefaultSqlSession#select(String statement, Object parameter, RowBounds rowBounds, ResultHandler handler)` 方法，代码如下：

```java
// DefaultSqlSession.java
@Override
public void select(String statement, Object parameter, RowBounds rowBounds, ResultHandler handler) {
    try {
        // 获得 MappedStatement 对象
        MappedStatement ms = configuration.getMappedStatement(statement);
        // 执行查询
        executor.query(ms, wrapCollection(parameter), rowBounds, handler);
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
    } finally {
        ErrorContext.instance().reset();
    }
}
```

==Mapper 接口里的方法，是不能重载的，因为是**全限名 + 方法名**的保存和寻找策略。==

# 七、Mapper接口绑定的实现方式

接口绑定有三种实现方式：

第一种，通过 **XML Mapper** 里面写 SQL 来绑定。在这种情况下，要指定 XML 映射文件里面的 `"namespace"` 必须为接口的全路径名。

> 如：**\<mapper namespace="com.taotao.mapper.TbContentCategoryMapper" >**

第二种，通过**注解**绑定，就是在接口的方法上面加上 `@Select`、`@Update`、`@Insert`、`@Delete` 注解，里面包含 SQL 语句来绑定。

第三种，是第二种的特例，也是通过**注解**绑定，在接口的方法上面加上 `@SelectProvider`、`@UpdateProvider`、`@InsertProvider`、`@DeleteProvider` 注解，通过 Java 代码，生成对应的动态 SQL 。

# 八、获取自动生成的(主)键值

不同的数据库，获取自动生成的(主)键值的方式是不同的。

MySQL 有两种方式，但是**自增主键**，代码如下：

```xml
<!--方式一，使用 useGeneratedKeys + keyProperty 属性-->
<insert id="insert" parameterType="Person" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO person(name, pswd)
    VALUE (#{name}, #{pswd})
</insert>
    
<!--方式二，使用 <selectKey /> 标签,返回主键-->
<insert id="insert" parameterType="Person" useGeneratedKeys="true" keyProperty="id">
    <selectKey keyProperty="id" resultType="long" order="AFTER">
        SELECT LAST_INSERT_ID()
    </selectKey>
    INSERT INTO person(name, pswd)
    VALUE (#{name}, #{pswd})
</insert>
```

> SELECT LAST_INSERT_ID()也可以换成select @@identity
>
> - resultType:指出获取的主键的类型。 
> - keyProperty:指出主键在 Java 类中对应的属性名。此处会将获取的主键值直接封装到被插入的实体类对象中。 
> - order:指出 id 的生成相对于 insert 语句的执行是在前还是在后。MySql 数据库表 中的 id，均是先执行 insert 语句，而后生成 id，所以需要设置为 AFTER;Oracle 数据库表中 的 id，则是在 insert 执行之前先生成，所以需要设置为 BEFORE。当前的 MyBatis 版本，不指 定 order 属性，则会根据所用 DBMS，自动选择其值。 
>
> **无论插入操作是提交还是回滚，DB 均会为 insert 的记录分配 id，即使发生回滚，这个 id 也已经被使用。**后面再插入并提交的记录数据，此 id 已经不能再使用，被分配的 id 是跳 过此 id 后的 id。 
>
> 另外，从前面\<selectKey/>中 order 属性值的设置讲解可知，MySql 在 insert 语句执行后 会自动生成该新插入记录的主键值。主键值的生成只与 insert 语句是否执行有关，而与最终 是否提交无关。 

Oracle 有两种方式，**序列**和**触发器**。基于**序列**，根据 `<selectKey />` 执行的时机，也有两种方式，代码如下：

```xml
<!--这个是创建表的自增序列
CREATE SEQUENCE student_sequence
INCREMENT BY 1
NOMAXVALUE
NOCYCLE
CACHE 10;
-->
<!--方式一，使用 `<selectKey />` 标签 + BEFORE-->
<insert id="add" parameterType="Student">
　　<selectKey keyProperty="student_id" resultType="int" order="BEFORE">
      select student_sequence.nextval FROM dual
    </selectKey>
    
     INSERT INTO student(student_id, student_name, student_age)
     VALUES (#{student_id},#{student_name},#{student_age})
</insert>

<!--方式二，使用 `<selectKey />` 标签 + AFTER-->
<insert id="save" parameterType="com.threeti.to.ZoneTO" >
    <selectKey resultType="java.lang.Long" keyProperty="id" order="AFTER" >
      SELECT SEQ_ZONE.CURRVAL AS id FROM dual
    </selectKey>
    
    INSERT INTO TBL_ZONE (ID, NAME ) 
    VALUES (SEQ_ZONE.NEXTVAL, #{name,jdbcType=VARCHAR})
</insert>
```

# 九、在 Mapper 中如何传递多个参数

第一种，直接使用 Map 集合封装，装载多个参数进行传递。代码如下：

```Java
// 调用方法
Map<String, Object> map = new HashMap();
map.put("start", start);
map.put("end", end);
return studentMapper.selectStudents(map);

// Mapper 接口
List<Student> selectStudents(Map<String, Object> map);

// Mapper XML 代码
<select id="selectStudents" parameterType="Map" resultType="Student">
    SELECT * 
    FROM students 
    LIMIT #{start}, #{end}
</select>
```

第二种，**保持传递多个参数，使用 `@Param` 注解。(推荐使用)**代码如下：

```Java
// 调用方法
return studentMapper.selectStudents(0, 10);

// Mapper 接口
List<Student> selectStudents(@Param("start") Integer start, @Param("end") Integer end);

// Mapper XML 代码
<select id="selectStudents" resultType="Student">
    SELECT * 
    FROM students 
    LIMIT #{start}, #{end}
</select>
```

第三种，保持传递多个参数，不使用 `@Param` 注解。代码如下：

```Java
// 调用方法
return studentMapper.selectStudents(0, 10);

// Mapper 接口
List<Student> selectStudents(Integer start, Integer end);

// Mapper XML 代码
<select id="selectStudents" resultType="Student">
    SELECT * 
    FROM students 
    LIMIT #{param1}, #{param2}
</select>
```

​	其中，按照参数在方法方法中的位置，从 1 开始，逐个为 `#{param1}`、`#{param2}`、`#{param3}` 不断向下排列，即通过#{param1}对应Mapper接口方法中的参数位置。

# 十、映射 Enum 枚举类

Mybatis 可以映射枚举类，对应的实现类为 EnumTypeHandler 或 EnumOrdinalTypeHandler 。

- EnumTypeHandler ，基于 `Enum.name` 属性( String )。**默认**。
- EnumOrdinalTypeHandler ，基于 `Enum.ordinal` 属性( `int` )。可通过 `<setting name="defaultEnumTypeHandler" value="EnumOrdinalTypeHandler" />` 来设置。

```Java
public class Dog {
    public static final int STATUS_GOOD = 1;
    public static final int STATUS_BETTER = 2;
    public static final int STATUS_BEST = 3；
    private int status;
}
```

并且，不单可以映射枚举类，Mybatis 可以映射任何对象到表的一列上。映射方式为自定义一个 TypeHandler 类，实现 TypeHandler 的`#setParameter(...)` 和 `#getResult(...)` 接口方法。

### TypeHandler 有两个作用：

- 一是，完成从 javaType 至 jdbcType 的转换。
- 二是，完成 jdbcType 至 javaType 的转换。

==具体体现为 `#setParameter(...)` 和 `#getResult(..)` 两个方法，分别代表设置 SQL 问号占位符参数和获取列查询结果。==

# 十一、Executor 执行器

​	Mybatis 有四种 Executor 执行器，分别是 SimpleExecutor、ReuseExecutor、BatchExecutor、CachingExecutor 。

- SimpleExecutor ：每执行一次 update 或 select 操作，就创建一个 Statement 对象，用完立刻关闭 Statement 对象。
- ReuseExecutor ：执行 update 或 select 操作，以 SQL 作为key 查找**缓存**的 Statement 对象，存在就使用，不存在就创建；==用完后，不关闭 Statement 对象，而是放置于缓存 `Map<String, Statement>` 内，供下一次使用。简言之，就是重复使用 Statement 对象。==
- BatchExecutor(批处理) ：==执行 update 操作（没有 select 操作，因为 JDBC 批处理不支持 select 操作），将所有 SQL 都添加到批处理中（通过 addBatch 方法），等待统一执行（使用 executeBatch 方法）==。它缓存了多个 Statement 对象，每个 Statement 对象都是调用 addBatch 方法完毕后，等待一次执行 executeBatch 批处理。**实际上，整个过程与 JDBC 批处理是相同**。
- CachingExecutor ：在上述的三个执行器之上，增加**二级缓存**的功能。

通过设置 `<setting name="defaultExecutorType" value="">` 的 `"value"` 属性，可传入 SIMPLE、REUSE、BATCH 三个值，分别使用 SimpleExecutor、ReuseExecutor、BatchExecutor 执行器。

通过设置 `<setting name="cacheEnabled" value=""` 的 `"value"` 属性为 `true` 时，创建 CachingExecutor 执行器。

# 十二、批量插入

mapper xml：

```xml
<insert id="insertUser" parameterType="String"> 
    INSERT INTO users(name) 
    VALUES (#{value}) 
</insert>
```

```Java
// mapper接口
public interface UserMapper {
    void insertUser(@Param("name") String name);
}

//调用接口
private static SqlSessionFactory sqlSessionFactory;
@Test
public void testBatch() {
    // 创建要插入的用户的名字的数组
    List<String> names = new ArrayList<>();
    names.add("占小狼");
    names.add("朱小厮");
    names.add("徐妈");
    names.add("飞哥");
    // 获得执行器类型为 Batch 的 SqlSession 对象，并且 autoCommit = false ，禁止事务自动提交
    try (SqlSession session = sqlSessionFactory.openSession(ExecutorType.BATCH, false)) {
        // 获得 Mapper 对象
        UserMapper mapper = session.getMapper(UserMapper.class);
        // 循环插入
        for (String name : names) {
            mapper.insertUser(name);
        }
        // 提交批量操作
        session.commit();
    }
}
```

另一种方式：

```xml
INSERT INTO [表名]([列名],[列名]) 
VALUES
([列值],[列值])),
([列值],[列值])),
([列值],[列值]));
```

- 对于这种方式，需要保证单条 SQL 不超过语句的最大限制 `max_allowed_packet` 大小，默认为 1 M 。

结论：少量插入请使用反复插入单条数据，方便。数量较多请使用批处理方式。（可以考虑以有需求的插入数据量20条左右为界吧，在我的测试和数据库环境下耗时都是百毫秒级的，方便最重要）。**无论何时都不用xml拼接sql的方式**。

这两种方式的性能对比，可以看看 [《[实验\]mybatis批量插入方式的比较》](https://www.jianshu.com/p/cce617be9f9e) 。

# 十三、源码分析

Mybatis的Dao实现类如下：

![image-20190515104350561](/Users/jack/Desktop/md/images/image-20190515104350561.png)

## A、 输入流的关闭

​	在输入流对象使用完毕后，不用手工进行流的关闭。因为在输入流被使用完毕后，**SqlSessionFactoryBuilder 对象的 build()方法会自动将输入流关闭。**

```java
//SqlSessionFactoryBuilder.java
public SqlSessionFactory build(InputStream inputStream) {
  return build(inputStream, null, null);
}
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    try {
      XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
      return build(parser.parse());
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {	// 关闭输入流
        inputStream.close();
      } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
      }
    }
  }
```

## B、 SqlSession 的创建

> ==SqlSession 接口对象用于执行持久化操作。一个 SqlSession 对应着一次数据库会话，一 次会话以 SqlSession 对象的创建开始，以 SqlSession 对象的关闭结束。==
>
> **SqlSession 接口对象是线程不安全的，所以每次数据库会话结束前，需要马上调用其 close()方法，将其关闭。**再次需要会话，再次创建。而在关闭时会判断当前的 SqlSession 是否被提交:若没有被提交，则会执行回滚后关闭;若已被提交，则直接将 SqlSession 关闭。 所以，SqlSession 无需手工回滚。 
>
> 主要是一些增删改查的方法。

​	SqlSession 对象的创建，需要使用 SqlSessionFactory 接口对象的 openSession()方法。 SqlSessionFactory 接口对象是一个重量级对象(系统开销大的对象)，是线程安全的，所以一个应用只需要一个该对象即可。**创建 SqlSession 需要使用 SqlSessionFactory 接口的的 openSession()方法。** 

- openSession(true):创建一个有自动提交功能的 SqlSession

- openSession(false):创建一个非自动提交功能的 SqlSession，需手动提交 
- openSession():同 openSession(false) ，即无参的openSession方法默认false是autoCommit的值

**SqlSessionFactory 接口的实现类为 DefaultSqlSessionFactory。** 

![image-20190515104608315](/Users/jack/Desktop/md/images/image-20190515104608315.png)

```java
// SqlSessionFactory.java
public interface SqlSessionFactory {
  SqlSession openSession();
    // 多个openSession方法
  Configuration getConfiguration();
}
// DefaultSqlSessionFactory.java
public SqlSession openSession() {
    // false是autoCommit的值，表示关闭事务的自动提交功能
    return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
 }
//autoCommit表示是否自动提交事务
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    try {
        //读取Mybatis的主配置文件
      final Environment environment = configuration.getEnvironment();
        // 获取事务管理器transcationManager，比如配置文件中的JDBC
      final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
      tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
       // 创建执行器，传入的是事务和执行器类型(SIMPLE, REUSE, BATCH)
      final Executor executor = configuration.newExecutor(tx, execType);
      return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
      closeTransaction(tx); // may have fetched a connection so lets call close()
      throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
//DefaultSqlSession.java
// 所谓创建SqlSession就是对一个dirty这个变量进行初始化，即是否为脏数据的意思
public DefaultSqlSession(Configuration configuration, Executor executor, boolean autoCommit) {
    // 对成员变量进行初始化
    this.configuration = configuration;
    this.executor = executor;
    this.dirty = false;		// 	这个变量为false表示现在DB中的数据还未被修改
    this.autoCommit = autoCommit;
  }
```

​	从以上源码可以看到，**无参的 openSession()方法，将事务的自动提交直接赋值为 false。**而所谓创建 SqlSession，就是加载了主配置文件，创建了一个执行器对象(将来用于执行映射文件中的 SQL 语句)，初始化了一个 DB 数据被修改的标志变量 dirty，关闭了事务的自动提交功能。

## C、 增删改的执行

 	==对于 SqlSession 的 insert()、delete()、update()方法，其底层均是调用执行了 update()方法==，只要对数据进行了增删改，那么dirty就会变为true，表示数据被修改了。

```java
// DefaultSqlSession.java
public int insert(String statement, Object parameter) {
    return update(statement, parameter);
  }
public int delete(String statement, Object parameter) {
    return update(statement, parameter);
  }
public int update(String statement, Object parameter) {
    try {
      dirty = true;		//这里要开始修改数据了，所以要将dirty改为true，表示此时是脏数据
      // statement是获取映射文件中制定的sql语句，即mapper映射文件中的sql id
      MappedStatement ms = configuration.getMappedStatement(statement);
      return executor.update(ms, wrapCollection(parameter));
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error updating database.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
```

​	**从以上源码可知，无论执行增、删还是改，均是对数据进行修改，均将 dirty 变量设置为了 true，且在获取到映射文件中指定 id 的 SQL 语句后，由执行器 executor 执行。**

## D、 SqlSession 的提交 commit()

```Java
// DefaultSqlSession.java
public void commit() {
  commit(false);
}
public void commit(boolean force) {
    try {
        // 执行提交
      executor.commit(isCommitOrRollbackRequired(force));
      dirty = false;	// 提交之后把dirty设置为false，表示数据未修改
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error committing transaction.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
// 提交还是回滚
/**当autoCommit为true时，返回false；
   当autoCommit为false，dirty为true时，返回true；
   当autoCommit为false，dirty为false时，如果force为true则返回true，为false则返回false
   在这里根据上面方法传过来的参数值，autoCommit为false，所以!false==true，dirty为true，force为false，所以isCommitOrRollbackRequired返回true。
*/
private boolean isCommitOrRollbackRequired(boolean force) {
    return (!autoCommit && dirty) || force;
  }
// CachingExecutor.java
// required根据上面的值是为true
public void commit(boolean required) throws SQLException {
    delegate.commit(required);
    tcm.commit();
  }
//BaseExecutor.java
public void commit(boolean required) throws SQLException {
    if (closed) throw new ExecutorException("Cannot commit, transaction is already closed");
    clearLocalCache();
    flushStatements();
    if (required) {	// 根据上面返回的结果，required为true，提交事务
      transaction.commit();
    }
  }
```

​	由以上代码可知，执行 SqlSession 的无参 commit()方法，最终会将事务进行提交。

## E、 SqlSession 的关闭

```java
//DefaultSqlSession.java
public void close() {
  try {
      // 如果执行了commit方法，那么这里返回的是false，即close方法中传入的是false
    executor.close(isCommitOrRollbackRequired(false));
    dirty = false;
  } finally {
    ErrorContext.instance().reset();
  }
}
// 这里的force为false，autoCommit在最开始的openSession方法中传入的是为false，dirty在commit之后，而在commit方法中，将dirty设置为false了，所以这里dirty是false，所以这里整体返回的是false
private boolean isCommitOrRollbackRequired(boolean force) {
    return (!autoCommit && dirty) || force;
  }
//BaseExecutor.java
public void close(boolean forceRollback) {
    try {
      try {
          // 根据上面传入的值，forceRollback为false
        rollback(forceRollback);
      } finally {	// 最后要确认事务关闭，如果前面执行了增删改查方法，说明提交了事务，所以事务不为空
        if (transaction != null) transaction.close();
      }
    } catch (SQLException e) {
      // Ignore.  There's nothing that can be done at this point.
      log.warn("Unexpected exception on closing transaction.  Cause: " + e);
    } finally {	//释放各种资源，并将关闭标志closed重置为true
      transaction = null;
      deferredLoads = null;
      localCache = null;
      localOutputParameterCache = null;
      closed = true;
    }
  }
// 根据上面传进来的值，required为false
public void rollback(boolean required) throws SQLException {
    if (!closed) {	// 此时还未关闭，所以closed为false，这里!closed为true
      try {
        clearLocalCache();
        flushStatements(true);
      } finally {
        if (required) {		// required为false，不会回滚事务
          transaction.rollback();
        }
      }
    }
  }
```

​	从以上代码分析可知，在 SqlSession 进行关闭时，如果执行了commit，那么不会回滚事务；如果没有执行commit方法，那么就会回滚事务，那么数据不会插入到数据库。所以，对于MyBatis 程序，无需通过显式地对 SqlSession 进行回滚，达到事务回滚的目的。

# 十四、一级和二级缓存实现原理

























































































































































































































































































































































































































































































































































































































































































































参照：芋道源码

mybatis文档：<http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html>