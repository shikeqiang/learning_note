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

![image-20190913181944751](/Users/jack/Desktop/md/images/image-20190913181944751.png)

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

































































