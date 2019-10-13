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

###      Business层

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

### 注册中心挂了Dubbo还可以通信

​	**对于正在运行的 Consumer 调用 Provider 是不需要经过注册中心，所以不受影响。并且，Consumer 进程中，内存已经缓存了 Provider 列表。**

​	那么，此时 Provider 如果下线呢？如果 Provider 是**正常关闭**，它会主动且直接对和其处于连接中的 Consumer 们，发送一条“我要关闭”了的消息。那么，Consumer 们就不会调用该 Provider ，而调用其它的 Provider 。

另外，因为 Consumer 也会持久化 Provider 列表到本地文件。所以，此处如果 Consumer 重启，依然能够通过本地缓存的文件，获得到 Provider 列表。

==再另外，一般情况下，注册中心是一个集群，如果一个节点挂了，Dubbo Consumer 和 Provider 将自动切换到集群的另外一个节点上。==

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
- ==ThreadLocalCache 目前没有过期或清理机制==，所以**需要注意**。

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

# 七、Zookeeper存储的Dubbo信息

## 1.zookeeper 注册中心

​	[Zookeeper](http://zookeeper.apache.org/) 是 Apacahe Hadoop 的子项目，是一个树型的目录服务，支持变更推送，适合作为 Dubbo 服务的注册中心，工业强度较高，可用于生产环境，并推荐使用 [1](https://dubbo.gitbooks.io/dubbo-user-book/references/registry/zookeeper.html#fn_1)。

![/user-guide/images/zookeeper.jpg](/Users/jack/Desktop/md/images/zookeeper.jpg)

> - 在图中，我们可以看到 Zookeeper 的节点层级，自上而下是：
>   - **Root** 层：根目录，**可通过 `<dubbo:registry group="dubbo" />` 的 `"group"` 设置 Zookeeper 的根节点，缺省使用 `"dubbo"` 。**
>   - **Service** 层：服务接口全名。
>   - **Type** 层：分类。目前除了我们在图中看到的 `"providers"`( 服务提供者列表 ) `"consumers"`( 服务消费者列表 ) 外，还有 [`"routes"`](https://dubbo.gitbooks.io/dubbo-user-book/demos/routing-rule.html)( 路由规则列表 ) 和 [`"configurations"`](https://dubbo.gitbooks.io/dubbo-user-book/demos/config-rule.html)( 配置规则列表 )。
>   - **URL** 层：URL ，根据不同 Type 目录，**下面可以是服务提供者 URL 、服务消费者 URL 、路由规则 URL 、配置规则 URL 。**
>   - 实际上 URL 上带有 `"category"` 参数，已经能判断每个 URL 的分类，但是 Zookeeper 是基于节点目录订阅的，所以增加了 **Type**层。
> - 实际上，**服务消费者**启动后，不仅仅订阅了 `"providers"` 分类，也订阅了 `"routes"` `"configurations"` 分类。

流程说明：

- 服务提供者启动时: 向 `/dubbo/com.foo.BarService/providers` 目录下写入自己的 URL 地址
- 服务消费者启动时: **订阅 `/dubbo/com.foo.BarService/providers` 目录下的提供者 URL 地址。并向 `/dubbo/com.foo.BarService/consumers` 目录下写入自己的 URL 地址**
- 监控中心启动时: 订阅 `/dubbo/com.foo.BarService` 目录下的所有提供者和消费者 URL 地址。

支持以下功能：

- 当提供者出现断电等异常停机时，注册中心能自动删除提供者信息
- 当注册中心重启时，能自动恢复注册数据，以及订阅请求
- 当会话过期时，能自动恢复注册数据，以及订阅请求
- 当设置 `<dubbo:registry check="false" />` 时，记录失败注册和订阅请求，后台定时重试
- 可通过 `<dubbo:registry username="admin" password="1234" />` 设置 zookeeper 登录信息
- 可通过 `<dubbo:registry group="dubbo" />` 设置 zookeeper 的根节点，不设置将使用无根树
- 支持 `*` 号通配符 `<dubbo:reference group="*" version="*" />`，可订阅服务的所有分组和所有版本的提供者

### 使用 

在 provider 和 consumer 中增加 zookeeper 客户端 jar 包依赖，Dubbo 支持 zkclient 和 curator 两种 Zookeeper 客户端实现：

#### 使用 zkclient 客户端

从 `2.2.0` 版本开始缺省为 zkclient 实现，以提升 zookeeper 客户端的健状性。[zkclient](https://github.com/sgroschupf/zkclient) 是 Datameer 开源的一个 Zookeeper 客户端实现。

缺省配置：

```xml
<dubbo:registry ... client="zkclient" />
```

或：

```sh
dubbo.registry.client=zkclient
```

或：

```sh
zookeeper://10.20.153.10:2181?client=zkclient
```

需添加依赖或直接[下载](http://repo1.maven.org/maven2/com/github/sgroschupf/zkclient)：

```xml
<dependency>
    <groupId>com.github.sgroschupf</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.1</version>
</dependency>
```

#### 使用 curator 客户端

从 `2.3.0` 版本开始支持可选 curator 实现。[Curator](https://github.com/Netflix/curator) 是 Netflix 开源的一个 Zookeeper 客户端实现。

如果需要改为 curator 实现，请配置：

```xml
<dubbo:registry ... client="curator" />
```

或：

```sh
dubbo.registry.client=curator
```

或：

```sh
zookeeper://10.20.153.10:2181?client=curator
```

需依赖或直接[下载](http://repo1.maven.org/maven2/com/netflix/curator/curator-framework)：

```xml
<dependency>
    <groupId>com.netflix.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>1.1.10</version>
</dependency>
```

#### Zookeeper 单机配置:

```xml
<dubbo:registry address="zookeeper://10.20.153.10:2181" />
```

或：

```xml
<dubbo:registry protocol="zookeeper" address="10.20.153.10:2181" />
```

#### Zookeeper 集群配置：

```xml
<dubbo:registry address="zookeeper://10.20.153.10:2181?backup=10.20.153.11:2181,10.20.153.12:2181" />
```

或：

```xml
<dubbo:registry protocol="zookeeper" address="10.20.153.10:2181,10.20.153.11:2181,10.20.153.12:2181" />
```

同一 Zookeeper，分成多组注册中心:

```xml
<dubbo:registry id="chinaRegistry" protocol="zookeeper" address="10.20.153.10:2181" group="china" />
<dubbo:registry id="intlRegistry" protocol="zookeeper" address="10.20.153.10:2181" group="intl" />
```

参照： [《Dubbo 用户指南 —— zookeeper 注册中心》](https://dubbo.gitbooks.io/dubbo-user-book/references/registry/zookeeper.html)

# 八、Dubbo Provider 优雅停机

​	Dubbo 是通过 JDK 的 ShutdownHook 来完成优雅停机的，所以如果用户使用 `kill -9 PID` 等强制关闭指令，是不会执行优雅停机的，只有通过 `kill PID` 时，才会执行。

## 原理

### 服务提供方

- 停止时，先标记为不接收新请求，新请求过来时直接报错，让客户端重试其它机器。
- 然后，检测线程池中的线程是否正在运行，如果有，等待所有线程执行完成，除非超时，则强制关闭。

### 服务消费方

- 停止时，不再发起新的调用请求，所有新的调用在客户端即报错。
- 然后，检测有没有请求的响应还没有返回，等待响应返回，除非超时，则强制关闭。

## 设置方式

设置优雅停机超时时间，缺省超时时间是 10 秒，如果超时则强制关闭。

```properties
# dubbo.properties
dubbo.service.shutdown.wait=15000
```

​	如果 ShutdownHook 不能生效，可以自行调用，**使用tomcat等容器部署的場景，建议通过扩展ContextListener等自行调用以下代码实现优雅停机**：

```java
ProtocolConfig.destroyAll();
```

参照：[《Dubbo 用户指南 —— 优雅停机》](http://dubbo.apache.org/zh-cn/docs/user/demos/graceful-shutdown.html)

**服务提供方的优雅停机过程**

1. 首先，从注册中心中取消注册自己，从而使消费者不要再拉取到它。
2. 然后，sleep 10 秒( 可以自己配置 )，等到服务消费，接收到注册中心通知到该服务提供者已经下线，加大了在不重试情况下优雅停机的成功率。
3. 之后，广播 READONLY 事件给所有 Consumer 们，告诉它们不要在调用我了！**并且，如果此处注册中心挂掉的情况，依然能达到告诉 Consumer ，我要下线了的功能。**
4. 再之后，sleep 10 毫秒，保证 Consumer 们，尽可能接收到该消息。
5. 再再之后，先标记为不接收新请求，新请求过来时直接报错，让客户端重试其它机器。
6. 再再再之后，关闭心跳线程。
7. 最后，检测线程池中的线程是否正在运行，如果有，等待所有线程执行完成，除非超时，则强制关闭。
8. 最最后，关闭服务器。

详细参照： [《精尽 Dubbo 源码解析 —— 优雅停机》](http://svip.iocoder.cn/Dubbo/graceful-shutdown/) 。

**服务消费方的优雅停机过程**

1. 停止时，不再发起新的调用请求，所有新的调用在客户端即报错。
2. 然后，检测有没有请求的响应还没有返回，等待响应返回，除非超时，则强制关闭。

在使用Zookeeper作为注册中心时，服务提供者，注册到 Zookeeper 上时，创建的是 EPHEMERAL 临时节点。所以在服务提供者异常关闭时，等待 Zookeeper 会话超时，那么该临时节点就会自动删除。这样就可以解决Dubbo Provider 异步关闭时，从注册中心下线。

# 九、Dubbo Consumer 可调用注册中心外的 Provider

## 直连提供者

​	Consumer 可以强制直连 Provider 。在**开发及测试环境**下，经常需要绕过注册中心，只测试指定服务提供者，这时候可能需要点对点直连，点对点直连方式，将以服务接口为单位，忽略注册中心的提供者列表，A 接口配置点对点，不影响 B 接口从注册中心获取列表。

![/user-guide/images/dubbo-directly.jpg](/Users/jack/Desktop/md/images/dubbo-directly-20190703112226503.jpg)

### 通过 XML 配置

​	如果是线上需求需要点对点，可在 `<dubbo:reference>` 中配置 url 指向提供者，将绕过注册中心，多个地址用分号隔开，配置如下 ：

```xml
<dubbo:reference id="xxxService" interface="com.alibaba.xxx.XxxService" url="dubbo://localhost:20890" />
```

### 通过 -D 参数指定

在 JVM 启动参数中加入-D参数映射服务地址，如：

```sh
java -Dcom.alibaba.xxx.XxxService=dubbo://localhost:20890
```

### 通过文件映射

如果服务比较多，也可以用文件映射，用 `-Ddubbo.resolve.file` 指定映射文件路径，此配置优先级高于 `<dubbo:reference>` 中的配置，如：

```sh
java -Ddubbo.resolve.file=xxx.properties
```

然后在映射文件 `xxx.properties` 中加入配置，其中 key 为服务名，value 为服务提供者 URL：

```properties
com.alibaba.xxx.XxxService=dubbo://localhost:20890
```

**注意** 为了避免复杂化线上环境，不要在线上使用这个功能，只应在测试阶段使用。

参见 [《Dubbo 用户指南 —— 直连提供者》](http://dubbo.apache.org/zh-cn/docs/user/demos/explicit-target.html) 。

​	另外，直连 Dubbo Provider 时，如果要 Debug 调试 Dubbo Provider ，可以通过配置，禁用该 Provider 注册到注册中心。否则，会被其它 Consumer 调用到。

## 只订阅

为方便开发测试，经常会在线下共用一个所有服务可用的注册中心，这时，如果一个正在开发中的服务提供者注册，可能会影响消费者不能正常运行。

可以让服务提供者开发方，只订阅服务(开发的服务可能依赖其它服务)，而不注册正在开发的服务，通过直连测试正在开发的服务。

![/user-guide/images/subscribe-only.jpg](/Users/jack/Desktop/md/images/subscribe-only.jpg)

禁用注册配置

```xml
<dubbo:registry address="10.20.153.10:9090" register="false" />
```

或者

```xml
<dubbo:registry address="10.20.153.10:9090?register=false" />
```

参照： [《Dubbo 用户指南 —— 只订阅》](http://dubbo.apache.org/zh-cn/docs/user/demos/subscribe-only.html) 。

# 十、Dubbo 支持的通信协议

> 对应【protocol 远程调用层】。

Dubbo 目前支持如下 9 种通信协议：

- 【重要】`dubbo://` ，默认协议。参见 [《Dubbo 用户指南 —— dubbo://》](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/dubbo.html) 。

  ==Dubbo 缺省协议采用单一长连接和 NIO 异步通讯，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况。==

  反之，Dubbo 缺省协议不适合传送大数据量的服务，比如传文件，传视频等，除非请求量很低。

  ![dubbo-protocol.jpg](/Users/jack/Desktop/md/images/dubbo-protocol.jpg)

  > ## 配置
  >
  > 配置协议：
  >
  > ```xml
  > <dubbo:protocol name="dubbo" port="20880" />
  > ```
  >
  > 设置默认协议：
  >
  > ```xml
  > <dubbo:provider protocol="dubbo" />
  > ```
  >
  > 设置服务协议：
  >
  > ```xml
  > <dubbo:service protocol="dubbo" />
  > ```
  >
  > 多端口：
  >
  > ```xml
  > <dubbo:protocol id="dubbo1" name="dubbo" port="20880" />
  > <dubbo:protocol id="dubbo2" name="dubbo" port="20881" />
  > ```
  >
  > 配置协议选项：
  >
  > ```xml
  > <dubbo:protocol name=“dubbo” port=“9090” server=“netty” client=“netty” codec=“dubbo” serialization=“hessian2” charset=“UTF-8” threadpool=“fixed” threads=“100” queues=“0” iothreads=“9” buffer=“8192” accepts=“1000” payload=“8388608” />
  > ```
  >
  > 多连接配置：
  >
  > Dubbo 协议缺省每服务每提供者每消费者使用单一长连接，如果数据量较大，可以使用多个连接。
  >
  > ```xml
  > <dubbo:service connections="1"/>
  > <dubbo:reference connections="1"/>
  > ```
  >
  > - `<dubbo:service connections="0">` 或 `<dubbo:reference connections="0">` 表示该服务使用 JVM 共享长连接。**缺省**
  > - `<dubbo:service connections="1">` 或 `<dubbo:reference connections="1">` 表示该服务使用独立长连接。
  > - `<dubbo:service connections="2">` 或`<dubbo:reference connections="2">` 表示该服务使用独立两条长连接。
  >
  > 为防止被大量连接撑挂，可在服务提供方限制大接收连接数，以实现服务提供方自我保护。
  >
  > ```xml
  > <dubbo:protocol name="dubbo" accepts="1000" />
  > ```
  >
  > `dubbo.properties` 配置：
  >
  > ```sh
  > dubbo.service.protocol=dubbo
  > ```

- 【重要】`rest://` ，贡献自 Dubbox ，目前最合适的 HTTP Restful API 协议。参见 [《Dubbo 用户指南 —— rest://》](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/rest.html) 。

  > ## 快速入门
  >
  > 在dubbo中开发一个REST风格的服务会比较简单，下面以一个注册用户的简单服务为例说明。
  >
  > 这个服务要实现的功能是提供如下URL（注：这个URL不是完全符合REST的风格，但是更简单实用）：
  >
  > ```
  > http://localhost:8080/users/register
  > ```
  >
  > 而任何客户端都可以将包含用户信息的JSON字符串POST到以上URL来完成用户注册。
  >
  > 首先，开发服务的接口：
  >
  > ```java
  > public interface UserService {    
  >    void registerUser(User user);
  > }
  > ```
  >
  > 然后，开发服务的实现：
  >
  > ```java
  > @Path("/users")
  > public class UserServiceImpl implements UserService {
  >        
  >     @POST
  >     @Path("/register")
  >     @Consumes({MediaType.APPLICATION_JSON})
  >     public void registerUser(User user) {
  >         // save the user...
  >     }
  > }
  > ```
  >
  > 上面的实现非常简单，但是由于该 REST 服务是要发布到指定 URL 上，供任意语言的客户端甚至浏览器来访问，所以这里额外添加了几个 JAX-RS 的标准 annotation 来做相关的配置。
  >
  > @Path("/users")：指定访问UserService的URL相对路径是/users，即http://localhost:8080/users
  >
  > @Path("/register")：指定访问registerUser()方法的URL相对路径是/register，再结合上一个@Path为UserService指定的路径，则调用UserService.register()的完整路径为http://localhost:8080/users/register
  >
  > @POST：指定访问registerUser()用HTTP POST方法
  >
  > @Consumes({MediaType.APPLICATION_JSON})：指定registerUser()接收JSON格式的数据。REST框架会自动将JSON数据反序列化为User对象
  >
  > 最后，在spring配置文件中添加此服务，即完成所有服务开发工作：
  >
  > ```xml
  > <!-- 用rest协议在8080端口暴露服务 -->
  > <dubbo:protocol name="rest" port="8080"/>
  > 
  > <!-- 声明需要暴露的服务接口 -->
  > <dubbo:service interface="xxx.UserService" ref="userService"/>
  > 
  > <!-- 和本地bean一样实现服务 -->
  > <bean id="userService" class="xxx.UserServiceImpl" />
  > ```

- `rmi://` ，参见 [《Dubbo 用户指南 —— rmi://》](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/rmi.html) 。

- `webservice://` ，参见 [《Dubbo 用户指南 —— webservice://》](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/webservice.html) 。

- `hessian://` ，参见 [《Dubbo 用户指南 —— hessian://》](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/hessian.html) 。

- `thrift://` ，参见 [《Dubbo 用户指南 —— thrift://》](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/thrift.html) 。

- `memcached://` ，参见 [《Dubbo 用户指南 —— memcached://》](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/memcached.html) 。

- `redis://` ，参见 [《Dubbo 用户指南 —— redis://》](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/redis.html) 。

- `http://` ，参见 [《Dubbo 用户指南 —— http://》](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/http.html) 。注意，这个和我们理解的 HTTP 协议有差异，而是 Spring 的 HttpInvoker 实现。

实际上，社区里还有其他通信协议正处于孵化：

- `jsonrpc://` ，对应 Github 仓库为 <https://github.com/apache/incubator-dubbo-rpc-jsonrpc> ，来自千米网的贡献。

每一种通信协议的实现，在 [《精尽 Dubbo 源码解析》](http://svip.iocoder.cn/categories/Dubbo/) 中，都有详细解析。

另外，在 [《Dubbo 用户指南 —— 性能测试报告》](http://dubbo.apache.org/zh-cn/docs/user/perf-test.html) 中，官方提供了上述协议的性能测试对比。

## Dubbo使用的通信框架

Dubbo 在通信层拆分成了 API 层、实现层。项目结构如下：

- API 层：
  - `dubbo-remoting-api`
- 实现层：
  - `dubbo-remoting-netty3`
  - `dubbo-remoting-netty4`
  - `dubbo-remoting-mina`
  - `dubbo-remoting-grizzly`

再配合上 Dubbo SPI 的机制，使用者可以自定义使用哪一种具体的实现。美滋滋。

在 Dubbo 的最新版本，默认使用 **Netty4** 的版本。

# 十一、本地调用和远程调用

​	每次 Consumer 调用 Provider 都是跨进程，需要进行网络通信。

### 本地调用

本地调用使用了 injvm 协议，是一个伪协议，它不开启端口，不发起远程调用，只在 JVM 内直接关联，但执行 Dubbo 的 Filter 链。

### 配置

定义 injvm 协议

```xml
<dubbo:protocol name="injvm" />
```

设置默认协议

```xml
<dubbo:provider protocol="injvm" />
```

设置服务协议

```xml
<dubbo:service protocol="injvm" />
```

优先使用 injvm

```xml
<dubbo:consumer injvm="true" .../>
<dubbo:provider injvm="true" .../>
```

或

```xml
<dubbo:reference injvm="true" .../>
<dubbo:service injvm="true" .../>
```

注意：dubbo从 `2.2.0` 每个服务默认都会在本地暴露,无需进行任何配置即可进行本地引用,如果不希望服务进行远程暴露,只需要在provider将protocol设置成injvm即可

### 自动暴露、引用本地服务

从 `2.2.0` 开始，每个服务默认都会在本地暴露。在引用服务的时候，默认优先引用本地服务。如果希望引用远程服务可以使用一下配置强制引用远程服务。

```xml
<dubbo:reference ... scope="remote" />
```

# 十二、Dubbo序列化的方式

> 对应【serialize 数据序列化层】。

Dubbo 目前支付如下 7 种序列化方式：

- 【重要】Hessian2 ：基于 Hessian 实现的序列化拓展。dubbo://   协议的默认序列化方案。

  - Hessian 除了是 Web 服务，也提供了其序列化实现，因此 Dubbo 基于它实现了序列化拓展。
  - 另外，Dubbo 维护了自己的 [`hessian-lite`](https://github.com/alibaba/dubbo/tree/4bbc0ddddacc915ddc8ff292dd28745bbc0031fd/hessian-lite) ，对 [Hessian 2](http://hessian.caucho.com/) 的 **序列化** 部分的精简、改进、BugFix 。

- Dubbo ：Dubbo 自己实现的序列化拓展。

  - 具体可参见 [《精尽 Dubbo 源码分析 —— 序列化（二）之 Dubbo 实现》](http://svip.iocoder.cn/Dubbo/serialize-2-dubbo/) 。

- Kryo ：基于Kryo实现的序列化拓展。

- FST ：基于FST实现的序列化拓展。

  > ## 启用Kryo和FST
  >
  > 使用Kryo和FST非常简单，只需要在dubbo RPC的XML配置中添加一个属性即可：
  >
  > ```xml
  > <dubbo:protocol name="dubbo" serialization="kryo"/>
  > <dubbo:protocol name="dubbo" serialization="fst"/>
  > ```
  >
  > ## 注册被序列化类
  >
  > 要让Kryo和FST完全发挥出高性能，最好将那些需要被序列化的类注册到dubbo系统中，例如，我们可以实现如下回调接口：
  >
  > ```java
  > public class SerializationOptimizerImpl implements SerializationOptimizer {
  >     public Collection<Class> getSerializableClasses() {
  >         List<Class> classes = new LinkedList<Class>();
  >         classes.add(BidRequest.class);
  >         classes.add(BidResponse.class);
  >         classes.add(Device.class);
  >         classes.add(Geo.class);
  >         classes.add(Impression.class);
  >         classes.add(SeatBid.class);
  >         return classes;
  >     }
  > }
  > ```
  >
  > 然后在XML配置中添加：
  >
  > ```xml
  > <dubbo:protocol name="dubbo" serialization="kryo" optimizer="org.apache.dubbo.demo.SerializationOptimizerImpl"/>
  > ```
  >
  > 在注册这些类后，序列化的性能可能被大大提升，特别针对小数量的嵌套对象的时候。
  >
  > 当然，在对一个类做序列化的时候，可能还级联引用到很多类，比如Java集合类。针对这种情况，我们已经自动将JDK中的常用类进行了注册，所以你不需要重复注册它们（当然你重复注册了也没有任何影响），包括：
  >
  > ```
  > GregorianCalendar
  > InvocationHandler
  > BigDecimal
  > BigInteger
  > Pattern
  > BitSet
  > URI
  > UUID
  > HashMap
  > ArrayList
  > LinkedList
  > HashSet
  > TreeSet
  > Hashtable
  > Date
  > Calendar
  > ConcurrentHashMap
  > SimpleDateFormat
  > Vector
  > BitSet
  > StringBuffer
  > StringBuilder
  > Object
  > Object[]
  > String[]
  > byte[]
  > char[]
  > int[]
  > float[]
  > double[]
  > ```
  >
  > 由于注册被序列化的类仅仅是出于性能优化的目的，所以即使你忘记注册某些类也没有关系。事实上，即使不注册任何类，Kryo和FST的性能依然普遍优于hessian和dubbo序列化。
  >
  > 参照 [《Dubbo 用户指南 —— FST 序列化》](http://dubbo.apache.org/zh-cn/docs/user/demos/serialization.html)

- JSON ：基于 [Fastjson](https://www.oschina.net/p/fastjson) 实现的序列化拓展。

- NativeJava ：基于 Java 原生的序列化拓展。

- CompactedJava ：在 **NativeJava** 的基础上，实现了对 ClassDescriptor 的处理。

微博的 Motan 有实现对 Protobuf 序列化的支持，可以看 [《深入理解RPC之序列化篇 —— 总结篇》](https://www.cnkirito.moe/rpc-serialize-2/) 的 [「Protostuff实现」](http://svip.iocoder.cn/Dubbo/Interview/#) 小节。

# 十三、Dubbo负载均衡策略

 Dubbo 内置 4 种负载均衡策略。其中，默认使用 `random` 随机调用策略。

## 负载均衡策略

### Random LoadBalance

- **随机**，按权重设置随机概率。
- 在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。

### RoundRobin LoadBalance

- **轮询**，按公约后的权重设置轮询比率。
- 存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上。

### LeastActive LoadBalance

- **最少活跃调用数**，相同活跃数的随机，活跃数指调用前后计数差。
- 使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。

### ConsistentHash LoadBalance

- **一致性 Hash**，相同参数的请求总是发到同一提供者。
- 当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。
- 缺省只对第一个参数 Hash，如果要修改，请配置 `<dubbo:parameter key="hash.arguments" value="0,1" />`
- 缺省用 160 份虚拟节点，如果要修改，请配置 `<dubbo:parameter key="hash.nodes" value="320" />`

## 配置

### 服务端服务级别

```xml
<dubbo:service interface="..." loadbalance="roundrobin" />
```

### 客户端服务级别

```xml
<dubbo:reference interface="..." loadbalance="roundrobin" />
```

### 服务端方法级别

```xml
<dubbo:service interface="...">
    <dubbo:method name="..." loadbalance="roundrobin"/>
</dubbo:service>
```

### 客户端方法级别

```xml
<dubbo:reference interface="...">
    <dubbo:method name="..." loadbalance="roundrobin"/>
</dubbo:reference>
```

参照：[《Dubbo 用户指南 —— 负载均衡》](http://dubbo.apache.org/zh-cn/docs/user/demos/loadbalance.html) 

# 十四、Dubbo集群容错策略

> 对应【cluster 路由层】的 Cluster 组件。

​	在集群调用失败时，Dubbo 提供了多种容错方案，缺省为 failover 重试。

![cluster](/Users/jack/Desktop/md/images/cluster.jpg)

各节点关系：

- 这里的 `Invoker` 是 `Provider` 的一个可调用 `Service` 的抽象，`Invoker` 封装了 `Provider` 地址及 `Service` 接口信息
- `Directory` 代表多个 `Invoker`，可以把它看成 `List<Invoker>` ，但与 `List` 不同的是，它的值可能是动态变化的，比如**注册中心推送变更**
- `Cluster` 将 `Directory` 中的多个 `Invoker` 伪装成一个 `Invoker`，对上层透明，伪装过程包含了容错逻辑，调用失败后，重试另一个
- `Router` 负责从多个 `Invoker` 中按路由规则选出子集，比如读写分离，应用隔离等
- `LoadBalance` 负责从多个 `Invoker` 中选出具体的一个用于本次调用，选的过程包含了负载均衡算法，调用失败后，需要重选

Dubbo提供了6种集群容错策略：

### Failover Cluster

​	失败自动切换，当出现失败，重试其它服务器。通常用于读操作，但重试会带来更长延迟。可通过 `retries="2"` 来设置重试次数(不含第一次)。

重试次数配置有以下三种方式：

```xml
<dubbo:service retries="2" />
<dubbo:reference retries="2" />
<dubbo:reference>
    <dubbo:method name="findFoo" retries="2" />
</dubbo:reference>
```

### Failfast Cluster

​	快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。

### Failsafe Cluster

​	失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。

### Failback Cluster

​	失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。

### Forking Cluster

​	并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 `forks="2"` 来设置最大并行数。

### Broadcast Cluster

​	广播调用所有提供者，逐个调用，任意一台报错则报错 [[2\]](http://dubbo.apache.org/zh-cn/docs/user/demos/fault-tolerent-strategy.html#fn2)。通常用于通知所有提供者更新缓存或日志等本地资源信息。

## 集群模式配置

按照以下两种方式在服务提供方和消费方配置集群模式

```xml
<dubbo:service cluster="failsafe" />
<dubbo:reference cluster="failsafe" />
```

## 失败重试

​	所谓失败重试，就是 consumer 调用 provider 要是失败了，比如抛异常了，此时应该是可以重试的，或者调用超时了也可以重试。

​	实际场景下，我们一般会**禁用掉重试**。因为，因为超时后重试会有问题，超时你不知道是成功还是失败。例如，可能会导致两次扣款的问题。

所以，**我们一般使用 failfast 集群容错策略，而不是 failover 策略。**配置如下：

```xml
<dubbo:service cluster="failfast" timeout="2000" />
```

另外，一定一定一定要配置适合自己业务的**超时时间**。

当然，可以将操作分成**读**和**写**两种，前者支持重试，后者不支持重试。因为，**读**操作天然具有幂等性。

# 十五、Dubbo的动态代理策略

> 对应【proxy 服务代理层】。

通过动态创建对应调用 Dubbo 服务的实现类。简化代码如下：

```java
// ProxyFactory.java
/**
 * create proxy.
 * 创建 Proxy ，在引用服务调用。
 * @param invoker Invoker 对象
 * @return proxy
 */
@Adaptive({Constants.PROXY_KEY})
<T> T getProxy(Invoker<T> invoker) throws RpcException;
```

- 方法参数 `invoker` ，实现了调用 Dubbo 服务的逻辑。
- 返回的 `<T>` 结果，就是 XXXService 的实现类，而这个实现类，就是通过动态代理的**工具类**进行生成。

通过动态代理的方式，实现了对于我们开发使用 Dubbo 时，透明的效果。当然，因为实际场景下，我们是结合 Spring 场景在使用，所以不会直接使用该 API 。

------

目前实现动态代理的**工具类**还是蛮多的，如下：

- Javassist
- JDK *原生自带*
- CGLIB
- ASM

其中，Dubbo 动态代理使用了 Javassist 和 JDK 两种方式。

- 默认情况下，使用 Javassist 。
- 可通过 SPI 机制，切换使用 JDK 的方式。

**为什么默认使用 Javassist？**

在 Dubbo 开发者【梁飞】的博客 [《动态代理方案性能对比》](https://javatar.iteye.com/blog/814426) 中，我们可以看到这几种方式的性能差异，而 Javassit 排在第一。也就是说，因为**性能**的原因。

​	有一点需要注意，Javassit 提供**字节码** bytecode 生成方式和动态代理接口两种方式。后者的性能比 JDK 自带的还慢，所以 Dubbo 使用的是前者**字节码** bytecode 生成方式。

​	实际上，JDK 代理在 JDK 1.8 版本下，性能已经有很大的提升，并且无需引入三方工具的依赖，也是非常棒的选择。所以，Spring 和 Motan 在动态代理生成上，优先选择 JDK 代理。

> 注意，Spring 同时也选择了 CGLIB 作为生成动态代理的工具之一。

推荐阅读徐妈的 [《深入理解 RPC 之动态代理篇》](http://www.iocoder.cn/RPC/laoxu/rpc-dynamic-proxy/) 。

# 十六、Dubbo的SPI实现思想

## 1.SPI机制

​	JDK提供的SPI(Service Provider Interface)机制，主要是针对厂商或者插件的，也可以在一些框架的扩展中看到。其核心类`java.util.ServiceLoader`可以在[jdk1.8](https://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html)的文档中看到详细的介绍，这个类用于加载固定类路径下文件的一个加载器，正是它加载了对应接口声明的实现类。

### SPI 在实际项目中的应用

​	通过SPI扩展的包或者插件，在其jar包目录下都会有个resources/META-INF/services这样子的文件，比如下面的数据库驱动包。

![image-20190704152931432](/Users/jack/Desktop/md/images/image-20190704152931432.png)

1. 在mysql-connector-java-xxx.jar中发现了META-INF\services\java.sql.Driver文件，里面只有两行记录：

   ```
   com.mysql.jdbc.Driver
   com.mysql.fabric.jdbc.FabricMySQLDriver
   ```

   `java.sql.Driver`是一个规范接口，`com.mysql.jdbc.Driver`
   `com.mysql.fabric.jdbc.FabricMySQLDriver`则是mysql-connector-java-xxx.jar对这个规范的实现接口。

2. 在jcl-over-slf4j-xxxx.jar中发现了META-INF\services\org.apache.commons.logging.LogFactory文件，里面只有一行记录：

   ```
   org.apache.commons.logging.impl.SLF4JLogFactory
   ```

### SPI 在扩展方面的应用

​	SPI不仅仅是为厂商指定的标准，同样也为框架扩展提供了一个思路。框架可以预留出SPI接口，这样可以在不侵入代码的前提下，通过增删依赖来扩展框架。前提是，框架得预留出核心接口，也就是本例中interface模块中类似的接口，剩下的适配工作便留给了开发者。

参照：[《JAVA 拾遗 —— 关于 SPI 机制》](http://www.iocoder.cn/Fight/xuma/spi/) 。

## 2.Dubbo中SPI的实现

​	之所以Dubbo会自己实现SPI主要是因为：

- **JDK 标准的 SPI 会一次性实例化扩展点所有实现，**如果有扩展实现初始化很耗时，但如果没用上也加载，会很浪费资源。

  Dubbo 有很多的拓展点，例如 Protocol、Filter 等等。并且每个拓展点有多种的实现，例如 Protocol 有 DubboProtocol、InjvmProtocol、RestProtocol 等等。那么使用 JDK SPI 机制，会初始化无用的拓展点及其实现，造成不必要的耗时与资源浪费。

- 如果扩展点加载失败，连扩展点的名称都拿不到了。比如：JDK 标准的 ScriptEngine，通过 getName() 获取脚本类型的名称，但如果 RubyScriptEngine 因为所依赖的 jruby.jar 不存在，导致 RubyScriptEngine 类加载失败，这个失败原因被吃掉了，和 ruby 对应不起来，当用户执行 ruby 脚本时，会报不支持 ruby，而不是真正失败的原因。

- 增加了对扩展点 IoC 和 AOP 的支持，一个扩展点可以直接 setter 注入其它扩展点。

想要自定义一个 Dubbo SPI 某个拓展点的实现，可以阅读 [《Dubbo 开发指南 —— 扩展点加载》](http://dubbo.apache.org/zh-cn/docs/dev/SPI.html) 。当然，如果你是首次写，可能会有一丢丢复杂。实际场景下，我们写的最多的是 [Filter 调用拦截扩展](http://dubbo.apache.org/zh-cn/docs/dev/impls/filter.html) 。

# 十七、Dubbo服务降级

比如说服务 A 调用服务 B，结果服务 B 挂掉了。服务 A 再重试几次调用服务 B，还是不行，那么直接降级，走一个备用的逻辑，给用户返回响应。

在 Dubbo 中，实现服务降级的功能，一共有两大种方式。

**① Dubbo 原生自带的服务降级功能**

​	可以通过服务降级功能临时屏蔽某个出错的非关键服务，并定义降级后的返回策略。向注册中心写入动态配置覆盖规则：

```java
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));
registry.register(URL.valueOf("override://0.0.0.0/com.foo.BarService?category=configurators&dynamic=false&application=foo&mock=force:return+null"));
```

其中：

- `mock=force:return+null` 表示消费方对该服务的方法调用都直接返回 null 值，不发起远程调用。用来屏蔽不重要服务不可用时对调用方的影响。
- 还可以改为 `mock=fail:return+null` 表示消费方对该服务的方法调用在失败后，再返回 null 值，不抛异常。用来容忍不重要服务不稳定时对调用方的影响。

**② 引入支持服务降级的组件**

目前开源社区常用的有两种组件支持服务降级的功能，分别是：

- Alibaba Sentinel
- Netflix Hystrix

目前 Hystrix 已经停止维护，并且和 Dubbo 的集成度不是特别高，需要做二次开发，所以推荐使用 Sentinel 。

> 随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
>
> Sentinel 具有以下特征:
>
> - **丰富的应用场景**：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
> - **完备的实时监控**：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
> - **广泛的开源生态**：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
> - **完善的 SPI 扩展点**：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。
>
> Sentinel 的主要特性：
>
> ![Sentinel-features-overview](/Users/jack/Desktop/md/images/50505538-2c484880-0aaf-11e9-9ffc-cbaaef20be2b.png)
>
> Sentinel 分为两个部分:
>
> - 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
> - 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。
>
> 参照： [《Sentinel 介绍》](https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D)

# 十八、Dubbo升级接口及令牌验证

当一个接口实现，出现不兼容升级时，**可以用版本号过渡**，版本号不同的服务相互间不引用。

可以按照以下的步骤进行版本迁移：

1. 在低压力时间段，先升级一半提供者为新版本。
2. 再将所有消费者升级为新版本。
3. 然后将剩下的一半提供者升级为新版本。

利用多版本的特性，我们也能实现灰度的功能。对于第 2 步，不要升级所有消费者为新版本，而是一半。

老版本服务提供者配置：

```xml
<dubbo:service interface="com.foo.BarService" version="1.0.0" />
```

新版本服务提供者配置：

```xml
<dubbo:service interface="com.foo.BarService" version="2.0.0" />
```

老版本服务消费者配置：

```xml
<dubbo:reference id="barService" interface="com.foo.BarService" version="1.0.0" />
```

新版本服务消费者配置：

```xml
<dubbo:reference id="barService" interface="com.foo.BarService" version="2.0.0" />
```

如果不需要区分版本，可以按照以下的方式配置:

```xml
<dubbo:reference id="barService" interface="com.foo.BarService" version="*" />
```

## 令牌验证

​	通过令牌验证在注册中心控制权限，以决定要不要下发令牌给消费者，可以防止消费者绕过注册中心访问提供者，另外通过注册中心可灵活改变授权方式，而不需修改或升级提供者

![/user-guide/images/dubbo-token.jpg](/Users/jack/Desktop/md/images/dubbo-token.jpg)

可以全局设置开启令牌验证：

```xml
<!--随机token令牌，使用UUID生成-->
<dubbo:provider interface="com.foo.BarService" token="true" />
```

或

```xml
<!--固定token令牌，相当于密码-->
<dubbo:provider interface="com.foo.BarService" token="123456" />
```

也可在服务级别设置：

```xml
<!--随机token令牌，使用UUID生成-->
<dubbo:service interface="com.foo.BarService" token="true" />
```

或

```xml
<!--固定token令牌，相当于密码-->
<dubbo:service interface="com.foo.BarService" token="123456" />
```













参考与推荐如下文章：

- [《Dubbo 用户指南》](http://dubbo.apache.org/zh-cn/docs/user/quick-start.html) 必选。
- [《Dubbo 开发指南》](http://dubbo.apache.org/zh-cn/docs/dev/build.html) 进阶。
- [《Dubbo 运维管理》](http://dubbo.apache.org/zh-cn/docs/admin/install/provider-demo.html) 可选。
- [《分布式系统互斥性与幂等性问题的分析与解决》](https://tech.meituan.com/distributed_system_mutually_exclusive_idempotence_cerberus_gtis.html)
- 黑马程序员 [《【上海校区】整理的 Dubbo 面试题》](https://blog.csdn.net/liyanlei5858/article/details/79236685)
- 美团 [《说一下 Dubbo 的工作原理？注册中心挂了可以继续通信吗？》](https://github.com/doocs/advanced-java/blob/master/docs/distributed-system/dubbo-operating-principle.md)
- lijiaccy [《Java 面试之 Dubbo》](https://www.jianshu.com/p/9d062eceb765)
- Java 知音 [Dubbo 面试题](https://juejin.im/entry/5b3af5fd518825621d57791a)
- [《Dubbo 支持哪些序列化协议？说一下 Hessian 的数据结构？PB 知道吗？为什么 PB 的效率是最高的？》](https://github.com/doocs/advanced-java/blob/master/docs/distributed-system/dubbo-serialization-protocol.md)
- [《Dubbo 的 SPI 思想是什么？》](https://github.com/doocs/advanced-java/blob/master/docs/distributed-system/dubbo-spi.md)
- [《如何基于 Dubbo 进行服务治理、服务降级、失败重试以及超时重试？》](https://github.com/doocs/advanced-java/blob/master/docs/distributed-system/dubbo-service-management.md)
- [《分布式服务接口的幂等性如何设计（比如不能重复扣款）？》](https://github.com/doocs/advanced-java/blob/master/docs/distributed-system/distributed-system-idempotency.md)
- [《为什么要进行系统拆分？如何进行系统拆分？拆分后不用 Dubbo 可以吗？》](https://github.com/doocs/advanced-java/blob/master/docs/distributed-system/why-dubbo.md)
- [《如何自己设计一个类似 Dubbo 的 rpc 框架？》](https://github.com/doocs/advanced-java/blob/master/docs/distributed-system/dubbo-rpc-design.md)