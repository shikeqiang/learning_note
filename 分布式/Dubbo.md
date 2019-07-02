# 一、Dubbo配置方式

- XML 配置

- 注解配置

- 属性配置

- Java API 配置

- 外部化配置

  > 参照： [《Dubbo 新编程模型之外部化配置》](https://segmentfault.com/a/1190000012661402)

## provider.xml 示例

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-provider"/>
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
    <dubbo:protocol name="dubbo" port="20890"/>
    <bean id="demoService" class="org.apache.dubbo.samples.basic.impl.DemoServiceImpl"/>
    <dubbo:service interface="org.apache.dubbo.samples.basic.api.DemoService" ref="demoService"/>
</beans>
```

## consumer.xml示例

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-consumer"/>
    <dubbo:registry group="aaa" address="zookeeper://127.0.0.1:2181"/>
    <dubbo:reference id="demoService" check="false" interface="org.apache.dubbo.samples.basic.api.DemoService"/>
</beans>
```

> 此外，还可以使用\<dubbo:annotation/> 标签，基于注解的dubbo配置。

所有标签都支持自定义参数，用于不同扩展点实现的特殊配置，如：

```xml
<dubbo:protocol name="jms">
    <dubbo:parameter key="queue" value="your_queue" />
</dubbo:protocol>
```

或：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">  
    <dubbo:protocol name="jms" p:queue="your_queue" />  
</beans>
```

## 配置之间的关系

![dubbo-config](/Users/jack/Desktop/md/images/dubbo-config.jpg)

 Dubbo 内置配置类：

| 配置类              | 标签                   | 用途         | 解释                                                         |
| ------------------- | ---------------------- | ------------ | ------------------------------------------------------------ |
| `ProtocolConfig`    | `<dubbo:protocol/>`    | 协议配置     | 用于配置提供服务的协议信息，协议由提供方指定，消费方被动接受 |
| `ApplicationConfig` | `<dubbo:application/>` | 应用配置     | 用于配置当前应用信息，不管该应用是提供者还是消费者           |
| `ModuleConfig`      | `<dubbo:module/>`      | 模块配置     | 用于配置当前模块信息，可选                                   |
| `RegistryConfig`    | `<dubbo:registry/>`    | 注册中心配置 | 用于配置连接注册中心相关信息                                 |
| `MonitorConfig`     | `<dubbo:monitor/>`     | 监控中心配置 | 用于配置连接监控中心相关信息，可选                           |
| `ProviderConfig`    | `<dubbo:provider/>`    | 提供方配置   | 当 ProtocolConfig 和 ServiceConfig 某属性没有配置时，采用此缺省值，可选 |
| `ConsumerConfig`    | `<dubbo:consumer/>`    | 消费方配置   | 当 ReferenceConfig 某属性没有配置时，采用此缺省值，可选      |
| `MethodConfig`      | `<dubbo:method/>`      | 方法配置     | 用于 ServiceConfig 和 ReferenceConfig 指定方法级的配置信息   |
| `ArgumentConfig`    | `<dubbo:argument/>`    | 参数配置     | 用于指定方法参数配置                                         |
| ReferenceConfig     | \<dubbo:reference/>    | 引用配置     | 用于创建一个远程服务代理，一个引用可以指向多个注册中心。     |

> 注意：
>
> 引用配置中，引用缺省是延迟初始化的，只有引用被注入到其它 Bean，或被 `getBean()` 获取，才会初始化。如果需要饥饿加载，即没有人引用也立即生成动态代理，可以配置：`<dubbo:reference ... init="true" />`

## 不同粒度配置的覆盖关系

以 timeout 为例，下图显示了配置的查找顺序，其它 retries, loadbalance, actives 等类似：

- 方法级优先，接口级次之，全局配置再次之。
- 如果级别一样，则消费方优先，提供方次之。

其中，服务提供方配置，通过 URL 经由注册中心传递给消费方。

![dubbo-config-override](/Users/jack/Desktop/md/images/dubbo-config-override.jpg)

（==建议由服务提供方设置超时，因为一个方法需要执行多长时间，服务提供方更清楚，如果一个消费方同时引用多个服务，就不需要关心每个服务的超时设置==）。

理论上 ReferenceConfig 中除了`interface`这一项，其他所有配置项都可以缺省不配置，框架会自动使用ConsumerConfig，ServiceConfig, ProviderConfig等提供的缺省配置。

参照：[dubbo文档](http://dubbo.apache.org/zh-cn/docs/user/configuration/xml.html)

# 二、Dubbo 框架的分层设计

![æ´ä½è®¾è®¡](/Users/jack/Desktop/md/images/01-1967965.png)

## **图例说明**

- 最顶上九个**图标**，代表本图中的对象与流程，包括消费者，生产者，继承等。

- 图中左边 **淡蓝背景**( Consumer ) 的为服务消费方使用的接口，右边 **淡绿色背景**( Provider ) 的为服务提供方使用的接口，位于中轴线上的为双方都用到的接口。

- 图中从下至上分为十层，各层均为**单向**依赖，右边的 **黑色箭头**( Depend ) 代表层之间的依赖关系，每一层都可以剥离上层被复用。其中，Service 和 Config 层为 API，其它各层均为 [SPI](http://blog.csdn.net/top_code/article/details/51934459) 。

  > SPI是JDK内置的一种服务提供发现机制，一**个服务(Service)通常指的是已知的接口或者抽象类，服务提供方就是对这个接口或者抽象类的实现，然后按照SPI 标准存放到资源路径META-INF/services目录下，文件的命名为该服务接口的全限定名。**
  >
  > 注意，Dubbo 并未使用 JDK SPI 机制，而是自己实现了一套 Dubbo SPI 机制。

- 图中 **绿色小块**( Interface ) 的为扩展接口，**蓝色小块**( Class ) 为实现类，图中只显示用于关联各层的实现类。

- 图中 **蓝色虚线**( Init ) 为初始化过程，即启动时组装链。**红色实线**( Call )为方法调用过程，即运行时调时链。**紫色三角箭头**( Inherit )为继承，可以把子类看作父类的同一个节点，线上的文字为调用的方法。

## **各层说明**

​	总共有10层，主要分为Business、RPC、Remoting **三大层**。如下：

###      Business

- **Service 业务层**：业务代码的接口与实现。我们实际使用 Dubbo 的业务层级。

  > 接口层，给服务提供者和消费者来实现的。

  ### RPC

- **config 配置层**：对外配置接口，以 ServiceConfig, ReferenceConfig 为中心，可以直接初始化配置类，也可以通过 Spring 解析配置生成配置类。

  > 配置层，主要是对 Dubbo 进行各种配置的。

- **proxy 服务代理层**：服务接口透明代理，生成服务的客户端 Stub 和服务器端 Skeleton, 扩展接口为 ProxyFactory 。

  > 服务代理层，无论是 consumer 还是 provider，Dubbo 都会给你生成代理，代理之间进行网络通信。
  >
  > 类比成 Feign 对于 consumer ，Spring MVC 对于 provider 。

- **registry** 注册中心层：封装服务地址的注册与发现，以服务 URL 为中心，扩展接口为 RegistryFactory, Registry, RegistryService 。

  > 服务注册层，负责服务的注册与发现,可以类比成 Eureka Client 。

- **cluster 路由层**：封装多个提供者的路由及负载均衡，并桥接注册中心，以 Invoker 为中心，扩展接口为 Cluster, Directory, Router, LoadBalance 。

  > 集群层，封装多个服务提供者的路由以及负载均衡，将多个实例组合成一个服务,可以类比城 Ribbon 。

- **monitor 监控层**：RPC 调用次数和调用时间监控，以 Statistics 为中心，扩展接口为 MonitorFactory, Monitor, MonitorService 。

  > 监控层，对 rpc 接口的调用次数和调用时间进行监控。
  >
  > 如果胖友了解 SkyWalking 链路追踪，你会发现，SkyWalking 基于 MonitorFilter 实现增强，从而透明化埋点监控。

  ### Remoting

- **protocol 远程调用层**：==封将 RPC 调用，以 Invocation, Result 为中心，扩展接口为 Protocol, Invoker, Exporter 。==

  > 远程调用层，封装 rpc 调用。

- **exchange 信息交换层**：封装请求响应模式，同步转异步，以 Request, Response 为中心，扩展接口为 Exchanger, ExchangeChannel, ExchangeClient, ExchangeServer 。

  > 信息交换层，封装请求响应模式，同步转异步。

- **transport 网络传输层**：抽象 mina 和 netty 为统一接口，以 Message 为中心，扩展接口为 Channel, Transporter, Client, Server, Codec 。

  > 网络传输层，抽象 mina 和 netty 为统一接口。

- **serialize 数据序列化层**：可复用的一些工具，扩展接口为 Serialization, ObjectInput, ObjectOutput, ThreadPool 。

  > 数据序列化层。

# 三、Dubbo 调用流程

​	==默认情况下，Dubbo调用是**同步**的方式。==

![简化调用图](/Users/jack/Desktop/md/images/01-20190701174924500.png)

- Provider
  - 第 0 步，start 启动服务。
  - 第 1 步，register 注册服务到注册中心。
- Consumer
  - 第 2 步，subscribe 向注册中心订阅服务。
    - ==注意，只订阅使用到的服务，而且首次会拉取订阅的服务列表，缓存在本地。==
  - 【**异步**】第 3 步，**notify 当服务发生变化时，获取最新的服务列表，更新本地缓存。如果有变更，注册中心将基于长连接推送变更数据给消费者。**
- invoke 调用
  - Consumer 直接发起对 Provider 的调用，无需经过注册中心。而对多个 Provider 的负载均衡，Consumer 通过 **cluster** 组件实现。
- count 监控
  - 【异步】Consumer 和 Provider 都异步通知监控中心。

更立体的展示 Dubbo 的调用流程：

![详细调用图](/Users/jack/Desktop/md/images/01-20190701174924731.png)

- 注意，图中的【代理】指的是 **proxy 代理服务层**，和 Consumer 或 Provider 在同一进程中。
- 注意，图中的【负载均衡】指的是 **cluster 路由层**，和 Consumer 或 Provider 在同一进程中。

# 四、Dubbo异常处理

​	Dubbo 异常处理机制涉及的内容比较多，核心在于 Provider 的 异常过滤器 **ExceptionFilter** 对调用结果的各种情况的处理。

## 1.自定义异常的方式

Dubbo的异常处理类是com.alibaba.dubbo.rpc.filter.ExceptionFilter 类,对异常的处理分为下面几类:

1)如果provider实现了GenericService接口,直接抛出

2)如果是checked异常，直接抛出

3)在方法签名上有声明，直接抛出

4)异常类和接口类在同一jar包里，直接抛出

5)是JDK自带的异常，直接抛出

6)是Dubbo本身的异常，直接抛出

7)否则，包装成RuntimeException抛给客户端

最终选择的方案：

![img](/Users/jack/Desktop/md/images/Center.png)

​	在异常处理这里,加上自定义异常处理的代码，或者直接将112行的RuntimeException替换成自己的自定义异常!

​	修改源码后,可以替换maven仓库的jar,或者是在自己项目里面建一个ExceptionFilter,包名和dubbo的相同,用来覆盖掉dubbo的(如果替换112行为自定义异常,则要引入自定义的包等).

这样就从根本上解决了异常处理的问题.后续有其他问题,也可以直接修改.

参照：[《Dubbo(四) 异常处理》](https://blog.csdn.net/qq315737546/article/details/53915067)

## 2.ExceptionFilter源码解析

​	ExceptionFilter用于服务**提供者**中，用途如下：

> 1. 不期望的异常(unchecked exception)会在 ERROR级别被打印到日志( Provider端 )。**不期望的日志即接口上没有声明的Unchecked异常。**
> 2. 异常不在 API 包中，则 Wrap 一层 RuntimeException 。RPC 对于第一层异常会直接序列化传输( Cause 异常会 String 化) ，避免异常在 Client 出不能**反序列化**问题。

源码如下：

```java
@Activate(group = Constants.PROVIDER)
public class ExceptionFilter implements Filter {
    private final Logger logger;
    public ExceptionFilter() {
        this(LoggerFactory.getLogger(ExceptionFilter.class));
    }
    public ExceptionFilter(Logger logger) {
        this.logger = logger;
    }
    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        try {
            Result result = invoker.invoke(invocation);		// 服务调用
            if (result.hasException() && GenericService.class != invoker.getInterface()) {	// 有异常，并且非泛化调用
                try {
                    Throwable exception = result.getException();
                    // 如果是checked异常，直接抛出(返回)。因为checked 异常，肯定定义在接口上。
                    if (!(exception instanceof RuntimeException) && (exception instanceof Exception)) {
                        return result;
                    }
                    // 在方法签名上有声明，直接抛出
                    try {
                        Method method = invoker.getInterface().getMethod(invocation.getMethodName(), invocation.getParameterTypes());
                        Class<?>[] exceptionClassses = method.getExceptionTypes();
                        for (Class<?> exceptionClass : exceptionClassses) {
                            if (exception.getClass().equals(exceptionClass)) {
                                return result;
                            }
                        }
                    } catch (NoSuchMethodException e) {
                        return result;
                    }
                    // 未在方法签名上定义的异常，在服务器端打印 ERROR 日志
                    logger.error("Got unchecked and undeclared exception which called by " + RpcContext.getContext().getRemoteHost()
                            + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName()
                            + ", exception: " + exception.getClass().getName() + ": " + exception.getMessage(), exception);

      // 异常类和接口类在同一 jar 包里，直接返回结果，因为服务消费者可以反序列化该异常。代码在引用里：
                    String serviceFile = ReflectUtils.getCodeBase(invoker.getInterface());
                    String exceptionFile = ReflectUtils.getCodeBase(exception.getClass());
                    if (serviceFile == null || exceptionFile == null || serviceFile.equals(exceptionFile)) {
                        return result;
                    }
                    // 如果是JDK自带的异常，直接抛出
                    String className = exception.getClass().getName();
                    if (className.startsWith("java.") || className.startsWith("javax.")) {
                        return result;
                    }
                    //是Dubbo本身的异常，直接抛出
                    if (exception instanceof RpcException) {
                        return result;
                    }
 // 否则，包装成 RuntimeException 异常返回给服务消费者，同时把异常堆栈给包进去。代码如引用里
                    return new RpcResult(new RuntimeException(StringUtils.toString(exception)));
                } catch (Throwable e) {
                    logger.warn("Fail to ExceptionFilter when called by " + RpcContext.getContext().getRemoteHost()
                            + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName()
                            + ", exception: " + e.getClass().getName() + ": " + e.getMessage(), e);
                    return result;
                }
            }
            return result;
        } catch (RuntimeException e) {
            logger.error("Got unchecked and undeclared exception which called by " + RpcContext.getContext().getRemoteHost()
                    + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName()
                    + ", exception: " + e.getClass().getName() + ": " + e.getMessage(), e);
            throw e;
        }
    }
}
```

> 服务消费者可以反序列化该异常：
>
> ```java
> public static String getCodeBase(Class<?> cls) {
>     if (cls == null) {
>         return null;
>     }
>     ProtectionDomain domain = cls.getProtectionDomain();
>     if (domain == null) {
>         return null;
>     }
>     CodeSource source = domain.getCodeSource();
>     if (source == null) {
>         return null;
>     }
>     URL location = source.getLocation();
>     if (location == null) {
>         return null;
>     }
>     return location.getFile();
> }
> ```
>
> 包装成RuntimeException
>
> ```java
> public static String toString(Throwable e) {
>     UnsafeStringWriter w = new UnsafeStringWriter();
>     PrintWriter p = new PrintWriter(w);
>     p.print(e.getClass().getName());
>     if (e.getMessage() != null) {
>         p.print(": " + e.getMessage());
>     }
>     p.println();
>     try {
>         e.printStackTrace(p);
>         return w.toString();
>     } finally {
>         p.close();
>     }
> }
> ```

参照：

- [《浅谈 Dubbo 的 ExceptionFilter 异常处理》](https://blog.csdn.net/mj158518/article/details/51228649)
- [《精尽 Dubbo 源码分析 —— 过滤器（七）之 ExceptionFilter》](http://svip.iocoder.cn/Dubbo/filter-exception-filter/)

# 五、参数验证

​	参数验证功能是基于 [JSR303](https://jcp.org/en/jsr/detail?id=303) 实现的，用户只需标识 JSR303 标准的验证 annotation，并通过声明 filter 来实现验证。

- 参数校验功能，通过参数校验过滤器 **ValidationFilter** 来实现。
- ValidationFilter 在 Dubbo Provider 和 Consumer 都可生效。
  - 如果我们将**校验注解**写在 Service 接口的方法上，那么 Consumer 在本地就会校验。如果校验不通过，直接抛出校验失败的异常，不会发起 Dubbo 调用。
  - 如果我们将**校验注解**写在 Service 实现的方法上，那么 Consumer 在本地不会校验，而是由 Provider 校验。

## Maven 依赖

```xml
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.0.0.GA</version>
</dependency>
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>4.2.0.Final</version>
</dependency>
```

## 示例：

### 参数标注示例

```java
import java.io.Serializable;
import java.util.Date;
import javax.validation.constraints.Future;
import javax.validation.constraints.Max;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Past;
import javax.validation.constraints.Pattern;
import javax.validation.constraints.Size;
 
