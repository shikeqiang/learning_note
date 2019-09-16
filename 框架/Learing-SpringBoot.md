# 一、Web Flux

## Servlet 规范：

请求接口：ServletRequest 或者 HttpServletRequest

响应接口：ServletResponse 或者 HttpServletResponse

## Spring 5.0 

​	重新定义了服务请求的响应接口:

请求接口：ServerRequest

响应接口：ServerResponse

即可支持 Servlet 规范，也可以支持自定义，比如 Netty （Web Server）

如下面例子：

定义 GET 请求，并且放回所有的用户对象 URI：/person/find/all

Flux 是 0 - N 个对象集合

Mono 是 0 - 1 个对象集合

==Reactive 中的 Flux 或者 Mono 是异步处理（非阻塞）==

集合对象基本上是同步处理（阻塞）

Flux 或者 Mono 都是 Publisher

```java 
//路由函数
@Bean
public RouterFunction<ServerResponse> personFindAll(UserRepository userRepository){
    Collection<User> users = userRepository.findAll();//获取用户对象
    return RouterFunctions.route(RequestPredicates.GET("/person/find/all"),
            request -> {
                Flux<User> userFlux = Flux.fromIterable(users);
                return ServerResponse.ok().body(userFlux,User.class);
            });
}
```

Web应用：https://www.jianshu.com/p/d703b1544346

# 二、装配

## 1.模式注解装配(单个功能)	

​	模式注解是一种用于声明在应用中扮演“组件”角色的注解。如 Spring Framework 中的 @Repository 标注在任何类上 ，用于扮演仓储角色的模式注解。

​	@Component 作为一种由 Spring 容器托管的通用模式组件，任何被 @Component 标准的组件均为组件扫描的候选对象。类似地，凡是被 @Component 元标注(meta-annotated)的注解，如 @Service ，当任何组件标注它时，也被视作组件扫 描的候选对象。**如下为serveice注解，其中包含了Component注解(派生性)。**

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {

   /**
    * The value may indicate a suggestion for a logical component name,
    * to be turned into a Spring bean in case of an autodetected component.
    * @return the suggested component name, if any (or empty String otherwise)
    */
   @AliasFor(annotation = Component.class)
   String value() default "";

}
```

例子：

![image-20190913181944751](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190913181944751.png)

## 2.@Enable 模块装配(多个功能)

​	Spring Framework 3.1 开始支持”@Enable 模块驱动“。所谓“模块”是指具备相同领域的功能组件集合， 组合所形成一个独立 的单元。比如 Web MVC 模块、AspectJ代理模块、Caching(缓存)模块、JMX(Java 管 理扩展)模块、Async(异步处 理)模块等。

### 实现方式

#### 注解驱动方式

```JAVA
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}

@Configuration
public class DelegatingWebMvcConfiguration extends
WebMvcConfigurationSupport {
	... 
}
```

#### 接口编程方式

```JAVA 
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(CachingConfigurationSelector.class)
public @interface EnableCaching {
	... 
}
//最终实现的是ImportSelector接口，其中有个String[] selectImports(AnnotationMetadata var1)
 public class CachingConfigurationSelector extends AdviceModeImportSelector<EnableCaching> {
    /**
    * {@inheritDoc}
    * @return {@link ProxyCachingConfiguration} or {@code
    AspectJCacheConfiguration} for
    * {@code PROXY} and {@code ASPECTJ} values of {@link
    EnableCaching#mode()}, respectively
    */
    public String[] selectImports(AdviceMode adviceMode) {
        switch (adviceMode) {
            case PROXY:
                return new String[] {
AutoProxyRegistrar.class.getName(),ProxyCachingConfiguration.class.getName() };
        case ASPECTJ:
            return new String[] {
                AnnotationConfigUtils.CACHE_ASPECT_CONFIGURATION_CLASS_NAME };
        default:
	} 
}
```

例子：

```java
/**
 * HelloWorld {@link ImportSelector} 实现，这里会返回HelloWorldConfiguration的相关内容
 * @author baichen
 */
public class HelloWorldImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{HelloWorldConfiguration.class.getName()};
    }
}

/**
 *  激活 HelloWorld 模块，@Enable模块装配注解
 * @author baichen
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
//@Import(HelloWorldConfiguration.class)
@Import(HelloWorldImportSelector.class) // 通过selector实现bean的调用
public @interface EnableHelloWorld {
}

/**
 * HelloWorld 配置
 * @author baichen
 */
