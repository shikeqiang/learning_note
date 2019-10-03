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

> 最终是通过WebMvcConfigurationSupport实现的

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

#### 常用注解

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

## 自动装配

- 版本依赖 
  - Spring Framework 3.1 + 
  - Servlet 3.0 + 

- Servlet SPI
  - Servlet SPI ServletContainerInitializer ，参考 Servlet 3.0 规范 

- 配合 @HandlesTypes 

- Spring 适配 ：SpringServletContainerInitializer

### Spring SPI(自动回调) 

- 基础接口: WebApplicationInitializer
- 编程驱动: AbstractDispatcherServletInitializer
- 注解驱动: AbstractAnnotationConfigDispatcherServletInitializer

## 简化 Spring Web MVC

### 完全自动装配(三大组件)

- 自动装配 DispatcherServlet : DispatcherServletAutoConfiguration

  > 在DispatcherServletAutoConfiguration中有一个Bean会返回ServletRegistrationBean类型的方法，ServletRegistrationBean最顶层接口是ServletContextInitializer

- 替换 @EnableWebMvc : WebMvcAutoConfiguration

  > @EnableWebMvc注解中引入了DelegatingWebMvcConfiguration，这个类继承了WebMvcConfigurationSupport；当WebMvcConfigurationSupport不存在时，才会装配WebMvcAutoConfiguration(@ConditionalOnMissingBean(WebMvcConfigurationSupport.class))

- Servlet 容器 : ServletWebServerFactoryAutoConfiguration(构造配置)

### 自动配置顺序性

- 绝对顺序: @AutoConfigureOrder
- 相对顺序: @AutoConfigureAfter

### 装配条件

- Web 类型判断( ConditionalOnWebApplication )

WebApplicationType：Servlet 类型: WebApplicationType.SERVLET

- API 判断( @ConditionalOnClass )

  Servlet

  Spring Web MVC

  ​    DispatcherServlet
  ​    WebMvcConfigurer

- Bean 判断( @ConditionalOnMissingBean 、 @ConditionalOnBean )

  - 比如：WebMvcConfigurationSupport

### 外部化配置

#### Web MVC 配置：WebMvcProperties：

​	这个类有个外部化配置：@ConfigurationProperties(prefix = "spring.mvc")，在additional-spring-configuration-metadata.json中配置

​	WebMvcProperties：这个类有个外部化配置：@ConfigurationProperties(prefix = "spring.mvc")，在additional-spring-configuration-metadata.json中配置

#### 资源配置：ResourceProperties

# 五、Web MVC视图应用

## 核心要素

### 资源定位(模板来源 )

- 通用资源抽象
  - 文件资源: File
  - ClassPath资源: ClassLoader
  - 统一资源: URL
  - Web资源: ServletContext

- Spring 资源抽象:
  - Spring 资源: Resource

### 渲染上下文(变量来源 )

- 不同的实现
  - Context :Thyemeaf 渲染上下文
  - Model :Spring Web MVC 模型
  - Attribute :Servlet 上下文

### 模板引擎(模板渲染)

- ITemplateEngine 实现
  - TemplateEngine :Thymeleaf 原生实现
  - SpringTemplateEngine :Spring 实现
  - SpringWebFluxTemplateEngine :Spring WebFlux 实现

## 视图处理

### Spring Web MVC 视图组件

- ViewResolver :视图解析器
- View : 视图组件
- DispatcherServlet :总控

### Thymeleaf 整合 Spring Web MVC

- ViewResolver : Thymeleaf
- ViewResolver View : ThymeleafView
- ITemplateEngine : SpringTemplateEngine

### 交互流程

![image-20190919233057800](/Users/jack/Desktop/md/images/image-20190919233057800.png)

## 视图内容协商

![image-20190921163247889](/Users/jack/Desktop/md/images/image-20190921163247889.png)

### 核心组件

- 视图解析
  - ContentNegotiatingViewResolver(关联ViewResolver Bean列表 -> 关联ContentNegotiationManager Bean -> 解析最佳匹配View)
    - InternalResourceViewResolver
    - BeanNameViewResolver
    - ThymeleafViewResolver

- 配置策略
  - 配置 Bean: WebMvcConfigurer
  - 配置对象: ContentNegotiationConfigurer

- 策略管理
  - Bean: ContentNegotiationManager
  - FactoryBean : ContentNegotiationManagerFactoryBean