public class ValidationParameter implements Serializable {
    private static final long serialVersionUID = 7158911668568000392L;
    @NotNull // 不允许为空
    @Size(min = 1, max = 20) // 长度或大小范围
    private String name;
    @NotNull(groups = ValidationService.Save.class) // 保存时不允许为空，更新时允许为空 ，表示不更新该字段
    @Pattern(regexp = "^\\s*\\w+(?:\\.{0,1}[\\w-]+)*@[a-zA-Z0-9]+(?:[-.][a-zA-Z0-9]+)*\\.[a-zA-Z]+\\s*$")
    private String email;
    @Min(18) // 最小值
    @Max(100) // 最大值
    private int age;
    @Past // 必须为一个过去的时间
    private Date loginDate;
    @Future // 必须为一个未来的时间
    private Date expiryDate;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getEmail() {
        return email;
    }
    public void setEmail(String email) {
        this.email = email;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public Date getLoginDate() {
        return loginDate;
    }
    public void setLoginDate(Date loginDate) {
        this.loginDate = loginDate;
    }
    public Date getExpiryDate() {
        return expiryDate;
    }
    public void setExpiryDate(Date expiryDate) {
        this.expiryDate = expiryDate;
    }
}
```

### 分组验证示例

```java
public interface ValidationService { 
    // 缺省可按服务接口区分验证场景，如：@NotNull(groups = ValidationService.class)   
    @interface Save{} // 与方法同名接口，首字母大写，用于区分验证场景，如：@NotNull(groups = ValidationService.Save.class)，可选
    void save(ValidationParameter parameter);
    void update(ValidationParameter parameter);
}
```

### 关联验证示例

```java
import javax.validation.GroupSequence;
public interface ValidationService {   
    @GroupSequence(Update.class) // 同时验证Update组规则
    @interface Save{}
    void save(ValidationParameter parameter);
    @interface Update{} 
    void update(ValidationParameter parameter);
}
```

### 参数验证示例

```java
import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;
 
public interface ValidationService {
    void save(@NotNull ValidationParameter parameter); // 验证参数不为空
    void delete(@Min(1) int id); // 直接对基本类型参数验证
}
```

## 配置

### 在客户端验证参数

```xml
<dubbo:reference id="validationService" interface="org.apache.dubbo.examples.validation.api.ValidationService" validation="true" />
```

### 在服务器端验证参数

```xml
<dubbo:service interface="org.apache.dubbo.examples.validation.api.ValidationService" ref="validationService" validation="true" />
```

## 验证异常信息

```java
import javax.validation.ConstraintViolationException;
import javax.validation.ConstraintViolationException;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.apache.dubbo.examples.validation.api.ValidationParameter;
import org.apache.dubbo.examples.validation.api.ValidationService;
import org.apache.dubbo.rpc.RpcException;
 
public class ValidationConsumer {   
    public static void main(String[] args) throws Exception {
        String config = ValidationConsumer.class.getPackage().getName().replace('.', '/') + "/validation-consumer.xml";
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(config);
        context.start();
        ValidationService validationService = (ValidationService)context.getBean("validationService");
        // Error
        try {
            parameter = new ValidationParameter();
            validationService.save(parameter);
            System.out.println("Validation ERROR");
        } catch (RpcException e) { // 抛出的是RpcException
            ConstraintViolationException ve = (ConstraintViolationException) e.getCause(); // 里面嵌了一个ConstraintViolationException
            Set<ConstraintViolation<?>> violations = ve.getConstraintViolations(); // 可以拿到一个验证错误详细信息的集合
            System.out.println(violations);
        }
    } 
}
```

参照：[Dubbo文档](http://dubbo.apache.org/zh-cn/docs/user/demos/parameter-validation.html)

# 六、缓存结果

​	**Dubbo 通过 CacheFilter 过滤器，提供结果缓存的功能**，既可以适用于 Consumer 也可适用于 Provider 。

​	**通过结果缓存，用于加速热门数据的访问速度，Dubbo 提供声明式缓存，以减少用户加缓存的工作量。**

Dubbo 目前提供三种实现：

- `lru` ：基于最近最少使用原则删除多余缓存，保持最热的数据被缓存。
- `threadlocal` ：当前线程缓存，比如一个页面渲染，用到很多 portal，每个 portal 都要去查用户信息，通过线程缓存，可以减少这种多余访问。
- `jcache` ：与 [JSR107](https://jcp.org/en/jsr/detail?id=107) 集成，可以桥接各种缓存实现。

## 配置

```xml
<dubbo:reference interface="com.foo.BarService" cache="lru" />
```

或：

```xml
<dubbo:reference interface="com.foo.BarService">
    <dubbo:method name="findBar" cache="lru" />
</dubbo:reference>
```

参照： [《Dubbo 用户指南 —— 结果缓存》](http://dubbo.apache.org/zh-cn/docs/user/demos/result-cache.html) 

## 源码分析

![ç±"å¾](/Users/jack/Desktop/md/images/01-20190702175511709.png)

### 1.CacheFilter过滤器

```java
@Activate(group = {Constants.CONSUMER, Constants.PROVIDER}, value = Constants.CACHE_KEY)
public class CacheFilter implements Filter {
	/**
 	* CacheFactory$Adaptive 对象。
	* 通过 Dubbo SPI 机制，调用 {@link #setCacheFactory(CacheFactory)} 方法，进行注入
	*/
    private CacheFactory cacheFactory;
    public void setCacheFactory(CacheFactory cacheFactory) {
        this.cacheFactory = cacheFactory;
    }
    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        // 方法开启 Cache 功能，因为一个服务里，可能只有部分方法开启了 Cache 功能。
        if (cacheFactory != null && ConfigUtils.isNotEmpty(invoker.getUrl().getMethodParameter(invocation.getMethodName(), Constants.CACHE_KEY))) {
             // 基于 URL + Method 获得 Cache 对象。
            Cache cache = cacheFactory.getCache(invoker.getUrl(), invocation);
            if (cache != null) {
                 // 获得 Cache Key
                String key = StringUtils.toArgumentString(invocation.getArguments());
                 // 从缓存中获得结果。若存在，创建 RpcResult 对象。
                Object value = cache.get(key);
                if (value != null) {
                    return new RpcResult(value);
                }
                Result result = invoker.invoke(invocation);		// 服务调用
                if (!result.hasException()) {	 // 若非异常结果，缓存结果
                    cache.put(key, result.getValue());	
                }
                return result;
            }
        }
        return invoker.invoke(invocation);		// 服务调用
    }
}
```

获取Cache key：

```java
/**
 * 将参数数组，拼接成字符串。
 * 1. 使用逗号分隔
 * 2. 使用 JSON 格式化对象
 * @param args 参数数组
 * @return 字符串
 */