public class HelloWorldConfiguration {
    @Bean
    public String helloWorld() { // 方法名即 Bean 名称
        return "Hello,World 2019";
    }
}
```

## 3.条件装配

| Spring 注解  | 场景说明       | 起始版本 |
| ------------ | -------------- | -------- |
| @Profile     | 配置化条件装配 | 3.1      |
| @Conditional | 编程条件装配   | 4.0      |

@Profile例子：

```java
@SpringBootApplication(scanBasePackages = "com.baichen.autoconfigure.service")
public class CalculateServiceBootstrap {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = new SpringApplicationBuilder(CalculateServiceBootstrap.class)
                .web(WebApplicationType.NONE)
                .profiles("Java7")  // 添加对应的条件
                .run(args);
        // CalculateService Bean 是否存在,不存在则报错，计算服务
        CalculateService calculateService = context.getBean(CalculateService.class);
        System.out.println("calculateService.sum(1...10) : " +
                calculateService.sum(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));
        // 关闭上下文
        context.close();
    }
}
```

@Conditional例子

```java

/**
 * Java 系统属性 条件判断
 * @author baichen
 */
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.TYPE, ElementType.METHOD })
@Documented
@Conditional(OnSystemPropertyCondition.class)
public @interface ConditionalOnSystemProperty {

    /**
     * Java 系统属性名称
     * @return
     */
    String name();
    /**
     * Java 系统属性值
     * @return
     */
    String value();
}

/**
 * 系统属性条件判断,实现了最顶层的Condition接口
 * @author baichen
 */
public class OnSystemPropertyCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 传入注解类名
        Map<String, Object> attributes = metadata.getAnnotationAttributes(ConditionalOnSystemProperty.class.getName());
        // 获取自定义的name和value对应的值
        String propertyName = String.valueOf(attributes.get("name"));
        String propertyValue = String.valueOf(attributes.get("value"));
        String javaPropertyValue = System.getProperty(propertyName);//根据name获取系统的用户名
        return propertyValue.equals(javaPropertyValue);
    }
}

public class ConditionalOnSystemPropertyBootstrap {

    @Bean
    @ConditionalOnSystemProperty(name = "user.name", value = "jack") //定义条件值，在OnSystemPropertyCondition类中实现了条件判断
    public String helloWorld() {
        return "Hello,World jack";
    }

    public static void main(String[] args) {
        ConfigurableApplicationContext context = new SpringApplicationBuilder(ConditionalOnSystemPropertyBootstrap.class)
                .web(WebApplicationType.NONE)
                .run(args);
        // 通过名称和类型获取 helloWorld Bean
        String helloWorld = context.getBean("helloWorld", String.class);
        System.out.println("helloWorld Bean : " + helloWorld);

        // 关闭上下文
        context.close();
    }
}
```

## 4.Spring Boot自动装配

​	在 Spring Boot 场景下，基于约定大于配置的原则，实现 Spring 组件自动装配的目的。其中使用了

### 底层装配技术

- Spring 模式注解装配
- Spring @Enable 模块装配
- Spring 条件装配装配Spring 工厂加载机制
  - 实现类: SpringFactoriesLoader
  - 配置资源: META-INF/spring.factories

### 自动装配举例

```properties
# 自动装配
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.baichen.autoconfigure.configuration.HelloWorldConfiguration
```

### 实现方法 

1. 激活自动装配 - @EnableAutoConfiguration
2. 实现自动装配 - XXXAutoConfiguration
3. 配置自动装配实现 - META-INF/spring.factories 

### 自定义自动装配 

 HelloWorldAutoConfiguration

条件判断: user.name == "jack" 

模式注解: @Configuration 

@Enable 模块: @EnableHelloWorld -> HelloWorldImportSelector -> HelloWorldConfiguration - > helloWorld 

代码参照：https://github.com/JDawnF/Learn_SpringBoot/tree/master/SB_SECOND/SB_AutoConfigure

# 三、SpringApplication

## 1.SpringApplication的使用

通过 SpringApplication API 调整：

```java
SpringApplication springApplication = new SpringApplication(DiveInSpringBootApplication.class);
springApplication.setBannerMode(Banner.Mode.CONSOLE);
springApplication.setWebApplicationType(WebApplicationType.NONE);
springApplication.setAdditionalProfiles("prod");
springApplication.setHeadless(true);
```

通过 SpringApplicationBuilder API 调整(可以实现Stream API)：

```java
 new SpringApplicationBuilder(DiveInSpringBootApplication.class)
    .bannerMode(Banner.Mode.CONSOLE)
    .web(WebApplicationType.NONE)
    .profiles("prod")
    .headless(true)
    .run(args);