- 策略实现
  - ContentNegotiationStrategy
    - 固定 MediaType : FixedContentNegotiationStrategy
    - "Accept" 请求头: HeaderContentNegotiationStrategy
    - 请求参数: ParameterContentNegotiationStrategy
    - 路径扩展名: PathExtensionContentNegotiationStrategy

![image-20190920000030855](/Users/jack/Desktop/md/images/image-20190920000030855.png)

#### 交互图

![image-20190921163553578](/Users/jack/Desktop/md/images/image-20190921163553578.png)

### 示例:多视图处理器内容协商

- 视图处理器协商

  - ContentNegotiatingViewResolver

    - BeanNameViewResolver
    - InternalResourceViewResolver Content-Type : text/xml;charset=UTF-8
    - ThymeleafViewResolver Content-Type : text/html;charset=UTF-8

    > 在ContentNegotiatingViewResolver中会读取不同的ContentType做不同处理

- 目的
  - 理解 BeanNameViewResolver
  - 理解 HTTP Accept 请求头 与 View Content-Type 匹配 
  - 理解最佳 View 匹配规则
    - ViewResolver 优先规则
      - 自定义 InternalResourceViewResolver 
      - ThymeleafViewResolver
      - 默认 InternalResourceViewResolver 
    - MediaType 匹配规则
      - Accept 头策略 
      - 请求参数策略

> 四种请求头：
>
> ![image-20190921165358743](/Users/jack/Desktop/md/images/image-20190921165358743.png)

## 视图组件自动装配

### 自动装配 Bean 

#### 视图处理器 

- InternalResourceViewResolver 
- BeanNameViewResolver 
- ContentNegotiatingViewResolver 
- ViewResolverComposite 
- ThymeleafViewResolver ( Thymeleaf 可用) 

#### 内容协商 ：ContentNegotiationManager

#### 外部化配置 

- WebMvcProperties
- WebMvcProperties.Contentnegotiation
- WebMvcProperties.View

# 六、Web MVC REST 应用

## Web MVC REST 支持

### 定义

| 注解            | 说明                                         | Spring Framework 版本 |
| --------------- | -------------------------------------------- | --------------------- |
| @Controller     | 应用控制器注解声明，Spring 模式注解          | 2.5 +                 |
| @RestController | 等效于 @Controller + @ResponseBody，组合注解 | 4.0 +                 |

### 映射

| 注解            | 说明                                                         | Spring Framework  版本 |
| --------------- | ------------------------------------------------------------ | ---------------------- |
| @RequestMapping | 应用控制器映射注解声明                                       | 2.5 +                  |
| @GetMapping     | GET 方法映射，等效于 @RequestMapping(method = RequestMethod.GET) | 4.3 +                  |
| @PostMapping    | POST 方法映射，等效于 @RequestMapping(method = RequestMethod.POST) | 4.3 +                  |
| @PutMapping     | PUT 方法映射，等效于 @RequestMapping(method = RequestMethod.PUT) | 4.3 +                  |
| @DeleteMapping  | DELETE 方法映射，等效于 @RequestMapping(method = RequestMethod.DELETE) | 4.3 +                  |
| @GetMapping     | GET 方法映射，等效于 @RequestMapping(method = RequestMethod.GET) | 4.3 +                  |
| @PatchMapping   | PATCH 方法映射，等效于 @RequestMapping(method = RequestMethod.PATCH) | 4.3 +                  |

### 请求

![image-20190921204540060](/Users/jack/Desktop/md/images/image-20190921204540060.png)

### 响应

| 注解                                                         | 说明                           | Spring Framework 版本 |
| ------------------------------------------------------------ | ------------------------------ | --------------------- |
| @ResponseBody                                                | 响应主题注解声明               | 2.5 +                 |
| @ResponseEntity                                              | 响应内容(包括响应主体和响应头) | 3.0.2 +               |
| ![page3image56360256.png](/Users/jack/Library/Application Support/typora-user-images/page3image56360256.png) ResponseCookie | 响应 Cookie 内容               | 5.0 +                 |

### 拦截

| 注解                                                         | 说明                         | Spring Framework 版本 |
| ------------------------------------------------------------ | ---------------------------- | --------------------- |
| @RestControllerAdvice                                        | @RestController 注解切面通知 | 4.3 +                 |
| ![page3image56347376.png](/Users/jack/Library/Application Support/typora-user-images/page3image56347376.png) HandlerInterceptor | 处理方法拦截器               | 1.0                   |

### 跨域