public static String toArgumentString(Object[] args) {
    StringBuilder buf = new StringBuilder();
    for (Object arg : args) {
        if (buf.length() > 0) {
            buf.append(Constants.COMMA_SEPARATOR); // 分隔
        }
        // 拼接参数
        if (arg == null || ReflectUtils.isPrimitives(arg.getClass())) {
            buf.append(arg);
        } else {
            try {
                buf.append(JSON.toJSONString(arg)); // 使用 JSON 格式化对象
            } catch (Exception e) {
                logger.warn(e.getMessage(), e);
                buf.append(arg);
            }
        }
    }
    return buf.toString();
}
```

### 2.API 定义

#### 2.1 Cache

`com.alibaba.dubbo.cache.Cache` ，缓存**容器**接口，Cache 是个缓存容器，内部可以管理缓存的键值。方法如下：

```java
public interface Cache {
    /**
     * 添加键值
     * @param key 键
     * @param value 值
     */
    void put(Object key, Object value);
    /**
     * 获得值
     * @param key 键
     * @return 值
     */
    Object get(Object key);
}
```

#### 2.2 CacheFactory

`com.alibaba.dubbo.cache.CacheFactory` ，Cache 工厂**接口**。方法如下：

```java
@SPI("lru")
public interface CacheFactory {
    /**
     * 获得缓存对象
     * @param url URL 对象
     * @return 缓存对象
     */
    @Adaptive("cache")
    Cache getCache(URL url);
}
```

- `@SPI("lru")` 注解，Dubbo SPI **拓展点**，默认为 `"lru"` 。
- `@Adaptive("cache")` 注解，基于 Dubbo SPI Adaptive 机制，加载对应的 Cache 实现，使用 `URL.cache` 属性。

##### 2.2.1 AbstractCacheFactory

`com.alibaba.dubbo.cache.support.AbstractCacheFactory` ，Cache 工厂**抽象类**。代码如下：

```java
public abstract class AbstractCacheFactory implements CacheFactory {
    /**
     * Cache 集合
     * key：URL
     */
    private final ConcurrentMap<String, Cache> caches = new ConcurrentHashMap<String, Cache>();
    @Override
    public Cache getCache(URL url) {
        // 获得 Cache 对象
        String key = url.toFullString();
        Cache cache = caches.get(key);
        // 不存在，创建 Cache 对象，并缓存
        if (cache == null) {
            caches.put(key, createCache(url));
            cache = caches.get(key);
        }
        return cache;
    }
    /**
     * 创建 Cache 对象
     * @param url URL
     * @return Cache 对象
     */
    protected abstract Cache createCache(URL url);

}
```

### 3.LRU 实现

`lru` ，基于最近最少使用原则删除多余缓存，保持最热的数据被缓存。

#### 3.1 LruCache

`com.alibaba.dubbo.cache.support.lru.LruCache` ，实现 Cache 接口，代码如下：

```java
public class LruCache implements Cache {
    // 缓存集合
    private final Map<Object, Object> store;

