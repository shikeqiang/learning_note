# 一、IoC

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

## 4、Spring中的IoC 容器

Spring 提供了两种IoC 容器，分别是 BeanFactory、ApplicationContext 。

**BeanFactory**

> BeanFactory 在 `spring-beans` 项目提供。

BeanFactory ，就像一个包含 Bean 集合的工厂类。它会在客户端要求时实例化 Bean 对象。

可以理解为就是个 HashMap，Key 是 BeanName，Value 是 Bean 实例。通常只提供注册（put），获取（get）这两个功能。我们可以称之为 **“低级容器”**。

**ApplicationContext**

> ApplicationContext 在 `spring-context` 项目提供。

**ApplicationContext可以称之为 “高级容器”,因为他 接口扩展了 BeanFactory 接口，它在 BeanFactory 基础上提供了一些额外的功能。内置如下功能**：

- MessageSource ：管理 message ，实现国际化等功能。
- ApplicationEventPublisher ：事件发布。
- ResourcePatternResolver ：多资源加载。
- EnvironmentCapable ：系统 Environment（profile + Properties）相关。
- Lifecycle ：管理生命周期。
- Closable ：关闭，释放资源
- InitializingBean：自定义初始化。
- BeanNameAware：设置 beanName 的 Aware 接口。

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

​	IoC 启动过程其实就是ClassPathXmlApplicationContext 这个类，在启动时，都做了啥。

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
2. 上下文开始事件（ContextStartedEvent）：当容器调用ConfigurableApplicationContext 的 `#start()` 方法开始/重新开始容器时触发该事件。
3. 上下文停止事件（ContextStoppedEvent）：当容器调用 ConfigurableApplicationContext 的 `#stop()` 方法停止容器时触发该事件。
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

## 2、Bean Scope作用域有哪些

Spring Bean 支持 5 种 Scope ，分别如下：

- Singleton - 每个 Spring IoC 容器仅有一个单 Bean 实例。**默认**
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