| 注解                              | 说明             | Spring Framework 版本 |
| --------------------------------- | ---------------- | --------------------- |
| @CrossOrigin                      | 资源跨域声明注解 | 4.2 +                 |
| @CorsFilter                       | 资源跨域拦截器   | 4.2 +                 |
| @WebMvcConfigurer#addCorsMappings | 注册资源跨域信息 | 4.2 +                 |

## REST 内容协商

### 核心组件

![image-20190921205211007](/Users/jack/Desktop/md/images/image-20190921205211007.png)

### Spring Web MVC REST 处理流程

![image-20190921210623953](/Users/jack/Desktop/md/images/image-20190921210623953.png)

### 内容协商处理流程

![image-20190921221824895](/Users/jack/Desktop/md/images/image-20190921221824895.png)

> 在AbstractMessageConverterMethodProcessor#writeWithMessageConverters方法中，有两个个地方判断媒体类型是否为通配类型：
>
> ```java
> if (contentType != null && contentType.isConcrete()) {
>    mediaTypesToUse = Collections.singletonList(contentType);
> }
> 	...
> for (MediaType mediaType : mediaTypesToUse) {
> 			if (mediaType.isConcrete()) {
> 				selectedMediaType = mediaType;
> 				break;
> 			}
> 			else if (mediaType.equals(MediaType.ALL) || mediaType.equals(MEDIA_TYPE_APPLICATION)) {
> 				selectedMediaType = MediaType.APPLICATION_OCTET_STREAM;
> 				break;
> 			}
> 		}
> ```
>
> 具体即没有通配符，有通配符或者子通配符则为不具体。

#### 请求的媒体类型

经过 ContentNegotiationManager 的 `ContentNegotiationStrategy` 解析请求中的媒体类型，比如: Accept 请求头