    public LruCache(URL url) {
        // "cache.size" 配置项，设置缓存大小
        final int max = url.getParameter("cache.size", 1000);
        // 创建 LRUCache 对象
        this.store = new LRUCache<Object, Object>(max);
    }
    @Override
    public void put(Object key, Object value) {
        store.put(key, value);
    }
    @Override
    public Object get(Object key) {
        return store.get(key);
    }
}
```

- `"cache.size"` 配置项，设置缓存**大小**。
- 基于 `com.alibaba.dubbo.common.utils.LRUCache` 实现。

##### 3.1.1 LRUCache

[`com.alibaba.dubbo.common.utils.LRUCache`](https://github.com/YunaiV/dubbo/blob/26d2f1811c23096224fc9a973b3526f01aabeb28/dubbo-common/src/main/java/com/alibaba/dubbo/common/utils/LRUCache.java) ，实现 LinkedHashMap 类，LRU 缓存实现类。

- 构造方法，设置 LRUCache 为**按访问顺序(调用get方法)**的链表。代码如下：

  ```java
  public LRUCache(int maxCapacity) {
      super(16, DEFAULT_LOAD_FACTOR, true); // 最后一个参数，按访问顺序(调用get方法)的链表
      this.maxCapacity = maxCapacity;
  }
  ```

- 重写 removeEldestEntry 方法返回 true 值，**指定插入元素时移除最老的元素。**代码如下：

  ```java
  @Override
  protected boolean removeEldestEntry(java.util.Map.Entry<K, V> eldest) {
      return size() > maxCapacity;
  }
  ```

  - 根据链表中元素的顺序可以分为：按插入顺序的链表，和按访问顺序(调用get方法)的链表。默认是按插入顺序排序，如果指定按访问顺序排序，那么调用get方法后，会将这次访问的元素移至链表尾部，不断访问可以形成按访问顺序排序的链表。

- `lock` 属性，锁。避免并发读写，导致死锁。参见 [《疫苗：JAVA HashMap 的死循环》](https://coolshell.cn/articles/9606.html) 。涉及该属性的方法示例：

  ```java
  @Override
  public V put(K key, V value) {
      try {
          lock.lock();
          return super.put(key, value);
      } finally {
          lock.unlock();
      }
  }
  @Override
  public V remove(Object key) {
      try {
          lock.lock();
          return super.remove(key);
      } finally {
          lock.unlock();
      }
  }
  ```

#### 3.2 LruCacheFactory

`com.alibaba.dubbo.cache.support.lru.LruCacheFactory` ，实现 AbstractCacheFactory 抽象类，代码如下：

```java
public class LruCacheFactory extends AbstractCacheFactory {
    @Override
    protected Cache createCache(URL url) {
        return new LruCache(url);
    }
}
```

### 4.ThreadLocal 实现

​	基于 **ThreadLocal** ，当前线程缓存，比如一个页面渲染，用到很多 portal，每个 portal 都要去查用户信息，通过线程缓存，可以减少这种多余访问。

#### 4.1 ThreadLocalCache

`com.alibaba.dubbo.cache.support.threadlocal.ThreadLocalCache` ，实现 Cache 接口，代码如下：

```java
public class ThreadLocalCache implements Cache {
    private final ThreadLocal<Map<Object, Object>> store; // 线程变量
    public ThreadLocalCache(URL url) {
        this.store = new ThreadLocal<Map<Object, Object>>() {
            @Override
            protected Map<Object, Object> initialValue() {
                return new HashMap<Object, Object>();
            }
        };
    }
    @Override
    public void put(Object key, Object value) {
        store.get().put(key, value);
    }
    @Override
    public Object get(Object key) {
        return store.get().get(key);
    }
}
```

- 基于 ThreadLocal 实现，相当于一个线程，一个 ThreadLocalCache 对象。
- ThreadLocalCache 目前没有过期或清理机制，所以**需要注意**。

#### 4.2 ThreadLocalCacheFactory

`com.alibaba.dubbo.cache.support.threadlocal.ThreadLocalCacheFactory` ，实现 AbstractCacheFactory 抽象类，代码如下：

```java
public class ThreadLocalCacheFactory extends AbstractCacheFactory {
    @Override
    protected Cache createCache(URL url) {
        return new ThreadLocalCache(url);
    }
}
```

### 5.JCache 实现

与 [JSR107](https://jcp.org/en/jsr/detail?id=107) 集成，可以桥接各种缓存实现。

#### 5.1 JCache

`com.alibaba.dubbo.cache.support.jcache.JCache` ，实现 Cache 接口，代码如下：

```java
public class JCache implements com.alibaba.dubbo.cache.Cache {
    private final Cache<Object, Object> store;
    public JCache(URL url) {
        // 获得 Cache Key
        String method = url.getParameter(Constants.METHOD_KEY, "");
        String key = url.getAddress() + "." + url.getServiceKey() + "." + method;
        // "jcache"配置项，为 Java SPI 实现的全限定类名
        // jcache parameter is the full-qualified class name of SPI implementation
        String type = url.getParameter("jcache");
        // 基于类型，获得 javax.cache.CachingProvider 对象，
        CachingProvider provider = type == null || type.length() == 0 ? Caching.getCachingProvider() : Caching.getCachingProvider(type);
        // 获得 javax.cache.CacheManager 对象
        CacheManager cacheManager = provider.getCacheManager();
        // 获得 javax.cache.Cache 对象
        Cache<Object, Object> cache = cacheManager.getCache(key);
        // 不存在，则进行创建
        if (cache == null) {
            try {
                // 设置 Cache 配置项
                // configure the cache
                MutableConfiguration config =
                        new MutableConfiguration<Object, Object>()
                                // 类型
                                .setTypes(Object.class, Object.class)
                                // 过期策略，按照写入时间过期。通过 `"cache.write.expire"` 配置项设置过期时间，默认为 1 分钟。
                               .setExpiryPolicyFactory(CreatedExpiryPolicy.factoryOf(new Duration(TimeUnit.MILLISECONDS, url.getMethodParameter(method, "cache.write.expire", 60 * 1000))))
                                .setStoreByValue(false)
                                // 设置 MBean
                                .setManagementEnabled(true)
                                .setStatisticsEnabled(true);
                // 创建 javax.cache.Cache 对象
                cache = cacheManager.createCache(key, config);
            } catch (CacheException e) {
                // 初始化 cache 的并发情况
                // concurrent cache initialization
                cache = cacheManager.getCache(key);
            }
        }
        this.store = cache;
    }
    @Override
    public void put(Object key, Object value) {
        store.put(key, value);
    }
    @Override
    public Object get(Object key) {
        return store.get(key);
    }
}
```

参照： [《 Java Caching(缓存)-策略和JCache API》](https://blog.csdn.net/boonya/article/details/54632129)

#### 5.2 JCacheFactory

`com.alibaba.dubbo.cache.support.jcache.JCacheFactory` ，实现 AbstractCacheFactory 抽象类，代码如下：

```java
public class JCacheFactory extends AbstractCacheFactory {
    @Override
    protected Cache createCache(URL url) {
        return new JCache(url);
    }
}
```

参见 [《精尽 Dubbo 源码分析 —— 过滤器（十）之 CacheFilter》](http://svip.iocoder.cn/Dubbo/filter-cache-filter/) 。









参照：芋道源码