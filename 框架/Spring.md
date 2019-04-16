![image-20190416164514391](/Users/jack/Desktop/md/images/image-20190416164514391.png)

![image-20190416164642115](/Users/jack/Desktop/md/images/image-20190416164642115.png)

# 一、IoC

![image-20190416172122134](/Users/jack/Desktop/md/images/image-20190416172122134.png)

## 1、IoC 容器

​	**Spring 框架的核心是 Spring IoC 容器。容器创建 Bean 对象，将它们装配在一起，配置它们并管理它们的完整生命周期。**

- Spring 容器使用依赖注入来管理组成应用程序的 Bean 对象。
- **容器通过读取提供的配置元数据Bean Definition，** 来接收对象进行实例化，配置和组装的指令。
- ==该配置元数据 Bean Definition 可以通过 XML，Java 注解或 Java Config 代码提供。==

![Spring IoC](/Users/jack/Desktop/md/images/02.jpg)

## 2、依赖注入

在依赖注入中，你不必主动、手动创建对象，但必须描述如何创建它们。

- 你不是直接在代码中将组件和服务连接在一起，而是描述配置文件中哪些组件需要哪些服务。
- 然后，再由 IoC 容器将它们装配在一起。

另外，依赖注入的英文缩写是 Dependency Injection ，简称 DI 。

## 3、实现依赖注入的方式

通常，依赖注入可以通过三种方式完成，即：

- 接口注入

- 构造函数(即构造器)注入

  - ```Java
    /*带参数，方便利用构造器进行注入*/ 
    public CatDaoImpl(String message){
    	this. message = message; 
    }
    //XML
    <bean id="CatDaoImpl" class="com.CatDaoImpl"> 
    	<constructor-arg value=" message "></constructor-arg>
    </bean>
    ```

- setter 注入

  - ```Java
    public class Id {
    	private int id;
    	public int getId() { return id; }
    	public void setId(int id) { this.id = id; }
    }
    <bean id="id" class="com.id "> 
        <property name="id" value="123"></property> 
    </bean>
    ```

目前，在 Spring Framework 中，仅使用构造函数和 setter 注入这**两种**方式。