- 如果成功解析，返回合法 MediaType 列表 
- 否则，返回单元素 */* 媒体类型列表 - MediaType.ALL

> ```java
> @Override
> public List<MediaType> resolveMediaTypes(NativeWebRequest request) throws HttpMediaTypeNotAcceptableException {
>    for (ContentNegotiationStrategy strategy : this.strategies) {//遍历多个策略	
>       List<MediaType> mediaTypes = strategy.resolveMediaTypes(request);
>       if (mediaTypes.equals(MEDIA_TYPE_ALL_LIST)) {//如果都符合，最终会跳出循环，输出MEDIA_TYPE_ALL_LIST
>          continue;
>       }
>       return mediaTypes;
>    }
>    return MEDIA_TYPE_ALL_LIST;
> }
> ```

#### 可生成的媒体类型

返回 @Controller HandlerMethod @RequestMapping.produces() 属性所指定的 MediaType 列表:

- 如果 @RequestMapping.produces() 存在，返回指定 MediaType 列表(在controller类中的mapping注解上设置具体的produces属性的值，最终会生成对应响应头的媒体类型，参照上面的**核心组件**) 
- 否则，返回已注册的 HttpMessageConverter 列表中支持的 MediaType 列表

#### @RequestMapping#consumes

用于 @Controller HandlerMethod 匹配，请求头媒体类型映射:

- 如果请求头 Content-Type 媒体类型兼容 @RequestMapping.consumes() 属性，执行该 HandlerMethod
- 否则 HandlerMethod 不会被调用

#### @RequestMapping#produces

用于获取可生成的 MediaType 列表：

- 如果该列表与请求的媒体类型兼容，执行第一个兼容 HttpMessageConverter 的实现，默认@RequestMapping#produces 内容到响应头 Content-Type
- 否则，抛出 HttpMediaTypeNotAcceptableException , HTTP Status Code : 415

### 扩展 REST 内容协商

#### 自定义 HttpMessageConverter

​	实现 Content-Type 为 text/properties 媒体类型的 HttpMessageConverter

**实现步骤**

- 实现 HttpMessageConverter - PropertiesHttpMessageConverter
- 配置 PropertiesHttpMessageConverter 到 WebMvcConfigurer#extendMessageConverters

> ```java
> @Configuration
> public class RestWebMvcConfigurer implements WebMvcConfigurer {
> // 自定义converter
>     public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
> // 如果是直接添加(add)的话，则会添加到 converters 的末尾，不建议这样子，因为这样子可能会先加载了前面的converter，然后被略过
> //        converters.add(new PropertiesHttpMessageConverter());
> //        converters.set(0, new PropertiesHttpMessageConverter()); // 添加到集合首位
>     }
> }
> ```
>
> ```java 
> public class PropertiesHttpMessageConverter extends AbstractGenericHttpMessageConverter<Properties> {
> 
>     public PropertiesHttpMessageConverter() {
>         // 设置支持的 MediaType
>         super(new MediaType("text", "properties"));
>     }
> 
>     @Override
>     protected void writeInternal(Properties properties, Type type, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
>         // Properties -> String
>         // OutputStream -> Writer
>         HttpHeaders httpHeaders = outputMessage.getHeaders();
>         MediaType mediaType = httpHeaders.getContentType();
>         // 获取字符编码
>         Charset charset = mediaType.getCharset();
>         // 当 charset 不存在时，使用 UTF-8
>         charset = charset == null ? Charset.forName("UTF-8") : charset;
>         // 字节输出流
>         OutputStream outputStream = outputMessage.getBody();
>         // 字符输出流
>         Writer writer = new OutputStreamWriter(outputStream, charset);
>         // Properties 写入到字符输出流
>         properties.store(writer,"From PropertiesHttpMessageConverter");
>     }
> 
>     @Override
>     protected Properties readInternal(Class<? extends Properties> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
> 
>         // 字符流 -> 字符编码
>         // 从 请求头 Content-Type 解析编码
>         HttpHeaders httpHeaders = inputMessage.getHeaders();
>         MediaType mediaType = httpHeaders.getContentType();
>         // 获取字符编码
>         Charset charset = mediaType.getCharset();
>         // 当 charset 不存在时，使用 UTF-8
>         charset = charset == null ? Charset.forName("UTF-8") : charset;
> 
>         // 字节流
>         InputStream inputStream = inputMessage.getBody();
>         InputStreamReader reader = new InputStreamReader(inputStream, charset);
>         Properties properties = new Properties();
>         // 加载字符流成为 Properties 对象
>         properties.load(reader);
>         return properties;
>     }
> 
>     @Override
>     public Properties read(Type type, Class<?> contextClass, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
>         return readInternal(null, inputMessage);
>     }
> }
> ```

#### 自定义 HandlerMethodArgumentResolver

**需求**

- 不依赖 @RequestBody ， 实现 Properties 格式请求内容，解析为 Properties 对象的方法参数 
- 复用 PropertiesHttpMessageConverter

**实现步骤**

- 实现 HandlerMethodArgumentResolver - PropertiesHandlerMethodArgumentResolver

- ~~配置 PropertiesHandlerMethodArgumentResolver 到 WebMvcConfigurer#addArgumentResolvers~~

  > 添加自定义 HandlerMethodArgumentResolver，优先级低于内建 HandlerMethodArgumentResolver

- RequestMappingHandlerAdapter#setArgumentResolvers

#### 自定义 HandlerMethodReturnValueHandler

**需求**

- 不依赖 @ResponseBody ，实现 Properties 类型方法返回值，转化为 Properties 格式内容响应内容 
- 复用 PropertiesHttpMessageConverter

**实现步骤**

- 实现 HandlerMethodReturnValueHandler - PropertiesHandlerMethodReturnValueHandler

- ~~配置 PropertiesHandlerMethodReturnValueHandler 到 WebMvcConfigurer#addReturnValueHandlers~~

  > 同上面的自定义 HandlerMethodArgumentResolver

- RequestMappingHandlerAdapter#setReturnValueHandlers

  > ```java 
  > @Override
  > public void handleReturnValue(Object returnValue, MethodParameter returnType,
  >                               ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {
  >     // 强制装换
  >     Properties properties = (Properties) returnValue;
  >     // 复用 PropertiesHttpMessageConverter
  >     PropertiesHttpMessageConverter converter = new PropertiesHttpMessageConverter();
  > 
  >     ServletWebRequest servletWebRequest = (ServletWebRequest) webRequest;
  >     // Servlet Request API
  >     HttpServletRequest request = servletWebRequest.getRequest();
  >     String contentType = request.getHeader("Content-Type");
  >     // 获取请求头 Content-Type 中的媒体类型
  >     MediaType mediaType = MediaType.parseMediaType(contentType);
  > 
  >     // 获取 Servlet Response 对象
  >     HttpServletResponse response = servletWebRequest.getResponse();
  >     HttpOutputMessage message = new ServletServerHttpResponse(response);
  >     // 通过 PropertiesHttpMessageConverter 输出
  >     converter.write(properties, mediaType, message);
  >     // 告知 Spring Web MVC 当前请求已经处理完毕，否则会报错
  >     mavContainer.setRequestHandled(true);
  > }
  > ```

> 自定义HandlerMethodArgumentResolver和HandlerMethodReturnValueHandler顺序提前的实现
>
> ```java
> @Autowired
> private RequestMappingHandlerAdapter requestMappingHandlerAdapter;  // 改变自定义的HandlerMethodArgumentResolver的顺序
> 
> @PostConstruct
> public void init() {
>  // 获取当前 RequestMappingHandlerAdapter 所有的 Resolver 对象
>  List<HandlerMethodArgumentResolver> resolvers = requestMappingHandlerAdapter.getArgumentResolvers();
>  List<HandlerMethodArgumentResolver> newResolvers = new ArrayList<>(resolvers.size() + 1);
>  // 添加 PropertiesHandlerMethodArgumentResolver 到集合首位
>  newResolvers.add(new PropertiesHandlerMethodArgumentResolver());
>  // 添加 已注册的 Resolver 对象集合
>  newResolvers.addAll(resolvers);
>  // 重新设置 Resolver 对象集合
>  requestMappingHandlerAdapter.setArgumentResolvers(newResolvers);
> 
>  // 获取当前 HandlerMethodReturnValueHandler 所有的 Handler 对象
>  List<HandlerMethodReturnValueHandler> handlers = requestMappingHandlerAdapter.getReturnValueHandlers();
>  List<HandlerMethodReturnValueHandler> newHandlers = new ArrayList<>(handlers.size() + 1);
>  // 添加 PropertiesHandlerMethodReturnValueHandler 到集合首位
>  newHandlers.add(new PropertiesHandlerMethodReturnValueHandler());
>  // 添加 已注册的 Handler 对象集合
>  newHandlers.addAll(handlers);
>  // 重新设置 Handler 对象集合
>  requestMappingHandlerAdapter.setReturnValueHandlers(newHandlers);
> }
> ```
>
> 当controller的方法中，返回的不是pojo或者json无法解析的类型的时候，就需要外部的第三方扩展或者是Spring内部的扩展(如：HandlerMethodReturnValueHandler)。

## 跨域访问

- 注解驱动 @CrossOrigin

- 代码驱动 WebMvcConfigurer#addCorsMappings 

  > ```java
  > public void addCorsMappings(CorsRegistry registry) {
  >     registry.addMapping("/**").allowedOrigins("*");
  > }
  > ```

# 七、Servlet

## Servlet 核心 API

![image-20190922180016729](/Users/jack/Desktop/md/images/image-20190922180016729.png)

## Servlet 组件注册

![image-20190922175322726](/Users/jack/Desktop/md/images/image-20190922175322726.png)

```xml
 <?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-
app_2_5.xsd"
         metadata-complete="true" version="2.5">
    <context-param>
        <description>
Spring 配置文件路径参数，
该参数值将被 org.springframework.web.context.ContextLoaderListener 使用 </description>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath*:/META-INF/spring/spring-context.xml
        </param-value>
    </context-param>
    <listener>
        <description>
org.springframework.web.context.ContextLoaderListener 为可选申明Listener </description>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
</web-app>
```

## Spring Servlet Web

### Servlet 生命周期

- 初始化: init(ServletConfig)
- 服务: service(ServletRequest,ServletResponse)
- 销毁: destroy()

#### DispatcherServlet 初始化过程

![image-20190922180212448](/Users/jack/Desktop/md/images/image-20190922180212448.png)

> 上面基本都是抽象类，只有最后的DispatcherServlet才是实体类

### Filter 生命周期

- 初始化: init(FilterConfig)
- 服务: doFilter(ServletRequest,ServletResponse,FilterChain)
- 销毁: destroy()

### ServletContext 生命周期

- 初始化: contextInitialized(ServletContextEvent)
- 销毁: contextDestroyed(ServletContextEvent)

### Servlet 异步支持

- DeferredResult 支持(接口)
- Callable 支持
- CompletionStage 支持 

Spring Web MVC 异步 Servlet 实现原理

## Spring Boot Servlet Web

### Spring Boot 嵌入式 Servlet 容器限制

![image-20190922180544896](/Users/jack/Desktop/md/images/image-20190922180544896.png)

### Spring Boot Servlet 注册

#### 通过 RegistrationBean 注册

- ServletContextInitializer
  - RegistrationBean
    - ServletListenerRegistrationBean
      - @WebListener
    - FilterRegistrationBean
      - @WebFilter
    - ServletRegistrationBean
      - @WebServlet

@ServletComponentScan 扫描 package -> @Web* -> RegistrationBean Bean 定义 -> RegistrationBean Bean

#### 通过 @Bean 注册

#### 通过 @ServletComponentScan 注册

# 八、从 Reactive 到 WebFlux

## 1.理解 Reactive

- Reactive 是异步非阻塞编程
- Reactive 能够提升程序性能
- Reactive 解决传统编程模型遇到的困境

Reactive 框架:Java 9 Flow API 、RxJava(Reactive Extensions)、Reactor(Spring WebFlux Reactive)

## 2.Future链式问题

![image-20190929223226300](/Users/jack/Desktop/md/images/image-20190929223226300.png)

```java
/**
 * @Date: 2019-09-29 22:28
 * @Author: baichen
 * @Description 理解 Future 链式问题
 * 由于 Future 无法实现异步执行结果链式处理，尽管 FutureBlockingDataLoader 能够解决方法数据依赖
 * 以及顺序执行的问题，不过它将并行执行带回了阻塞(串行)执行,CompletableFuture 可以帮助提升 Future 的限制
 */