```

## 2.SpringApplication准备阶段

​	即构造器阶段

### 配置 Spring Boot Bean 源

​	Java 配置 Class 或 XML 上下文配置文件集合，用于 Spring Boot Bean `DefinitionLoader` 读取 ，并且将配置源解析加载为Spring Bean 定义，数量:一个或多个以上。

#### Java 配置 Class

​	用于 Spring 注解驱动中 Java 配置类，大多数情况是 Spring 模式注解所标注的类，如 @Configuration 。

#### XML 上下文配置文件

​	用于 Spring 传统配置驱动中的 XML 文件。

### 推断 Web 应用类型

根据当前应用 ClassPath 中是否存在相关实现类来推断 Web 应用的类型，包括:

- Web Reactive: WebApplicationType.REACTIVE
- Web Servlet: WebApplicationType.SERVLET
- 非 Web: WebApplicationType.NONE

> 参考方法: org.springframework.boot.SpringApplication#deduceWebApplicationType
>
> ```java
>  private WebApplicationType deduceWebApplicationType() {
>     if (ClassUtils.isPresent(REACTIVE_WEB_ENVIRONMENT_CLASS, null)
>             && !ClassUtils.isPresent(MVC_WEB_ENVIRONMENT_CLASS, null)) {
>         return WebApplicationType.REACTIVE;
>     }
>     for (String className : WEB_ENVIRONMENT_CLASSES) {
>         if (!ClassUtils.isPresent(className, null)) {
>             return WebApplicationType.NONE;
> 				} 				
>     }
>     return WebApplicationType.SERVLET;
> }
> ```

### 推断引导类(Main Class)

​	根据 Main 线程执行堆栈判断实际的引导类

> 参考方法: org.springframework.boot.SpringApplication#deduceMainApplicationClass
>
> ```java
> private Class<?> deduceMainApplicationClass() {
>    try {
>       StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
>       for (StackTraceElement stackTraceElement : stackTrace) {
>          if ("main".equals(stackTraceElement.getMethodName())) {
>             return Class.forName(stackTraceElement.getClassName());
>          }
>       }
>    }
>    catch (ClassNotFoundException ex) {
>       // Swallow and continue
>    }
>    return null;
> }
> ```

### 加载应用上下文初始器 ( ApplicationContextInitializer )

​	利用 Spring 工厂加载机制，实例化 ApplicationContextInitializer 实现类，并排序对象集合。

> - 实现：
>
>   ```
>    private <T> Collection<T> getSpringFactoriesInstances(Class<T> type,
>         Class<?>[] parameterTypes, Object... args) {
>      ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
>      // Use names and ensure unique to protect against duplicates
>      Set<String> names = new LinkedHashSet<>(
>            SpringFactoriesLoader.loadFactoryNames(type, classLoader));
>      List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
>            classLoader, args, names);
>      AnnotationAwareOrderComparator.sort(instances);
>      return instances;
>   }
>   ```
>
> - 技术
>
>   - 实现类: org.springframework.core.io.support.SpringFactoriesLoader
>   - 配置资源: META-INF/spring.factories
>   - 排序: AnnotationAwareOrderComparator#sort
>
> > 这个工厂类会先加载sb的autoconfigure里面的spring.factories文件

### 加载应用事件监听器( ApplicationListener )

​	利用 Spring 工厂加载机制，实例化 ApplicationListener 实现类，并排序对象集合

> 可以通过添加注解：@Order(Ordered.HIGHEST_PRECEDENCE)；或者实现Ordered接口的getOrder(返回特定的顺序)来实现排序功能

## SpringApplication 运行阶段

### 加载 SpringApplication 运行监听器

( SpringApplicationRunListeners )

​	利用 Spring 工厂加载机制，读取 SpringApplicationRunListener 对象集合，并且封装到组合类SpringApplicationRunListeners

> 这里也是通过spring.factories配置文件中定义的类去实现的，配置的是EventPublishingRunListener类。

### 运行 SpringApplication 运行监听器

( SpringApplicationRunListeners )

SpringApplicationRunListener 监听多个运行状态方法:

| 监听方法                                         | 阶段说明                                                     | Spring Boot 起始版本 |
| ------------------------------------------------ | ------------------------------------------------------------ | -------------------- |
| starting()                                       | Spring 应用刚启动                                            | 1.0                  |
| environmentPrepared(ConfigurableEnvironment)     | ConfigurableEnvironment 准备妥当，允许将其调整               | 1.0                  |
| contextPrepared(ConfigurableApplicationContext)  | ConfigurableApplicationContext 准备妥当，允许将其调整        | 1.0                  |
| contextLoaded(ConfigurableApplicationContext)    | ConfigurableApplicationContext 已装载，但仍未启动            | 1.0                  |
| started(ConfigurableApplicationContext)          | ConfigurableApplicationContext 已启动，此时 Spring Bean 已初始化完成 | 2.0                  |
| running(ConfigurableApplicationContext)          | Spring 应用正在运行                                          | 2.0                  |
| failed(ConfigurableApplicationContext,Throwable) | Spring 应用运行失败                                          | 2.0                  |

### 监听 Spring Boot 事件 / Spring 事件

​	Spring Boot 通过 SpringApplicationRunListener 的实现类 EventPublishingRunListener 利用 Spring Framework 事件API ，广播 Spring Boot 事件。

#### Spring Framework 事件/监听器编程模型

- Spring 应用事件
  - 普通应用事件: ApplicationEvent
  - 应用上下文事件: ApplicationContextEvent

- Spring 应用监听器
  - 接口编程模型: ApplicationListener
  - 注解编程模型: @EventListener(需要在bean中实现)

- Spring 应用事广播器
  - 接口: ApplicationEventMulticaster
  - 实现类: SimpleApplicationEventMulticaster
    - 执行模式:同步或异步

#### EventPublishingRunListener 监听方法与 Spring Boot 事件对应关系

![image-20190915181433740](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190915181433740.png)

### 创建 Spring 应用上下文

( ConfigurableApplicationContext )

根据准备阶段的推断 Web 应用类型创建对应的 ConfigurableApplicationContext 实例:

- Web Reactive: AnnotationConfigReactiveWebServerApplicationContext
- Web Servlet: AnnotationConfigServletWebServerApplicationContext
- 非 Web: AnnotationConfigApplicationContext

### 创建 Environment

根据准备阶段的推断 Web 应用类型创建对应的 ConfigurableEnvironment 实例:

- Web Reactive: StandardEnvironment
- Web Servlet: StandardServletEnvironment
- 非 Web: StandardEnvironment

# 四、Web MVC核心

## Spring Web MVC 架构

### 基础架构:Servlet

![image-20190916221050152](/Users/jack/Desktop/md/images/image-20190916221050152.png)

- 特点
  - 请求/响应式(Request/Response)
  - 屏蔽网络通讯的细节

- API 特性
  - 面向 HTTP 协议
  - 完整生命周期

- 职责
- 处理请求
- 资源管理(数据库连接、消息连接、其他)
- 视图渲染

### Spring Web MVC 架构

![image-20190916221305325](/Users/jack/Desktop/md/images/image-20190916221305325.png)

### Web Mvc的实现过程

- 实现 Controller
- 配置 Web MVC 组件
- 部署 DispatcherServlet

![image-20190916205347241](/Users/jack/Desktop/md/images/image-20190916205347241.png)

![image-20190916205326201](/Users/jack/Desktop/md/images/image-20190916205326201.png)

![image-20190916205612421](/Users/jack/Desktop/md/images/image-20190916205612421.png)

### Web MVC 核心组件

![image-20190916221652655](/Users/jack/Desktop/md/images/image-20190916221652655.png)

#### 交互流程

![image-20190916221716160](/Users/jack/Desktop/md/images/image-20190916221716160.png)

> 首先请求先来到DispatcherServlet类中的doDispatch方法(入口方法)，然后到getHandler方法中，遍历handlerMappings;然后就来到getHandlerAdapter方法中，遍历handlerAdapters;然后来到resolveViewName方法中，遍历viewResolvers(viewName其实是controller中返回的字符串，可以配置前后缀)，最后返回具体的页面。

### Web MVC 注解驱动

注解配置: @Configuration ( Spring 范式注解 ) 

组件激活: @EnableWebMvc (Spring 模块装配) 

> 这是个激活注解,里面有import注解引入了DelegatingWebMvcConfiguration,在DelegatingWebMvcConfiguration中继承了WebMvcConfigurationSupport，这个类有通过@Bean注入一些web组件，比如handler、adapter等等(871行)

自定义组件 : WebMvcConfigurer (Spring Bean)

参考：

## 常用注解

注册模型属性: @ModelAttribute

读取请求头: @RequestHeader

读取 Cookie: @CookieValue

校验参数: @Valid 、 @Validated

注解处理: @ExceptionHandler

> ```java 
> @ExceptionHandler(Throwable.class) // 拦截异常
> public ResponseEntity<String> onException(Throwable throwable) {
>     return ResponseEntity.ok(throwable.getMessage());
> }
> ```

切面通知: @ControllerAdvice











































