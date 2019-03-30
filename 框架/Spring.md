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

​	看下面的隶属 ApplicationContext 粉红色的 “高级容器”，依赖着 “低级容器”，这里说的是依赖，不是继承,因为ApplicationContext继承了ListableBeanFactory，而ListableBeanFactory又继承了BeanFactory。他依赖着 “低级容器” 的 getBean 功能。而高级容器有更多的功能：支持不同的信息源头，可以访问文件资源，支持应用事件（Observer 模式）。

通常用户看到的就是 “高级容器”。 但 BeanFactory 也非常够用啦！

​	左边灰色区域的是 “低级容器”， 只负载加载 Bean，获取 Bean。容器其他的高级功能是没有的。例如上图画的 refresh 刷新 Bean 工厂所有配置、生命周期事件回调等。

好，解释了低级容器和高级容器，我们可以看看一个 IoC 启动过程是什么样子的。说白了，就是 ClassPathXmlApplicationContext 这个类，在启动时，都做了啥。（由于我这是 interface21 的代码，肯定和你的 Spring 4.x 系列不同）。

下图是 ClassPathXmlApplicationContext 的构造过程，**实际就是 Spring IoC 的初始化过程**。

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

