参照： [《Spring两种依赖注入方式的比较》](https://my.oschina.net/itblog/blog/203746)。综述来说：

| 构造函数注入               | setter 注入                |
| -------------------------- | -------------------------- |
| 没有部分注入               | 有部分注入                 |
| 不会覆盖 setter 属性       | 会覆盖 setter 属性         |
| 任意修改都会创建一个新实例 | 任意修改不会创建一个新实例 |
| 适用于设置很多属性         | 适用于设置少量属性         |

- 实际场景下，setter 注入使用的更多。

## 4、Spring中的IoC 容器(***ApplicationContext*** 面向开发应用)

Spring 提供了两种IoC 容器，分别是 BeanFactory、ApplicationContext 。

**BeanFactory**

> BeanFactory 在 `spring-beans` 项目提供。

BeanFactory ，就像一个包含 Bean 集合的工厂类。它会在客户端要求时实例化 Bean 对象。

可以理解为就是个 HashMap，Key 是 BeanName，Value 是 Bean 实例。通常只提供注册（put），获取（get）这两个功能。我们可以称之为 **“低级容器”**。

**ApplicationContext**

> ApplicationContext 在 `spring-context` 项目提供。

**ApplicationContext可以称之为 “高级容器”,因为他 接口扩展了 BeanFactory 接口，它在 BeanFactory 基础上提供了一些额外的功能。内置如下功能**：

- MessageSource ：管理 message ，实现国际化等功能，为应用提供 i18n 国际化消息访问的功能;
- ApplicationEventPublisher ：事件发布，让容器拥有发布应用上下文事件的功能，包括容器启动事件、关闭事件等。
- ResourcePatternResolver ：多资源加载，所有ApplicationContext实现类都实现了类似于
  PathMatchingResourcePatternResolver 的功能，可以通过带前缀的 Ant 风格的资源文件路径装载 Spring 的配置文件。
- EnvironmentCapable ：系统 Environment（profile + Properties）相关。
- Lifecycle ：管理生命周期。该接口提供了 start()和 stop()两个方法，主要用于控制异步处理过程。在具体使用时，该接口同时被 ApplicationContext 实现及具体 Bean 实现， ApplicationContext 会将 start/stop 的信息传递给容器中所有实现了该接 口的 Bean，以达到管理和控制 JMX、任务调度等目的。
- Closable ：关闭，释放资源
- InitializingBean：自定义初始化。
- BeanNameAware：设置 beanName 的 Aware 接口。
- ConfigurableApplicationContext 扩展于 ApplicationContext，**它新增加了两个主要 的方法: refresh()和 close()，让 ApplicationContext 具有启动、刷新和关闭应用上下 文的能力。**在应用上下文关闭的情况下调用 refresh()即可启动应用上下文，在已经启动 的状态下，调用 refresh()则清除缓存并重新装载配置信息，而调用 close()则可关闭应用 上下文。

另外，ApplicationContext 会自动初始化非懒加载的 Bean 对象们。

可以细看 [《【死磕 Spring】—— ApplicationContext 相关接口架构分析》](http://svip.iocoder.cn/Spring/ApplicationContext/) 

总结下 BeanFactory 与 ApplicationContext 两者的差异：

| BeanFactory                | ApplicationContext       |
| -------------------------- | ------------------------ |
| 它使用懒加载               | 它使用即时加载           |
| 它使用语法显式提供资源对象 | 它自己创建和管理资源对象 |
| 不支持国际化               | 支持国际化               |
| 不支持基于依赖的注解       | 支持基于依赖的注解       |

另外，BeanFactory 也被称为**低级**容器，而 ApplicationContext 被称为**高级**容器。

​	为了更直观的展示 “低级容器” 和 “高级容器” 的关系，下面通过常用的 ClassPathXmlApplicationContext 类，来展示整个容器的层级 UML 关系。

![img](/Users/jack/Desktop/md/images/4236553-1e6ca4c8a58c9e8e.png)

*ListableBeanFactory* 

​	该接口定义了访问容器中 Bean 基本信息的若干方法，如查看 Bean 的个数、获取某一类型 Bean 的配置名、查看容器中是否包括某一 Bean 等方法; 

*HierarchicalBeanFactory* 父子级联 

​	**父子级联 IoC 容器的接口，子容器可以通过接口方法访问父容器; 通过 HierarchicalBeanFactory 接口， Spring 的 IoC 容器可以建立父子层级关联的容器体系，子 容器可以访问父容器中的 Bean，但父容器不能访问子容器的 Bean。**Spring 使用父子容器实 现了很多功能，比如在 Spring MVC 中，展现层 Bean 位于一个子容器中，而业务层和持久 层的 Bean 位于父容器中。这样，展现层 Bean 就可以引用业务层和持久层的 Bean，而业务 层和持久层的 Bean 则看不到展现层的 Bean。 

​	看下面的隶属 ApplicationContext 粉红色的 “高级容器”，依赖着 “低级容器”，这里说的是依赖，不是继承,因为ApplicationContext继承了ListableBeanFactory，而ListableBeanFactory又继承了BeanFactory。他依赖着 “低级容器” 的 getBean 功能。而高级容器有更多的功能：支持不同的信息源头，可以访问文件资源，支持应用事件（Observer 模式）。

通常用户看到的就是 “高级容器”。 但 BeanFactory 也非常够用啦！

​	左边灰色区域的是 “低级容器”， 只负载加载 Bean，获取 Bean。容器其他的高级功能是没有的。例如上图画的 refresh 刷新 Bean 工厂所有配置、生命周期事件回调等。

> ​	WebApplicationContext 是专门为 Web 应用准备的，它允许从相对于 Web 根目录的路径中装载配置文件完成初始化工作。==从 WebApplicationContext 中可以获得ServletContext 的引用，整个 Web 应用上下文对象将作为属性放置到 ServletContext中，以便 Web 应用环境可以访问 Spring 应用上下文。==

## 5、常用的 ApplicationContext 容器

以下是三种较常见的 ApplicationContext 实现方式：

- 1、**ClassPathXmlApplicationContext** ：从 ClassPath(类路径) 的 XML 配置文件中读取上下文，并生成上下文定义。应用程序上下文从程序环境变量中取得。示例代码如下：

  ```
  ApplicationContext context = new ClassPathXmlApplicationContext(“bean.xml”);
  ```

- 2、**FileSystemXmlApplicationContext** ：由文件系统中的XML配置文件读取上下文。示例代码如下：

  ```
  ApplicationContext context = new FileSystemXmlApplicationContext(“bean.xml”);
  ```

- 3、**XmlWebApplicationContext** ：由 Web 应用的XML文件读取上下文。例如我们在 Spring MVC 使用的情况。

当然，目前我们更多的是使用 Spring Boot 为主，所以使用的是第四种 ApplicationContext 容器，**ConfigServletWebServerApplicationContext** 。

## 6、Spring IoC 的实现机制

### 1.BeanDefinitionRegistry 注册表

​	Spring 配置文件中每一个节点元素在 Spring 容器里都通过一个 BeanDefinition 对象表示， 它描述了 Bean 的配置信息。而 BeanDefinitionRegistry 接口提供了向容器手工注册 BeanDefinition 对象的方法。

### 2.BeanFactory 顶层接口

​	位于类结构树的顶端 ，它最主要的方法就是 getBean(String beanName)，该方法从容器中 返回特定名称的 Bean，BeanFactory 的功能通过其他的接口得到不断扩展；

### 3.ListableBeanFactory

​	该接口定义了访问容器中 Bean 基本信息的若干方法，如查看 Bean 的个数、获取某一类型 Bean 的配置名、查看容器中是否包括某一 Bean 等方法;

### 4.HierarchicalBeanFactory 父子级联

​	父子级联 IoC 容器的接口，子容器可以通过接口方法访问父容器; 通过 HierarchicalBeanFactory 接口， Spring 的 IoC 容器可以建立父子层级关联的容器体系，子 容器可以访问父容器中的 Bean，但父容器不能访问子容器的 Bean。Spring 使用父子容器实 现了很多功能，比如在 Spring MVC 中，展现层 Bean 位于一个子容器中，而业务层和持久 层的 Bean 位于父容器中。这样，展现层 Bean 就可以引用业务层和持久层的 Bean，而业务 层和持久层的 Bean 则看不到展现层的 Bean。

### 5.ConfigurableBeanFactory

​	是一个重要的接口，增强了 IoC 容器的可定制性，它定义了设置类装载器、属性编辑器、容 器初始化后置处理器等方法;

### 6.AutowireCapableBeanFactory 自动装配

​	定义了将容器中的 Bean 按某种规则(如按名字匹配、按类型匹配等)进行自动装配的方法; 

### 7.SingletonBeanRegistry 运行期间注册单例 Bean

​	定义了允许在运行期间向容器注册单实例 Bean 的方法;**对于单实例( singleton)的 Bean 来说，BeanFactory 会缓存 Bean 实例，所以第二次使用 getBean() 获取 Bean 时将直接从 IoC 容器的缓存中获取 Bean 实例。**Spring 在 DefaultSingletonBeanRegistry 类中提供了一 个用于缓存单实例 Bean 的缓存器，它是一个用 HashMap 实现的缓存器，单实例的 Bean 以 beanName 为键保存在这个 HashMap 中。

### 8.依赖日志框框

​	在初始化 BeanFactory 时，必须为其提供一种日志框架，比如使用 Log4J， 即在类路径下提供 Log4J 配置文件，这样启动 Spring 容器才不会报错。

简单来说，Spring 中的 IoC 的实现原理，就是**工厂模式**加**反射机制**。代码如下：

```Java
interface Fruit {
     public abstract void eat();     
}
class Apple implements Fruit {

    public void eat(){
        System.out.println("Apple");
    }    
}
class Orange implements Fruit {
    public void eat(){
        System.out.println("Orange");
    }
}

class Factory {
    public static Fruit getInstance(String className) {
        Fruit f = null;
        try {
            f = (Fruit) Class.forName(className).newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return f;
    }
}
class Client {
    public static void main(String[] args) {
        Fruit f = Factory.getInstance("io.github.dunwu.spring.Apple");
        if(f != null){
            f.eat();
        }
    }
}
```

- Fruit 接口，有 Apple 和 Orange 两个实现类。
- Factory 工厂，通过反射机制，创建 `className` 对应的 Fruit 对象。
- Client 通过 Factory 工厂，获得对应的 Fruit 对象。
- 实际情况下，Spring IoC 比这个复杂很多很多，例如单例 Bean 对象，Bean 的属性注入，相互依赖的 Bean 的处理，以及等等。

​	==IoC 启动过程其实就是ClassPathXmlApplicationContext 这个类，在启动时，都做了啥。==

下图是 ClassPathXmlApplicationContext 的构造过程，**实际就是 Spring IoC 的初始化过程**。

![img](/Users/jack/Desktop/md/images/4236553-db065eecf16176c3.png)

1. 用户构造 **ClassPathXmlApplicationContext**（简称 CPAC）
2. CPAC 首先访问了 “抽象高级容器” 中由final修饰的refresh 方法，这个方法是模板方法。所以要回调子类（低级容器）的 **refreshBeanFactory 方法，这个方法的作用是使用低级容器加载所有 BeanDefinition 和  Properties 到容器中。**
3. 低级容器加载成功后，高级容器开始处理一些回调，例如 Bean 后置处理器。回调 setBeanFactory 方法。或者注册监听器等，发布事件，实例化单例 Bean 等等功能，这些功能，随着 Spring 的不断升级，功能越来越多。

简单说就是：

1. ==低级容器 加载配置文件（从 XML，数据库，Applet），并解析成 BeanDefinition 到低级容器中。==
2. ==加载成功后，高级容器启动高级功能，例如接口回调，监听器，自动实例化单例，发布事件等等功能。==

当我们创建好容器，就会使用 getBean 方法，获取 Bean，而 getBean 的流程如下：

![img](/Users/jack/Desktop/md/images/4236553-da9a2f92e4dfa9db.png)



getBean 的操作都是在低级容器里操作的。**这里的递归是因为**：

​	假设 ： 当 Bean_A 依赖着 Bean_B，而这个 Bean_A 在加载的时候，**其配置的 ref = “Bean_B” 在解析的时候只是一个占位符，被放入了 Bean_A 的属性集合中，当调用 getBean 时，需要真正 Bean_B 注入到 Bean_A 内部时，就需要从容器中获取这个 Bean_B，因此产生了递归。**

​	为什么不是在加载的时候，就直接注入呢？因为加载的顺序不同，很可能 Bean_A 依赖的 Bean_B 还没有加载好，也就无法从容器中获取，你不能要求用户把 Bean 的加载顺序排列好，这是不人道的。

所以，Spring 将其分为了 2 个步骤：

1. **加载所有的 Bean 配置成 BeanDefinition 到容器中，如果 Bean 有依赖关系，则使用占位符暂时代替。**
2. **然后，在调用 getBean 的时候，进行真正的依赖注入，即如果碰到了属性是 ref 的（占位符），那么就从容器里获取这个 Bean，然后注入到实例中 —— 称之为依赖注入。**

可以看到，依赖注入实际上，只需要 “低级容器” 就可以实现。**这就是 IoC。**

​	**所以 ApplicationContext  refresh 方法里面的操作不只是 IoC，是高级容器的所有功能（包括 IoC），IoC 的功能在低级容器里就可以实现。**

#### IoC 在 Spring 里，只需要低级容器就可以实现，2 个步骤：

a. 加载配置文件，解析成 BeanDefinition 放在 Map 里。

b. 调用 getBean的时候，从BeanDefinition 所属的 Map 里，拿出  Class 对象进行实例化，同时，如果有依赖关系，将递归调用  getBean 方法 —— 完成依赖注入。

上面就是 Spring 低级容器（BeanFactory）的 IoC。

​	至于高级容器 ApplicationContext，他包含了低级容器的功能，**当他执行 refresh 模板方法的时候，将刷新整个容器的 Bean。**同时其作为高级容器，包含了太多的功能。一句话，他不仅仅是 IoC。他支持不同信息源头，支持 BeanFactory 工具类，支持层级容器，支持访问文件资源，支持事件发布通知，支持接口回调等等。

可以预见，随着 Spring 的不断发展，高级容器的功能会越来越多。

参照查看 [《面试问烂的 Spring IoC 过程》](http://www.iocoder.cn/Fight/Interview-poorly-asked-Spring-IOC-process-1/) 这篇文章。

## 7、Spring 框架中的事件

​	**Spring 事件为bean 与 bean之间传递消息，一个bean处理完了希望其余一个接着处理，这时我们就需要其余的一个bean监听当前bean所发送的事件。**

​	Spring 的 ApplicationContext 提供了支持事件和代码中监听器的功能。

​	我们可以创建 Bean 用来监听在 ApplicationContext 中发布的事件。如果一个 Bean 实现了 ApplicationListener 接口，当一个ApplicationEvent 被发布以后，Bean 会自动被通知。示例代码如下：

```Java
public class AllApplicationEventListener implements ApplicationListener<ApplicationEvent> {  
    
    @Override  
    public void onApplicationEvent(ApplicationEvent applicationEvent) {  
        // process event  
    }
}
```

### Spring 提供了以下五种标准的事件：

1. 上下文更新事件（ContextRefreshedEvent）：该事件会在ApplicationContext 被初始化或者更新时发布。也可以在调用ConfigurableApplicationContext 接口中的 `refresh()` 方法时被触发。
2. 上下文开始事件（ContextStartedEvent）：当容器调用ConfigurableApplicationContext 的 `start()` 方法开始/重新开始容器时触发该事件。
3. 上下文停止事件（ContextStoppedEvent）：当容器调用 ConfigurableApplicationContext 的 `stop()` 方法停止容器时触发该事件。
4. 上下文关闭事件（ContextClosedEvent）：当ApplicationContext 被关闭时触发该事件。容器被关闭时，其管理的所有单例 Bean 都被销毁。
5. 请求处理事件（RequestHandledEvent）：在 We b应用中，当一个HTTP 请求（request）结束触发该事件。

除了上面介绍的事件以外，还可以通过扩展 ApplicationEvent 类来开发**自定义**的事件。

① 示例自定义的事件的类，代码如下：

```Java
public class CustomApplicationEvent extends ApplicationEvent{  
    public CustomApplicationEvent(Object source, final String msg) {  
        super(source);
    }  
}
```

② 为了监听这个事件，还需要创建一个监听器。示例代码如下：

```Java
public class CustomEventListener implements ApplicationListener<CustomApplicationEvent> {
    @Override  
    public void onApplicationEvent(CustomApplicationEvent applicationEvent) {  
        // handle event  
    }
}
```

③ 之后通过 ApplicationContext 接口的 `publishEvent(Object event)` 方法，来发布自定义事件。示例代码如下：

```Java
// 创建 CustomApplicationEvent 事件
CustomApplicationEvent customEvent = new CustomApplicationEvent(applicationContext, "Test message");
// 发布事件
applicationContext.publishEvent(customEvent);
```

### Spring事件使用步骤如下:

1.先自定义事件：你的事件需要继承 ApplicationEvent

2.定义事件监听器: 需要实现 ApplicationListener

3.使用容器对事件进行发布

还有个例子可以参照：<https://www.cnblogs.com/huzi007/p/6215024.html>

# 二、Bean

- Bean 由 Spring IoC 容器实例化，配置，装配和管理。
- Bean 是基于用户提供给 IoC 容器的配置元数据 Bean Definition 创建。

## 1、Spring 的配置方式

单纯从 Spring Framework 提供的方式，一共有三种：

- 1、XML 配置文件。

  ​	Bean 所需的依赖项和服务在 XML 格式的配置文件中指定。这些配置文件通常包含许多 bean 定义和特定于应用程序的配置选项。它们通常以 bean 标签开头。例如：

  ```xml
  <bean id="studentBean" class="org.edureka.firstSpring.StudentBean">
      <property name="name" value="Edureka"></property>
  </bean>
  ```

- 2、注解配置。

  ​	可以通过在相关的类，方法或字段声明上使用注解，将 Bean 配置为组件类本身，而不是使用 XML 来描述 Bean 装配。默认情况下，Spring 容器中未打开注解装配。因此，需要在使用它之前在 Spring 配置文件中启用它。例如：

  ```xml
  <beans>
  <context:annotation-config/>
  <!-- bean definitions go here
   	或者通过下面这种方式，指定一个包：
  	<context:component-scan base-package="cn.gacl.java"/>
  -->    
  </beans>
  ```

- 3、Java Config 配置。

  **Spring 的 Java 配置是通过使用 @Bean 和 @Configuration 来实现。**

  - `@Bean` 注解扮演与 `<bean />` 元素相同的角色。

  - `@Configuration` 类允许通过简单地调用同一个类中的其他 `@Bean` 方法来定义 Bean 间依赖关系。

  - 例如：

    ```Java
    @Configuration
    public class StudentConfig {    
        @Bean
        public StudentBean myStudent() {
            return new StudentBean();
        }
    }
    ```


目前主要使用 **Java Config** 配置为主。当然，三种配置方式是可以混合使用的。例如说：

- Dubbo 服务的配置，一般使用 XML 。

  - ```xml
    	<dubbo:protocol name="dubbo" port="20884"></dubbo:protocol>
        <dubbo:application name="lehuan-search-service"/>
        <dubbo:registry address="zookeeper://192.168.98.135:2181"/>
      	<!--相当于包扫描-->
        <dubbo:annotation package="com.lehuan.search.service.impl"/>
        <dubbo:provider delay="-1" timeout="10000"/>
    ```

- Spring MVC 请求的配置，一般使用 `@RequestMapping` 注解。

- Spring MVC 拦截器的配置，一般 Java Config 配置。

------

另外，现在已经是 Spring Boot 的天下，所以更加是 **Java Config** 配置为主。

## 2、Bean Scope作用域

Spring Bean 支持 5 种 Scope ，分别如下：

- Singleton - 每个 Spring IoC 容器仅有一个单 Bean 实例。**默认**

  > Spring IoC 容器中只会存在一个共享的 Bean 实例，无论有多少个 Bean 引用它，始终指向同一对象。该模式在多线程下是不安全的。Singleton 作用域是 Spring 中的缺省作用域，也可以显示的将 Bean 定义为 singleton 模式，配置为: 
  >
  > <bean id="userDao" class="com.ioc.UserDaoImpl" scope="singleton"/>

- Prototype - 每次通过 Spring 容器获取 prototype 定义的 bean 时，容器都将创建一个新的 Bean 实例，

- Request - 每一次 HTTP 请求都会产生一个新的 Bean 实例，并且该 Bean 仅在当前 HTTP 请求内有效。

- Session - 每一个的 Session 都会产生一个新的 Bean 实例，同时该 Bean 仅在当前 HTTP Session 内有效。

- Application - 每一个 Web Application 都会产生一个新的 Bean ，同时该 Bean 仅在当前 Web Application 内有效。

==仅当用户使用支持 Web 的 ApplicationContext 时，**最后三个才可用**。==

开发者是可以**自定义** Bean Scope ，具体可参见 [《Spring（10）—— Bean 作用范围（二）—— 自定义 Scope》](https://blog.csdn.net/elim168/article/details/75581670)

## 3、Beann的生命周期

Spring Bean 的**初始化**流程如下：

### 实例化 Bean 对象

1. 实例化一个 Bean，也就是我们常说的 new。Spring 容器根据配置中的 Bean Definition(定义)中**实例化** Bean 对象。

   > Bean Definition 可以通过 XML，Java 注解或 Java Config 代码提供。

### ***IOC*** 依赖注入

2. 按照 Spring 上下文对实例化的 Bean 进行配置，也就是 IOC 注入。 

### ***setBeanName*** 实现

3. 如果这个 Bean 已经实现了 BeanNameAware 接口，会调用它实现的 setBeanName(String name) 方法，此处传递的就是 Spring 配置文件中 Bean 的 id 值 

### ***BeanFactoryAware*** 实现 

4. 如果这个 Bean 已经实现了 BeanFactoryAware 接口，工厂通过传递自身的实例来调用 `setBeanFactory(BeanFactory beanFactory)` 方法。setBeanFactory(BeanFactory)传递的是 Spring 工厂自身(可以用这个方式来获取其它 Bean， 只需在 Spring 配置文件中配置一个普通的 Bean 就可以)。 

### ***ApplicationContextAware*** 实现 

5. 如果这个 Bean 已经实现了 ApplicationContextAware 接口，会调用 setApplicationContext(ApplicationContext)方法，传入 Spring 上下文(同样这个方式也 可以实现步骤 4 的内容，但比 4 更好，因为 ApplicationContext 是 BeanFactory 的子接 口，有更多的实现方法) 

### **postProcessBeforeInitialization** 接口实现***-***初始化预处理 

6. 如果这个 Bean 关联了 BeanPostProcessor 接口，将会调用 postProcessBeforeInitialization(Object obj, String s)方法，**BeanPostProcessor 经常被用 作是 Bean 内容的更改，并且由于这个是在 Bean 初始化结束时调用那个的方法，也可以被应 用于内存或缓存技术。** 

### ***init-method*** 

7. 如果 Bean 在 Spring 配置文件中配置了<bean /\>的 init-method 属性，会自动调用其配置的初始化方法。 

### ***postProcessAfterInitialization*** 

8. 如果这个 Bean 关联了 BeanPostProcessor 接口，将会调用 postProcessAfterInitialization(Object obj, String s)方法。 注:以上工作完成以后就可以应用这个 Bean 了，那这个 Bean 是一个 Singleton 的，所以一 般情况下我们调用同一个 id 的 Bean 会是在内容地址相同的实例，当然在 Spring 配置文件中 也可以配置非 Singleton。 

### ***Destroy*** 过期自动清理阶段

9. 当 Bean 不再需要时，会经过清理阶段，**如果 Bean 实现了 DisposableBean 这个接口，会调用其实现的 destroy()方法;** 

### ***destroy-method*** 自配置清理

10. 最后，如果这个 Bean 的 Spring 配置中配置了 <bean /\>的destroy-method 属性，会自动调用其配置的 销毁方法。 

11. bean 标签有两个重要的属性(init-method 和 destroy-method)。用它们你可以自己定制 初始化和注销方法。**它们也有相应的注解(@PostConstruct 和@PreDestroy)。** 

```xml
<bean id="" class="" init-method="初始化方法" destroy-method="销毁方法"> 
```

![image-20190331211349307](/Users/jack/Desktop/md/images/image-20190331211349307.png)

![æµç¨å¾](/Users/jack/Desktop/md/images/08.png)

## 4、内部Bean

**只有将 Bean 仅用作另一个 Bean 的属性时，才能将 Bean 声明为内部 Bean。**

- 为了定义 Bean，Spring 提供基于 XML 的配置元数据在 `<property>`或 `<constructor-arg>` 中提供了 `<bean>`元素的使用。
- **内部 Bean 总是匿名的，并且它们总是作为原型 Prototype 。**

例如，假设我们有一个 Student 类，其中引用了 Person 类。这里我们将只创建一个 Person 类实例并在 Student 中使用它。示例代码如下：

```Java
// Student.java
public class Student {
    private Person person;
    // ... Setters and Getters
}
// Person.java

public class Person {
    private String name;
    private String address;    
    // ... Setters and Getters
}
```

```xml
<!-- bean.xml -->

<bean id=“StudentBean" class="com.edureka.Student">
    <property name="person">
        <!--This is inner bean -->
        <bean class="com.edureka.Person">
            <property name="name" value=“Scott"></property>
            <property name="address" value=“Bangalore"></property>
        </bean>
    </property>
</bean>
```

​	可以理解为在一个bean标签中引用了另一个bean，并且这个内部bean是作为外部bean的属性存在的。

## 5、Spring 装配

​	当 Bean 在 Spring 容器中组合在一起时，它被称为**装配**或 **Bean 装配**。Spring 容器需要知道需要什么 Bean 以及容器应该如何使用依赖注入来将 Bean 绑定在一起，同时装配 Bean 。

> 装配，和上文提到的 DI 依赖注入，实际是一个东西。

### **自动装配有哪些方式？**

​	**Spring 容器能够自动装配 Bean 。也就是说，可以通过检查 BeanFactory 的内容让 Spring 自动解析 Bean 的协作者。**

#### 自动装配的不同模式：

- no - 这是默认设置，表示没有自动装配。应使用显式 Bean 引用进行装配。
- byName - 它根据 Bean 的名称注入对象依赖项。它匹配并装配其属性与 XML 文件中由相同名称定义的 Bean 。
- 【最常用】**byType** - 它根据类型注入对象依赖项。如果属性的类型与 XML 文件中的一个 Bean 类型匹配，则匹配并装配属性。
- 构造函数 - 它通过调用类的构造函数来注入依赖项。它有大量的参数。
- autodetect - 首先容器尝试通过构造函数使用 autowire 装配，如果不能，则尝试通过 byType 自动装配。

## 6、延迟加载

​	默认情况下，容器启动之后会将所有作用域为**单例**的 Bean 都创建好，但是有的业务场景我们并不需要它提前都创建好。此时，==我们可以在Bean 中设置 `lzay-init = "true"` 。==

- 这样，当容器启动之后，作用域为单例的 Bean ，就不在创建。
- 而是在获得该 Bean 时，才真正在创建加载。

## 7、单例 Bean 是否线程安全

Spring 框架并没有对[单例](http://howtodoinjava.com/2012/10/22/singleton-design-pattern-in-java/) Bean 进行任何多线程的封装处理。

- 关于单例 Bean 的[线程安全](http://howtodoinjava.com/2014/06/02/what-is-thread-safety/)和并发问题，需要开发者自行去搞定。
- 并且，单例的线程安全问题，也不是 Spring 应该去关心的。Spring 应该做的是，提供根据配置，创建单例 Bean 或多例 Bean 的功能。

​	**当然，但实际上，大部分的 Spring Bean 并没有可变的状态(比如Serview 类和 DAO 类)，所以在某种程度上说 Spring 的单例 Bean 是线程安全的。**

​	==如果你的 Bean 有多种状态的话，就需要自行保证线程安全。最浅显的解决办法，就是将多态 Bean 的作用域( Scope )由 Singleton 变更为 Prototype 。==

## 8、Spring解决循环依赖

### 8.1. 什么是循环依赖

​	循环依赖，其实就是循环引用，就是两个或者两个以上的 bean 互相引用对方，最终形成一个闭环，如 A 依赖 B，B 依赖 C，C 依赖 A。如下图所示：

![循环依赖](/Users/jack/Desktop/md/images/20170912082357749.jpeg)

​	循环依赖，其实就是一个**死循环**的过程，在初始化 A 的时候发现引用了 B，这时就会去初始化 B，然后又发现 B 引用 C，跑去初始化 C，初始化 C 的时候发现引用了 A，则又会去初始化 A，依次循环永不退出，除非有**终结条件**。

Spring 循环依赖的**场景**有两种：

1. 构造器的循环依赖。

   ```Java
   @Service
   public class A {  
       public A(B b) {  }
   }
   
   @Service
   public class B {  
       public B(C c) {  
       }
   }
   
   @Service
   public class C {  
       public C(A a) {  }
   }
   ```

   结果：项目启动失败，发现了一个cycle

2. field 属性的循环依赖。

   1. scope为单例的时候：

      ```Java
      @Service
      public class A1 {  
          @Autowired  
          private B1 b1;
      }
      
      @Service
      public class B1 {  
          @Autowired  
          public C1 c1;
      }
      
      @Service
      public class C1 {  
          @Autowired  public A1 a1;
      }
      ```

      结果：项目启动成功

   2. scope为prototype的时候

      ```Java
      @Service
      @Scope("prototype")
      public class A1 {  
          @Autowired  
          private B1 b1;
      }
      
      @Service
      @Scope("prototype")
      public class B1 {  
          @Autowired  
          public C1 c1;
      }
      
      @Service
      @Scope("prototype")
      public class C1 {  
          @Autowired  public A1 a1;
      }
      ```

      结果：项目启动失败，发现了一个cycle

      ![img](/Users/jack/Desktop/md/images/auto-orient.png)

​	**对于构造器的循环依赖，Spring 是无法解决的，只能抛出 BeanCurrentlyInCreationException 异常表示循环依赖，**所以下面我们分析的都是基于 field 属性的循环依赖。

​	Spring 只解决 scope 为 singleton 的循环依赖。对于scope 为 prototype 的 bean ，Spring 无法解决，直接抛出 BeanCurrentlyInCreationException 异常。

为什么 Spring 不处理 prototype bean 呢？其实如果理解 Spring 是如何解决 singleton bean 的循环依赖就明白了。这里先卖一个关子，我们先来关注 Spring 是如何解决 singleton bean 的循环依赖的。

### 8.2. 解决循环依赖

#### 2.1 getSingleton

​	我们先从加载 bean 最初始的方法 **AbstractBeanFactory** 的 `doGetBean(final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly)` 方法开始。

在 `doGetBean(...)` 方法中，首先会根据 `beanName` 从单例 bean 缓存中获取，**如果不为空则直接返回**。代码如下：

```Java
// AbstractBeanFactory.java

Object sharedInstance = getSingleton(beanName);
```

- 调用 `getSingleton(String beanName, boolean allowEarlyReference)` 方法，从单例缓存中获取。代码如下：

  ```Java
  // DefaultSingletonBeanRegistry.java
  
  @Nullable
  protected Object getSingleton(String beanName, boolean allowEarlyReference) {
      // 从单例缓冲中加载 bean
      Object singletonObject = this.singletonObjects.get(beanName);
      // 缓存中的 bean 为空，且当前 bean 正在创建
      if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
          // 加锁
          synchronized (this.singletonObjects) {
              // 从 earlySingletonObjects 获取
              singletonObject = this.earlySingletonObjects.get(beanName);
              // earlySingletonObjects 中没有，且允许提前创建
              if (singletonObject == null && allowEarlyReference) {
                  // 从 singletonFactories 中获取对应的 ObjectFactory
                  ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                  if (singletonFactory != null) {
                      // 获得 bean
                      singletonObject = singletonFactory.getObject();
                      // 添加 bean 到 earlySingletonObjects 中
                      this.earlySingletonObjects.put(beanName, singletonObject);
                      // 从 singletonFactories 中移除对应的 ObjectFactory
                      this.singletonFactories.remove(beanName);
                  }
              }
          }
      }
      return singletonObject;
  }
  ```

  - 这个方法主要是从三个缓存中获取，分别是：`singletonObjects`、`earlySingletonObjects`、`singletonFactories` 。三者定义如下：

    ```Java
    // DefaultSingletonBeanRegistry.java
    
    /**
     * Cache of singleton objects: bean name to bean instance.
     * 一级缓存，保存所有的singletonBean的实例
     * 存放的是单例 bean 的映射。
     * 对应关系为 bean name --> bean instance
     */
    private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
    
    /**
     * Cache of singleton factories: bean name to ObjectFactory.
     * 三级缓存，singletonBean的生产工厂
     * 存放的是【早期】的单例 bean 的映射。
     * 对应关系也是 bean name --> bean instance。
     *
     * 它与 {@link singletonObjects} 的区别区别在，于 earlySingletonObjects 中存放的 bean 不一定是完整的。
     *
     * 从 {@link getSingleton(String)} 方法中，中我们可以了解，bean 在创建过程中就已经加入到 earlySingletonObjects 中了，
     * 所以当在 bean 的创建过程中就可以通过 getBean() 方法获取。
     * 这个 Map 也是解决【循环依赖】的关键所在。
     **/
    private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
    
    /**
     * Cache of early singleton objects: bean name to bean instance.
     * 二级缓存，保存所有早期创建的Bean对象，这个Bean还没有完成依赖注入 
     * 存放的是 ObjectFactory 的映射，可以理解为创建单例 bean 的 factory 。
     * 对应关系是 bean name --> ObjectFactory
     */
    private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
    ```

    - `singletonObjects` ：单例对象的 Cache 。
    - `singletonFactories` ： 单例对象工厂的 Cache 。
    - `earlySingletonObjects` ：**提前曝光**的单例对象的 Cache 。

它们三，就是 Spring 解决 singleton bean 的关键因素所在，我称他们为**三级缓存**：

- 第一级为 `singletonObjects`
- 第二级为 `earlySingletonObjects`
- 第三级为 `singletonFactories`

这里，我们已经通过 `getSingleton(String beanName, boolean allowEarlyReference)` 方法，看到他们是如何配合的。详细分析该方法之前，提下其中的 `isSingletonCurrentlyInCreation(String beanName)` 方法和 `allowEarlyReference`变量：

- `isSingletonCurrentlyInCreation(String beanName)` 方法：判断当前 singleton bean 是否处于创建中。bean 处于创建中，也就是说 bean 在初始化但是没有完成初始化，有一个这样的过程其实和 Spring 解决 bean 循环依赖的理念相辅相成。**==因为 Spring 解决 singleton bean 的核心就在于提前曝光 bean==** 。
- `allowEarlyReference` 变量：从字面意思上面理解就是允许提前拿到引用。其实真正的意思是，是否允许从 `singletonFactories` 缓存中通过 `getObject()` 方法，拿到对象。**因为 singletonFactories 才是 Spring 解决 singleton bean 的诀窍所在**。

`getSingleton(String beanName, boolean allowEarlyReference)` 方法，整个过程如下：

- 首先，从一级缓存 `singletonObjects` 获取。

- 如果，没有且当前指定的 `beanName` 正在创建，就再从二级缓存 `earlySingletonObjects` 中获取。

- 如果，还是没有获取到且允许 `singletonFactories` 通过 `getObject()` 获取，则从三级缓存 `singletonFactories` 获取。如果获取到，则通过其 `getObject()` 方法，获取对象，并将其加入到二级缓存 `earlySingletonObjects` 中，并从三级缓存 `singletonFactories` 删除。代码如下：

  ```Java
  // DefaultSingletonBeanRegistry.java
  
  singletonObject = singletonFactory.getObject();
  this.earlySingletonObjects.put(beanName, singletonObject);
  this.singletonFactories.remove(beanName);
  ```

  - 这样，就从三级缓存**升级**到二级缓存了。
  - 所以，二级缓存存在的**意义**，就是缓存三级缓存中的 ObjectFactory 的 `getObject()` 方法的执行结果，提早曝光的**单例** Bean 对象。

#### 2.2 addSingletonFactory(二级缓存出处)

​	上面是从缓存中获取，但是缓存中的数据从哪里添加进来的呢？一直往下跟会发现在 AbstractAutowireCapableBeanFactory 的 `doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args)` 方法中，有这么一段代码：

```java
// AbstractAutowireCapableBeanFactory.java

boolean earlySingletonExposure = (mbd.isSingleton() // 单例模式
        && this.allowCircularReferences // 运行循环依赖
        && isSingletonCurrentlyInCreation(beanName)); // 当前单例 bean 是否正在被创建
if (earlySingletonExposure) {
    if (logger.isTraceEnabled()) {
        logger.trace("Eagerly caching bean '" + beanName +
                "' to allow for resolving potential circular references");
    }
    // 提前将创建的 bean 实例加入到 singletonFactories 中
    // 这里是为了后期避免循环依赖
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}
```

- **当一个 Bean 满足三个条件时，则调用addSingletonFactory(...)方法，将它添加到缓存中。**三个条件如下：

  - 单例
  - 运行提前暴露 bean
  - 当前 bean 正在创建中

- `addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory)` 方法，代码如下：

  ```java
  // DefaultSingletonBeanRegistry.java
  
  protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
  	Assert.notNull(singletonFactory, "Singleton factory must not be null");
  	synchronized (this.singletonObjects) {
  		if (!this.singletonObjects.containsKey(beanName)) {
  			this.singletonFactories.put(beanName, singletonFactory);
  			this.earlySingletonObjects.remove(beanName);
  			this.registeredSingletons.add(beanName);
  		}
  	}
  }
  ```

  - 从这段代码我们可以看出，**`singletonFactories` 这个三级缓存才是解决 Spring Bean 循环依赖的诀窍所在。**同时这段代码发生在 `createBeanInstance(...)` 方法之后，也就是说这个 bean 其实已经被创建出来了，**但是它还不是很完美（没有进行属性填充和初始化），但是对于其他依赖它的对象而言已经足够了（可以根据对象引用定位到堆中对象），能够被认出来了**。所以 Spring 在这个时候，选择将该对象提前曝光出来让大家认识认识。

#### 2.3 addSingleton(一级缓存出处)

​	介绍到这里我们发现三级缓存 `singletonFactories` 和 二级缓存 `earlySingletonObjects` 中的值都有出处了，那一级缓存在哪里设置的呢？在类 DefaultSingletonBeanRegistry 中，可以发现这个 `addSingleton(String beanName, Object singletonObject)` 方法，代码如下：

```java
// DefaultSingletonBeanRegistry.java

protected void addSingleton(String beanName, Object singletonObject) {
	synchronized (this.singletonObjects) {
		this.singletonObjects.put(beanName, singletonObject);
		this.singletonFactories.remove(beanName);
		this.earlySingletonObjects.remove(beanName);
		this.registeredSingletons.add(beanName);
	}
}
```

- 添加至一级缓存，同时从二级、三级缓存中删除。

- 这个方法在 `doGetBean(...)` 方法中，**处理不同 scope 时，如果是 singleton，则调用 `getSingleton(...)` 方法，如下图所示：**

  ![getSingleton](/Users/jack/Desktop/md/images/20170912091609918.jpeg)

- `getSingleton(String beanName, ObjectFactory<?> singletonFactory)` 方法，代码如下：

  ```Java
  // AbstractBeanFactory.java
  
  public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
      Assert.notNull(beanName, "Bean name must not be null");
      synchronized (this.singletonObjects) {
          Object singletonObject = this.singletonObjects.get(beanName);
          if (singletonObject == null) {
              //....
              try {
                  singletonObject = singletonFactory.getObject();
                  newSingleton = true;
              }
              //.....
              if (newSingleton) {
                  addSingleton(beanName, singletonObject);
              }
          }
          return singletonObject;
      }
  }
  ```

  - 注意，此处的 `getSingleton(String beanName, ObjectFactory<?> singletonFactory)` 方法，在 AbstractBeanFactory 类中实现，和 上面的2.1 getSingleton**不同**。

### 8.3. 小结

​	至此，Spring 关于 singleton bean 循环依赖已经分析完毕了。所以我们基本上可以确定 Spring 解决循环依赖的方案了：

- Spring 在创建 bean 的时候并不是等它完全完成，而是在创建过程中将创建中的 bean 的 ObjectFactory 提前曝光（即加入到 `singletonFactories` 缓存中）。
- 这样，一旦下一个 bean 创建的时候需要依赖 bean ，则直接使用 ObjectFactory 的 `getObject()` 方法来获取了，也就是上面的2.1 getSingleton 小结中的方法中的代码片段了。

​        到这里，关于 Spring 解决 bean 循环依赖就已经分析完毕了。最后来描述下就**上面那个循环依赖 Spring 解决的过程：**

- 首先 A 完成初始化第一步并将自己提前曝光出来（通过 ObjectFactory 将自己提前曝光），在初始化的时候，发现自己依赖对象 B，此时就会去尝试 get(B)，这个时候发现 B 还没有被创建出来
- 然后 B 就走创建流程，在 B 初始化的时候，同样发现自己依赖 C，C 也没有被创建出来
- 这个时候 C 又开始初始化进程，但是在初始化的过程中发现自己依赖 A，于是尝试 get(A)，这个时候由于 A 已经添加至缓存中（一般都是添加至三级缓存 `singletonFactories` ），通过 ObjectFactory 提前曝光，所以可以通过 `ObjectFactory的getObject()` 方法来拿到 A 对象，C 拿到 A 对象后顺利完成初始化，然后将自己添加到一级缓存中
- 回到 B ，B 也可以拿到 C 对象，完成初始化，A 可以顺利拿到 B 完成初始化。到这里整个链路就已经完成了初始化过程了

> 如下是《Spring 源码深度解析》P114 页的一张图，非常有助于理解。
>
> ![处理依赖循环](/Users/jack/Desktop/md/images/01.png)

![img](/Users/jack/Desktop/md/images/auto-orient-20190402104237476.png)

参照：<https://www.jianshu.com/p/8bb67ca11831>

# 三、注解

![image-20190416165220081](/Users/jack/Desktop/md/images/image-20190416165220081.png)

## 1、@Component, @Controller, @Repository, @Service 的区别

- `@Component` ：==它将 Java 类标记为 Bean 。它是任何 Spring 管理组件的**通用**构造型。==

- `@Controller` ：它将一个类标记为 Spring Web MVC **控制器**。

- `@Service` ：此注解是组件注解的特化。它不会对 `@Component` 注解提供任何其他行为。您可以在**服务层**类中使用 @Service 而不是 `@Component` ，因为它以更好的方式指定了意图。

  - @Service注解，其实做了两件事情：
    (1)、声明Zoo.java这个类是一个bean，这点很重要，因为Zoo.java是一个bean，**其他的类才可以使用@Autowired将Zoo作为一个成员变量自动注入。**
    (2)、Zoo.java在bean中的id是"zoo"，即类名且首字母小写。如果想用大写的话，可以用

    ```Java
    @Service("Zoo")
    @Scope("prototype")
    ```

- @Repository ：这个注解是具有类似用途和功能的 `@Component` 注解的特化。它为 **DAO** 提供了额外的好处。它将 DAO 导入 IoC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException 。对应数据访问层Bean。**@Repository(value="userDao")注解是告诉Spring，让Spring创建一个名字叫"userDao"的UserDaoImpl实例。**

- @Configuration  把一个类作为一个IoC容器，它的某个方法头上如果注册了@Bean，就会作为这个Spring容器中的Bean。

  > **@configruation和@component注解的作用都是把配置类交給spring容器管理，然后通过@bean注解的方法动态代理，生成一个被spring容器管理的实例，相当于\<bean id=""  class="url">\</bean>**

- ==@Bean  注解在方法上使用，声明当前方法的返回值是一个Bean。==

  > @Bean是一个方法级别上的注解，主要用在@Configuration注解的类里，也可以用在@Component注解的类里，添加的bean的id为方法名。

- @Scope注解 作用域

- @Lazy(true) 表示延迟初始化

- @Service    用于标注业务层组件、 

- @Controller用于标注控制层组件（如struts中的action）

- @Repository用于标注数据访问组件，即DAO组件。

- @Component泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。

- @Scope用于指定scope作用域的（用在类上）

- @PostConstruct用于指定初始化方法（用在方法上）

- @PreDestory用于指定销毁方法（用在方法上）

- @DependsOn：定义Bean初始化及销毁时的顺序

- @Primary：自动装配时当出现多个Bean候选者时，被注解为@Primary的Bean将作为首选者，否则将抛出异常

- @Autowired 默认按类型装配，如果我们想使用按名称装配，可以结合@Qualifier注解一起使用。如下：

- @Autowired @Qualifier("personDaoBean") 存在多个实例配合使用

- @Resource   **默认按名称装配，当找不到与名称匹配的bean才会按类型装配。**
  - 装配顺序：
    - (1)、@Resource后面没有任何内容，默认通过name属性去匹配bean，找不到再按type去匹配
    - (2)、指定了name或者type则根据指定的类型去匹配bean
    - (3)、指定了name和type则根据指定的name和type去匹配bean，任何一个不匹配都将报错

- @PostConstruct 初始化注解

- @PreDestroy 摧毁注解 默认 单例  启动就加载

- @Async异步方法调用

参照：[Spring系列之Spring常用注解总结](https://www.cnblogs.com/xiaoxi/p/5935009.html)

## 2、@RequestParam与@PathVariable的区别

 **@RequestParam与@PathVariable为Spring的注解，都可以用于在Controller层接收前端传递的数据，不过两者的应用场景不同。**

@PathVariable主要用于接收http://host:port/path/{参数值}数据。

@RequestParam主要用于接收http://host:port/path?参数名=参数值数据，这里后面也可以不跟参数值。

```Java
//@PathVariable用法
@RequestMapping(value = "/test/{id}",method = RequestMethod.DELETE)
public Result test(@PathVariable("id")String id) 
    
//@RequestParam用法,注意这里请求后面没有添加参数
@RequestMapping(value = "/test",method = RequestMethod.POST)
    public Result test(@RequestParam(value="id",required=false,defaultValue="0")String id) 
```

注意上面@RequestParam用法当中的参数。

> value表示接收数据的名称。
>
> required表示接收的参数值是否必须，默认为true，既默认参数必须不为空，当传递过来的参数可能为空的时候可以设置required=false。
>
> 此外还有一个参数defaultValue 表示如果此次参数未空则为其设置一个默认值。

# 四、AOP

## 1、什么是 AOP 

AOP(Aspect-Oriented Programming)，即**面向切面编程**, 它与 OOP( Object-Oriented Programming, 面向对象编程) 相辅相成， 提供了与 OOP 不同的抽象软件结构的视角。

- 在 OOP 中，以类( Class )作为基本单元
- 在 AOP 中，以**切面( Aspect )**作为基本单元。

## 2、什么是 Aspect 

Aspect 由 **PointCut** 和 **Advice** 组成。

- 它既包含了横切逻辑的定义，也包括了连接点的定义。
- Spring AOP 就是负责实施切面的框架，它将切面所定义的横切逻辑编织到切面所指定的连接点中。

AOP 的工作重心在于如何将增强编织目标对象的连接点上, 这里包含两个工作:

1. 如何通过 PointCut 和 Advice 定位到特定的 **JoinPoint** 上。
2. 如何在 Advice 中编写切面代码。

**可以简单地认为, 使用 @Aspect 注解的类就是切面**

![流程图](/Users/jack/Desktop/md/images/04.jpg)

## 3、AOP相关术语

### Joinpoint(连接点):

​	所谓连接点是指那些被拦截到的点。在spring中,这些点指的是方法,因为spring只支持方法类型的连接点。

### Pointcut(切入点):

​	所谓切入点是指我们要对哪些Joinpoint进行拦截的定义。

两者区别：

​	首先，Advice 通过 PointCut 查询需要被织入的 JoinPoint 。

​	然后，Advice 在查询到 JoinPoint 上执行逻辑。

### Advice(通知/增强):

​	**所谓通知是指拦截到Joinpoint之后所要做的事情就是通知**，通知分为前置通知,后置通知,异常通知,最终通知,环绕通知(切面要完成的功能)

- Before - 这些类型的 Advice 在 JoinPoint 方法之前执行，并使用 `@Before` 注解标记进行配置。
- After Returning - 这些类型的 Advice 在连接点方法正常执行后执行，并使用 `@AfterReturning` 注解标记进行配置。
- After Throwing - 这些类型的 Advice 仅在 JoinPoint 方法通过抛出异常退出并使用 `@AfterThrowing` 注解标记配置时执行。
- After Finally - 这些类型的 Advice 在连接点方法之后执行，无论方法退出是正常还是异常返回，并使用 `@After` 注解标记进行配置。
- Around - 这些类型的 Advice 在连接点之前和之后执行，并使用 `@Around` 注解标记进行配置。

### Introduction(引介):

​	引介是一种特殊的通知在不修改类代码的前提下, Introduction可以在运行期为类动态地添加一些方法或Field。

### Target(目标对象):

​	织入 Advice 的**目标对象**，代理的目标对象

- 因为 Spring AOP 使用运行时代理的方式来实现 Aspect ，因此 Advised Object 总是一个代理对象(Proxied Object) 。
- **注意, Advised Object 指的不是原来的对象，而是织入 Advice 后所产生的代理对象**。
- Advice + Target Object = Advised Object = Proxy 。

### Weaving(织入):

​	是指把增强应用到目标对象来创建新的代理对象的过程。Spring采用动态代理织入，而AspectJ采用编译期织入和类装在期织入

### Proxy（代理）:

​	一个类被AOP织入增强后，就产生一个结果代理类

### Aspect(切面): 

​	是切入点和通知（引介）的结合

## 4、AOP实现方式

实现 AOP 的技术，主要分为两大类：

- ① **静态代理** - 指使用 AOP 框架提供的命令进行编译，从而在编译阶段就可生成 AOP 代理类，因此也称为编译时增强。
  - 编译时编织（特殊编译器实现）
  - 类加载时编织（特殊的类加载器实现）。
- ② **动态代理** - 在运行时在内存中“临时”生成 AOP 动态代理类，因此也被称为**运行时增强**。目前 Spring 中使用了两种动态代理库：
  - JDK 动态代理
  - CGLIB

那么 Spring 什么时候使用 JDK 动态代理，什么时候使用 CGLIB 呢？

```Java
// From 《Spring 源码深度解析》P172
// Spring AOP 部分使用 JDK 动态代理或者 CGLIB 来为目标对象创建代理。（建议尽量使用 JDK 的动态代理）
// 如果被代理的目标对象实现了至少一个接口，则会使用 JDK 动态代理。所有该目标类型实现的接口都讲被代理。
// 若该目标对象没有实现任何接口，则创建一个 CGLIB 代理。
// 如果你希望强制使用 CGLIB 代理，（例如希望代理目标对象的所有方法，而不只是实现自接口的方法）那也可以。但是需要考虑以下两个方法：
//      1> 无法通知(advise) Final 方法，因为它们不能被覆盖。
//      2> 你需要将 CGLIB 二进制发型包放在 classpath 下面。
// 为什么 Spring 默认使用 JDK 的动态代理呢？笔者猜测原因如下：
//      1> 使用 JDK 原生支持，减少三方依赖
//      2> JDK8 开始后，JDK 代理的性能差距 CGLIB 的性能不会太多。可参见：https://www.cnblogs.com/haiq/p/4304615.html
```

或者，我们来换一个解答答案：

Spring AOP 中的动态代理主要有两种方式，

- JDK 动态代理

  ​        **JDK 动态代理通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口。JDK动态代理的核心是 InvocationHandler 接口和 Proxy 类。**

- CGLIB 动态代理

  ​        如果目标类没有实现接口，那么 Spring AOP 会选择使用 CGLIB 来动态代理目标类。当然，Spring 也支持配置，**强制**使用 CGLIB 动态代理。
  CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成某个类的子类，注意，CGLIB 是通过继承的方式做的动态代理，因此如果某个类被标记为 `final` ，那么它是无法使用 CGLIB 做动态代理的。

## 5、AspectJ

### AspectJ表达式:

\* 语法:execution(表达式)

​	execution(<访问修饰符>?<返回类型><方法名>(<参数>)<异常>)

\* execution(“* cn.itcast.spring3.demo1.dao.*(..)”)     ---只检索当前包

\* execution(“* cn.itcast.spring3.demo1.dao..*(..)”)        ---检索包及当前包的子包.

\* execution(* cn.itcast.dao.GenericDAO+.*(..))         ---检索GenericDAO及子类

### AspectJ增强:

@Before	 前置通知，相当于BeforeAdvice

@AfterReturning 	后置通知，相当于AfterReturningAdvice

@Around 	环绕通知(前后到通知)，相当于MethodInterceptor

@AfterThrowing	抛出通知，相当于ThrowAdvice

@After	 最终final通知，不管是否异常，该通知都会执行

@DeclareParents 	引介通知，相当于IntroductionInterceptor 

具体例子可参照：[彻底征服 Spring AOP 之 实战篇](https://segmentfault.com/a/1190000007469982)

# 五、Transaction

## 1、事务

​	事务就是对一系列的数据库操作（比如插入多条数据）进行统一的提交或回滚操作，如果插入成功，那么一起成功，如果中间有一条出现异常，那么回滚之前的所有操作。

这样可以防止出现脏数据，防止数据库数据出现问题。

## 2、事务的特性

指的是 **ACID** ，如下图所示：

![事务的特性](/Users/jack/Desktop/md/images/06-20190402181033382.png)

1. **原子性** Atomicity ：一个事务（transaction）中的所有操作，或者全部完成，或者全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被恢复（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。即，事务不可分割、不可约简。
2. **一致性** Consistency ：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设[约束](https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E5%AE%8C%E6%95%B4%E6%80%A7)、[触发器](https://zh.wikipedia.org/wiki/%E8%A7%A6%E5%8F%91%E5%99%A8_(%E6%95%B0%E6%8D%AE%E5%BA%93))、[级联回滚](https://zh.wikipedia.org/w/index.php?title=%E7%BA%A7%E8%81%94%E5%9B%9E%E6%BB%9A&action=edit&redlink=1)等。
3. **隔离性** Isolation ：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
4. **持久性** Durability ：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

## 3、列举 Spring 支持的事务管理类型

目前 Spring 提供两种类型的事务管理：

- **声明式**事务：基于AOP，通过使用注解或基于 XML 的配置事务，从而事务管理与业务代码分离。
- **编程式**事务：通过编码的方式实现事务管理，需要在代码中显式的调用事务的获得、提交、回滚，提供极大的灵活性，但维护起来非常困难。

### 3.1 声明式事务

​	==Spring的声明式事务管理建立在AOP基础上，其本质是在目标方法执行前进行拦截，在方法开始前创建一个事务，在执行完方法后根据执行情况**提交或回滚事务**==。声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不用侵入业务代码，只需要在配置文件中做相关的事物声明就可将业务规则应用到业务逻辑中。**和编程式事务相比，声明式事务唯一的不足是智能作用到方法级别，无法做到像编程式事务那样到代码块级别。** 

#### 声明式事务有四种方式：

a.基于TransactionInterceptor的声明式事务；

b.基于TransactionProxyFactoryBean的声明式事务；

c.基于\命名空间的声明式事务；

d.基于标注（@Transactional）的声明式事务。

##### a.基于TransactionInterceptor的声明式事务：

　　**TransactionInterceptor主要有两个属性，一个是transactionManager，用于指定一个事务管理器；另一个是transactionAttributes，通过键值对的方式指定相应方法的事物属性，其中键值可以使用通配符。**在config.xml配置文件中加入TransactionInterceptor配置：

```xml
...
    <bean id="transactionInterceptor"
          class="org.springframework.transaction.interceptor.TransactionInterceptor">
        <property name="transactionManager" ref="transactionManager"/>
        <property name="transactionAttributes">
            <props>
                <prop key="*">PROPAGATION_REQUIRED</prop>
            </props>
        </property>
    </bean>
    <bean id="userService"
          class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target">
            <bean class="com.xiaofan.test.UserService" />
        </property>
        <property name="interceptorNames">
            <list>
                <idref bean="transactionInterceptor"/>
            </list>
        </property>
    </bean>
...
```

```Java
public class UserService {
    @Resource
    UserDAO userDAO;
    public void addUser3(User user) {
        userDAO.insert(user);
        Integer i = 1;
        if (i.equals(0)) {
        }
    }
}
```

##### b.基于TransactionProxyFactoryBean的声明式事务

　　**以上基于TransactionInterceptor的方式每个服务bean都需要配置一个ProxyFactoryBean，这会导致配置文件冗长，为了缓解这个问题，Spring提供了基于TransactionProxyFactoryBean的声明式事务配置方式。**在config.xml配置文件中加入TransactionProxyFactoryBean配置：

```xml
...
    <bean id="userService" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
        <property name="target">
            <bean class="com.xiaofan.test.UserService" />
        </property>
        <property name="transactionManager" ref="transactionManager"/>
        <property name="transactionAttributes">
            <props>
                <prop key="insert*">PROPAGATION_REQUIRED</prop>
                <prop key="update*">PROPAGATION_REQUIRED</prop>
                <prop key="*">PROPAGATION_REQUIRED</prop>
            </props>
        </property>
    </bean>
...
```

UserService服务代码如上，不用变更。

##### c.基于命名空间的声明式事务

　　Spring 2.x引入了命名空间，加上命名空间的切点表达式支持，声明式事务变的更加强大，借助于切点表达式，可以不需要为每个业务类创建一个代理。为了使用动态代理，首先需要添加pom依赖：

```xml
...
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.7.4</version>
        </dependency>
...
```

config.xml文件添加如下配置：

```xml
...
   <bean id="userService" class="com.xiaofan.test.UserService">
    </bean>
    <tx:advice id="userAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>
    <aop:config>
        <!--匹配模式，匹配对应的方法，参照上面的AOP-->
        <aop:pointcut id="userPointcut" expression="execution (* com.xiaofan.test.*.*(..))"/>
        <aop:advisor advice-ref="userAdvice" pointcut-ref="userPointcut"/>
    </aop:config>
...
```

UserService服务代码如上，不用变更。

##### d.基于标注（@Transactional）的声明式事务

　　除了基于命名空间的事务配置方式外，Spring2.x还引入了基于注解的方式，**主要涉及@Transactional注解，它可以作用于接口、接口方法、类和类的方法上，当做用于类上时，该类的所有public方法都有该类型的事务属性，可被方法级事务覆盖。**在config.xml配置文件中加入注解识别配置：

```xml
...
    <tx:annotation-driven transaction-manager="transactionManager"/>
...
```

　==@Transactional注解应该被应用到public方法上，这是由AOP的本质决定的，如果应用在protected、private的方法上，事务将被忽略==。UserService服务代码如下：

```Java
public class UserService {
    @Resource
    UserDAO userDAO;
    @Transactional(propagation = Propagation.REQUIRED)
    public void addUser4(User user) {
        userDAO.insert(user);
        Integer i = 1;
        if (i.equals(0)) {

        }
    }
}
```

##### @Transactional注解的完整属性信息如下表：

![image-20190402182417942](/Users/jack/Desktop/md/images/image-20190402182417942.png)

​	**基于命名空间和基于注解的事务声明各有优缺点：基于命名空间的方式一个配置可以匹配多个方法，但配置较注解方式复杂；基于注解的方式需要在每个需要使用事务的方法或类上标注，但基于标注的方法学习成本更低。**

### 3.2 编程式事务

一般的使用：

Spring配置文件config.xml如下：

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
            http://www.springframework.org/schema/tx
                http://www.springframework.org/schema/tx/spring-tx-2.5.xsd
            http://www.springframework.org/schema/context
                http://www.springframework.org/schema/context/spring-context-3.0.xsd">
    <!-- 引入属性文件 -->
    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>classpath:config.properties</value>
            </list>
        </property>
    </bean>
    <!-- 配置数据源 -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName">
            <value>${driver}</value>
        </property>
        <property name="url">
            <value>${url}</value>
        </property>
        <property name="username">
            <value>${username}</value>
        </property>
        <property name="password">
            <value>${password}</value>
        </property>
    </bean>

    <!-- 自动扫描了所有的mapper配置文件对应的mapper接口文件 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.xiaofan.test" />
    </bean>
    <!-- 配置Mybatis的文件 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="mapperLocations" value="classpath:user_mapper.xml"/>
        <property name="configLocation" value="classpath:mybatis_config.xml" />
    </bean>
    <!-- 配置JDBC事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="userService" class="com.xiaofan.test.UserService">
    </bean>
</beans>
```

​	根据PlatformTransactionManager、TransactionDefinition和TransactionStatus三个接口，可以通过编程的方式来进行事务管理, TransactionDefinition实例用于定义一个事务，PlatformTransactionManager实例用语执行事务管理操作，TransactionStatus实例用于跟踪事务的状态。UserService服务中配置如下：

```Java
public class UserService {
    @Resource
    UserDAO userDAO;
    @Resource
    DataSource dataSource;
    @Resource
    PlatformTransactionManager transactionManager;
    public void addUser(User user) throws Exception {
        TransactionDefinition def = new DefaultTransactionDefinition();
        TransactionStatus status = transactionManager.getTransaction(def);
        try {
            // [1] 插入纪录
            userDAO.insert(user);
            // [2] 范例抛出异常
            Integer i = null;
            if (i.equals(0)) {

            }
            transactionManager.commit(status);
        } catch (Exception e) {
            transactionManager.rollback(status);
            throw e;
        }
        return;
    }
}
```

Spring测试代码如下：

```Java
@ContextConfiguration(locations = {"classpath:config.xml"})
@RunWith(SpringJUnit4ClassRunner.class)
public class Test extends AbstractJUnit4SpringContextTests{

    @Resource
    UserService userService;

    @org.junit.Test
    public void testAdd() {
        try {
            userService.addUser(new User(null, "LiLei", 25));
        } catch (Exception e) {
        }
    }
}
```

如果［2］处抛出异常，则事务执行回滚，如果［2］没有抛出异常，则提交执行纪录插入操作。

**另一种编程式事务管理**

​	以上这种事务管理方式容易理解，但事务管理代码散落在业务代码中，破坏了原有代码的条理性，且每个事务方法中都包含了启动事务、提交/回滚事务的功能，基于此，**Spring提供了模板方法模式（TransactionTemplate）。** 

在config.xml配置文件中加入TransactionTemplate bean配置：

```xml
...
    <bean id="transactionTemplate"  class="org.springframework.transaction.support.TransactionTemplate">
        <property name="transactionManager" ref="transactionManager"/>
    </bean>
...
```

TransactionTemplate 的execute()方法有一个TransactionCallback类型的参数，该接口中定义了一个doInTransaction()方法，可通过匿名内部累的方式实现TransactionCallBack接口，将业务代码写在doInTransaction()方法中，业务代码中不需要显示调用任何事物管理API，除了异常回滚外，也可以在业务代码的任意位置通过transactionStatus.setRollbackOnly();执行回滚操作。UserService服务代码变更为：

```Java
public class UserService {
    @Resource
    UserDAO userDAO;
    @Resource
    TransactionTemplate transactionTemplate;
    public void addUser(final User user) {
        transactionTemplate.execute(new TransactionCallback() {
            public Object doInTransaction(TransactionStatus transactionStatus) {
                userDAO.insert(user);
                // transactionStatus.setRollbackOnly();
                Integer i = null;
                if (i.equals(0)) {

                }
                return null;
            }
        });
    }
}
```

实际场景下，我们一般使用 Spring Boot + 注解的**声明式**事务。参照 [《Spring Boot 事务注解详解》](https://www.jianshu.com/p/cddeca2c9245) 。

另外，也推荐看看 [《Spring 事务管理 － 编程式事务、声明式事务》](https://blog.csdn.net/xktxoo/article/details/77919508) 一文。

## 4、事务管理接口

- **PlatformTransactionManager：** （平台）事务管理器
- **TransactionDefinition：** 事务定义信息(事务隔离级别、传播行为、超时、只读、回滚规则)
- **TransactionStatus：** 事务运行状态

**所谓事务管理，其实就是“按照给定的事务规则来执行提交或者回滚操作”。**

### 4.1 Spring 事务如何和不同的数据持久层框架做集成

​	Spring并不直接管理事务，而是提供了多种事务管理器 ，他们将事务管理的职责委托给Hibernate或者JTA等持久化机制所提供的相关平台框架的事务来实现。 Spring事务管理器的接口是： **org.springframework.transaction.PlatformTransactionManager** ，通过这个接口，Spring为各个平台如JDBC、Hibernate等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情了。

### 4.2 PlatformTransactionManager接口代码如下：

PlatformTransactionManager接口中定义了三个方法：

```Java
// PlatformTransactionManager.java

public interface PlatformTransactionManager {

    // 根据事务定义 TransactionDefinition ，获得 TransactionStatus 。根据指定的传播行为，返回当前活动的事务或创建一个新事务。 
    TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;
    // 根据情况，提交事务，使用事务目前的状态提交事务
    void commit(TransactionStatus status) throws TransactionException;
    // 根据情况，回滚事务，对执行的事务进行回滚
    void rollback(TransactionStatus status) throws TransactionException; 
}
```

- PlatformTransactionManager 是负责事务管理的接口，一共有三个接口方法，分别负责事务的获得、提交、回滚。

- getTransaction(TransactionDefinition definition)方法，根据事务定义 TransactionDefinition ，获得 TransactionStatus 。

  - 为什么不是创建事务呢？因为如果当前如果已经有事务，则不会进行创建，一般来说会跟当前线程进行绑定。如果不存在事务，则进行创建。
  - 为什么返回的是 TransactionStatus 对象？在 TransactionStatus 中，不仅仅包含事务属性，还包含事务的其它信息，例如是否只读、是否为新创建的事务等等。

- commit(TransactionStatus status)方法，根据 TransactionStatus 情况，提交事务。

  - 为什么根据 TransactionStatus 情况，进行提交？例如说，带@Transactional注解的的 A 方法，会调用

    @Transactional注解的的 B 方法。

    - 在 B 方法结束调用后，会执行 `PlatformTransactionManager#commit(TransactionStatus status)` 方法，此处事务**是不能**、**也不会**提交的。
    - 而是在 A 方法结束调用后，执行 `PlatformTransactionManager#commit(TransactionStatus status)` 方法，提交事务。

- rollback(TransactionStatus status)方法，根据 TransactionStatus 情况，回滚事务。

  - 为什么根据 TransactionStatus 情况，进行回滚？原因同 `commit(TransactionStatus status)` 方法。

**Spring中PlatformTransactionManager根据不同持久层框架所对应的接口实现类,几个比较常见的如下图所示**

![PlatformTransactionManager根据不同持久层框架所对应的接口实现](/Users/jack/Desktop/md/images/1637b21877cf626d.png)

`org.springframework.transaction.support.AbstractPlatformTransactionManager` ，基于 [模板方法模式](https://blog.csdn.net/carson_ho/article/details/54910518) ，实现事务整体逻辑的骨架，而抽象 `doCommit(DefaultTransactionStatus status)`、`doRollback(DefaultTransactionStatus status)` 等等方法，交由子类类来实现。

④ 最后，不同的数据持久层框架，会有其对应的 PlatformTransactionManager 实现类，如下图所示：![事务的特性](/Users/jack/Desktop/md/images/07.png)

- 所有的实现类，都基于 AbstractPlatformTransactionManager 这个骨架类。
- HibernateTransactionManager ，和 Hibernate5 的事务管理做集成。
- DataSourceTransactionManager ，和 JDBC 的事务管理做集成。所以，它也适用于 MyBatis、Spring JDBC 等等。
- JpaTransactionManager ，和 JPA 的事务管理做集成。

如下，是一个比较常见的 使用JDBC或者iBatis（就是Mybatis）进行数据持久化操作的XML 方式来配置的事务管理器，使用的是 DataSourceTransactionManager 。代码如下：

```XML
<!-- 事务管理器 -->
<bean id="transactionManager"
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <!-- 数据源 -->
    <property name="dataSource" ref="dataSource" />
</bean>
```

### 4.3 TransactionDefinition接口介绍

​	**事务管理器接口 PlatformTransactionManager 通过 getTransaction(TransactionDefinition definition) 方法来得到一个事务，这个方法里面的参数是 TransactionDefinition类 ，这个类就定义了一些基本的事务属性。**

#### 事务属性

​	事务属性可以理解成事务的一些基本配置，描述了事务策略如何应用到方法上。事务属性包含了5个方面。 ![äºå¡å±æ§](/Users/jack/Desktop/md/images/1637b43a47916b2d.png)

#### TransactionDefinition接口中的方法如下：

​	TransactionDefinition接口中定义了5个方法以及一些表示事务属性的常量比如隔离级别、传播行为等等的常量。

我下面只是列出了TransactionDefinition接口中的方法及常量隔离如下：

```Java
public interface TransactionDefinition {
    // 返回事务的传播行为
    int getPropagationBehavior(); 
    // 返回事务的隔离级别，事务管理器根据它来控制另外一个事务可以看到本事务内的哪些数据
    int getIsolationLevel(); 
    // 返回事务必须在多少秒内完成
    //返回事务的名字
    String getName()；
    int getTimeout();  
    // 返回是否优化为只读事务。
    boolean isReadOnly();
    /**
 * 【Spring 独有】使用后端数据库默认的隔离级别
 *
 * MySQL 默认采用的 REPEATABLE_READ隔离级别
 * Oracle 默认采用的 READ_COMMITTED隔离级别
 */
	int ISOLATION_DEFAULT = -1;

/**
 * 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读
 */
	int ISOLATION_READ_UNCOMMITTED = Connection.TRANSACTION_READ_UNCOMMITTED;
/**
 * 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生
 */
	int ISOLATION_READ_COMMITTED = Connection.TRANSACTION_READ_COMMITTED;
/**
 * 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
 */
	int ISOLATION_REPEATABLE_READ = Connection.TRANSACTION_REPEATABLE_READ;
/**
 * 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。
 * 但是这将严重影响程序的性能。通常情况下也不会用到该级别。
 */
	int ISOLATION_SERIALIZABLE = Connection.TRANSACTION_SERIALIZABLE;
    // ========== 支持当前事务的情况 ========== 
/**
 * 如果当前存在事务，则使用该事务。
 * 如果当前没有事务，则创建一个新的事务。
 */
	int PROPAGATION_REQUIRED = 0;
/**
 * 如果当前存在事务，则使用该事务。
 * 如果当前没有事务，则以非事务的方式继续运行。
 */
	int PROPAGATION_SUPPORTS = 1;
/**
 * 如果当前存在事务，则使用该事务。
 * 如果当前没有事务，则抛出异常。
 */
	int PROPAGATION_MANDATORY = 2;

// ========== 不支持当前事务的情况 ========== 
/**
 * 创建一个新的事务。
 * 如果当前存在事务，则把当前事务挂起。
 */
	int PROPAGATION_REQUIRES_NEW = 3;
/**
 * 以非事务方式运行。
 * 如果当前存在事务，则把当前事务挂起。
 */
	int PROPAGATION_NOT_SUPPORTED = 4;
/**
 * 以非事务方式运行。
 * 如果当前存在事务，则抛出异常。
 */
	int PROPAGATION_NEVER = 5;

// ========== 其他情况 ========== 
/**
 * 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行。
 * 如果当前没有事务，则等价于 {@link TransactionDefinition#PROPAGATION_REQUIRED}
 */
	int PROPAGATION_NESTED = 6;
} 
```

#### （1）事务隔离级别（定义了一个事务可能受其他并发事务影响的程度）：

> **并发事务带来的问题**

​	在典型的应用程序中，多个事务并发运行，经常会操作相同的数据来完成各自的任务（多个用户对统一数据进行操作）。并发虽然是必须的，但可能会导致一下的问题。

- **脏读（Dirty read）**: 当一个事务正在访问数据并且对数据进行了修改，而这种修改还没有提交到数据库中，这时另外一个事务也访问了这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据，那么另外一个事务读到的这个数据是“脏数据”，依据“脏数据”所做的操作可能是不正确的。
- **丢失修改（Lost to modify）**: 指在一个事务读取一个数据时，另外一个事务也访问了该数据，那么在第一个事务中修改了这个数据后，第二个事务也修改了这个数据。这样第一个事务内的修改结果就被丢失，因此称为丢失修改。
  - 例如：事务1读取某表中的数据A=20，事务2也读取A=20，事务1修改A=A-1，事务2也修改A=A-1，最终结果A=19，事务1的修改被丢失。

- **不可重复读（Unrepeatableread）**: 指在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改导致第一个事务两次读取的数据可能不太一样。这就发生了在一个事务内两次读到的数据是不一样的情况，因此称为不可重复读。
- **幻读（Phantom read）**: 幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。

##### 不可重复度和幻读区别：

==不可重复读的重点是修改，幻读的重点在于新增或者删除。==

例1（同样的条件, 你读取过的数据, 再次读取出来发现值不一样了 ）：事务1中的A先生读取自己的工资为 1000的操作还没完成，事务2中的B先生就修改了A的工资为2000，导 致A再读自己的工资时工资变为 2000；这就是不可重复读。

例2（同样的条件, 第1次和第2次读出来的记录数不一样 ）：假某工资单表中工资大于3000的有4人，事务1读取了所有工资大于3000的人，共查到4条记录，这时事务2 又插入了一条工资大于3000的记录，事务1再次读取时查到的记录就变为了5条，这样就导致了幻读。

> ##### **隔离级别**

##### TransactionDefinition 接口中定义了五个表示隔离级别的常量：

- **TransactionDefinition.ISOLATION_DEFAULT:** 使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别，Oracle 默认采用的 READ_COMMITTED隔离级别.
- **TransactionDefinition.ISOLATION_READ_UNCOMMITTED:** 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读
- **TransactionDefinition.ISOLATION_READ_COMMITTED:** 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生
- **TransactionDefinition.ISOLATION_REPEATABLE_READ:** 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
- **TransactionDefinition.ISOLATION_SERIALIZABLE:** 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

#### （2）事务传播行为（为了解决业务层方法之间互相调用的事务问题）：

​	**当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。**例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。在TransactionDefinition定义中包括了如下几个表示传播行为的常量：

##### 支持当前事务的情况：

- **TransactionDefinition.PROPAGATION_REQUIRED**： 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- **TransactionDefinition.PROPAGATION_SUPPORTS**： 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **TransactionDefinition.PROPAGATION_MANDATORY**： 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。

##### 不支持当前事务的情况：

- **TransactionDefinition.PROPAGATION_REQUIRES_NEW**： 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NOT_SUPPORTED**： 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NEVER**： 以非事务方式运行，如果当前存在事务，则抛出异常。
  其他情况：
- **TransactionDefinition.PROPAGATION_NESTED： 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。**

​	这里需要指出的是，前面的六种事务传播行为是 Spring 从 EJB 中引入的，他们共享相同的概念。而 **PROPAGATION_NESTED 是 Spring 所特有的。**==以 PROPAGATION_NESTED 启动的事务内嵌于外部事务中（如果存在外部事务的话），此时，内嵌事务并不是一个独立的事务，它依赖于外部事务的存在，只有通过外部的事务提交，才能引起内部事务的提交，嵌套的子事务不能单独提交==。

​	类似于 JDBC 中的保存点（SavePoint）的概念，其实嵌套的子事务就是保存点的一个应用，**一个事务中可以包括多个保存点，每一个嵌套子事务。另外，外部事务的回滚也会导致嵌套子事务的回滚。**

#### (3) 事务超时属性(一个事务允许执行的最长时间)

​	所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 TransactionDefinition 中以 int 的值来表示超时时间，其单位是秒。

#### (4) 事务只读属性（对事物资源是否执行只读操作）

​	**事务的只读属性是指，对事务性资源进行只读操作或者是读写操作。**所谓事务性资源就是指那些被事务管理的资源，比如数据源、 JMS 资源，以及自定义的事务性资源等等。==如果确定只对事务性资源进行只读操作，那么我们可以将事务标志为只读的，以提高事务处理的性能。==在 TransactionDefinition 中以 boolean 类型来表示该事务是否只读。

#### (5) 回滚规则（定义事务回滚规则）

​	**这些规则定义了哪些异常会导致事务回滚而哪些不会。**默认情况下，==事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚==（这一行为与EJB的回滚行为是一致的）。 
​	但是你可以声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚。同样，你还可以声明事务遇到特定的异常不回滚，即使这些异常是运行期异常。

### 4.4 TransactionStatus接口介绍

​	==TransactionStatus接口用来记录事务的状态 该接口定义了一组方法,用来获取或判断事务的相应状态信息。==

PlatformTransactionManager.getTransaction(…) 方法返回一个 TransactionStatus 对象。返回的TransactionStatus 对象可能代表一个新的或已经存在的事务（如果在当前调用堆栈有一个符合条件的事务）。

TransactionStatus接口接口内容如下：

```Java
public interface TransactionStatus{
    boolean isNewTransaction(); // 是否是新的事物
    boolean hasSavepoint(); // 是否有恢复点，在 {@link TransactionDefinition#PROPAGATION_NESTED} 传播级别使用。
    void setRollbackOnly();  // 设置为只回滚
    boolean isRollbackOnly(); // 是否为只回滚
    boolean isCompleted; // 是否已完成
} 
```

- 为什么没有事务对象呢？在 TransactionStatus 的实现类 DefaultTransactionStatus 中，有个 `Object transaction` 属性，表示事务对象。
- `#isNewTransaction()` 方法，表示是否是新创建的事务。作用主要参照上面的 `commit(TransactionStatus status)` 方法的解释。通过该方法，我们可以判断，当前事务是否当前方法所创建的，只有创建事务的方法，**才能且应该真正的提交事务**。

[可能是最漂亮的Spring事务管理详解](<https://blog.csdn.net/qq_34337272/article/details/80394121>)

