public class ChainDataLoader extends DataLoader {
    protected void doLoad() {
        // main -> submit -> ...
            // sub-thread : F1 -> F2 -> F3
        CompletableFuture
                .runAsync(super::loadConfigurations)
                .thenRun(super::loadUsers)
                .thenRun(super::loadOrders)
                .whenComplete((result, throwable) -> { // 完成时回调,都是同个线程
                    System.out.println("[线程 : "+Thread.currentThread().getName()+" ] 加载完成");
                })
                .join(); // 等待完成
    }
    public static void main(String[] args) {
        new ChainDataLoader().load();
    }
}
```

## 3.Reactive Programming 定义

​	不同机构或组织的不同定义：

### 1.The Reactive Manifesto

关键字:

- 响应的(Responsive) 
- 适应性强的(Resilient) 
- 弹性的(Elastic) 
- 消息驱动的(Message Driven)

侧重点:

面向 Reactive 系统、Reactive 系统原则

### 2.维基百科

关键字:

​	数据流(data streams )、传播变化( propagation of change) 

侧重点:

- 数据结构
  - 数组(arrays)
  - 事件发射器(event emitters) 

- 数据变化

技术连接:

- 数据流:Java 8 Stream
- 传播变化:Java Observable / Observer(也是通过回调实现)
- 事件:Java EventObject / EventListener

### 3.Spring Framwork

关键字:

​	变化响应(reacting to change )、非阻塞(non-blocking) 

侧重点:

- 响应通知

  操作完成(operations complete)、数据可用(data becomes available) 

技术连接:

- 非阻塞:Servlet 3.1 ReadListener / WriteListener
- 响应通知:Servlet 3.0 AsyncListener

### 4.ReactiveX

关键字：

​	观察者模式(Observer pattern ) 、数据/事件序列(Sequences of data and/or events )、序列操作符(Opeators) 屏蔽并发细节(abstracting away...)

侧重点:

  设计模式、数据结构、 数据操作、并发模型

技术连接:

- 观察者模式:Java Observable / Observer
- 数据/事件序列:Java 8 Stream
- 数据操作:Java 8 Stream
- 屏蔽并发细节(abstracting away...): Exectuor 、 Future 、 Runnable

### 5.Reactor

关键字:

​	观察者模式(Observer pattern )、 响应流模式(Reactive streams pattern ) 、迭代器模式(Iterator pattern) 、拉模式(pull-based) 和推模式(push-based)

侧重点:

​	设计模式 	、数据获取方式

技术连接:

- 观察者模式:Java Observable / Observer
- 响应流模式:Java 8 Stream
- 迭代器模式:Java Iterator

### 6.@andrestaltz

关键字:

​	异步(asynchronous ) 、数据流(data streams) 、并非新鲜事物(not anything new) 、过于理想化(idea on steroids)

侧重点:

  	并发模型、数据结构、技术本质

技术连接:

​	异步:Java Future

​	数据流:Java 8 Stream

## 4.Reactive Programming 特性

### 编程模型(Programming Models)

​		响应式编程和函数式编程





























































参照：[Spring Boot2.0深度实践](https://coding.imooc.com/class/252.html)