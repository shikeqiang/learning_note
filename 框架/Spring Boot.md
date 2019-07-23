# 一、Spring Boot的核心功能

- 1、独立运行 Spring 项目

  Spring Boot 可以以 jar 包形式独立运行，运行一个 Spring Boot 项目只需通过 `java -jar xx.jar` 来运行。

- 2、内嵌 Servlet 容器

  Spring Boot 可以选择内嵌 Tomcat、Jetty 或者 Undertow，这样我们无须以 war 包形式部署项目。

  > 在 Spring Boot 未出来的时候，大多数 Web 项目，是打包成 war 包，部署到 Tomcat、Jetty 等容器。

- 3、提供 Starter 简化 Maven 配置

  Spring 提供了一系列的 starter pom 来简化 Maven 的依赖加载。例如，当你使用了 `spring-boot-starter-web` ，会自动加入如下依赖：

![`spring-boot-starter-web` ç pom æä"¶](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/01-6804336-8711964.png)

- 4、[自动配置 Spring Bean](https://www.jianshu.com/p/ddb6e32e3faf)

  Spring Boot 检测到特定类的存在，就会针对这个应用做一定的配置，进行自动配置 Bean ，这样会极大地减少我们要使用的配置。

  若在实际开发中我们需要配置Bean ，而 Spring Boot 没有提供支持，则可以自定义自动配置进行解决。

- 5、[准生产的应用监控](https://blog.csdn.net/wangshuang1631/article/details/72810412)

  Spring Boot 提供基于 HTTP、JMX、SSH 对运行时的项目进行监控。

- 6、无代码生成和 XML 配置

  Spring Boot 没有引入任何形式的代码生成，它是使用的 Spring 4.0 的条件 `@Condition` 注解以实现根据条件进行配置。同时使用了 Maven /Gradle 的**依赖传递解析机制**来实现 Spring 应用里面的自动配置。

# 二、统一引入 Spring Boot 版本

① 方式一：继承 `spring-boot-starter-parent` 项目。配置代码如下：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.1.RELEASE</version>
</parent>
```

② 方式二：导入 spring-boot-dependencies 项目依赖。配置代码如下：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.5.1.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

​	因为一般我们的项目中，都有项目自己的 Maven parent 项目，所以【方式一】显然会存在冲突。所以实际场景下，推荐使用【方式二】。另外，在使用 Spring Cloud 的时候，也可以使用这样的方式。

详细的，推荐阅读 [《Spring Boot 不使用默认的 parent，改用自己的项目的 parent》](https://blog.csdn.net/rainbow702/article/details/55046298) 文章。

# 三、打包和运行项目

​	通过引入 `spring-boot-maven-plugin` 插件，执行 `mvn clean package` 命令，将 Spring Boot 项目打成一个 Fat Jar 。后续，我们就可以直接使用 `java -jar` 运行。

关于 `spring-boot-maven-plugin` 插件，更多详细的可以看看 [《创建可执行 jar》](https://qbgbook.gitbooks.io/spring-boot-reference-guide-zh/II.%20Getting%20started/11.5.%20Creating%20an%20executable%20jar.html) 。

运行：

- 1、打包成 Fat Jar ，直接使用 `java -jar` 运行。目前主流的做法，推荐。
- 2、在 IDEA 或 Eclipse 中，直接运行应用的 Spring Boot 启动类的 `#main(String[] args)` 启动。适用于开发调试场景。
- 3、如果是 Web 项目，可以打包成 War 包，使用外部 Tomcat 或 Jetty 等容器。

### 热部署：

- 【推荐】`spring-boot-devtools` 插件。注意，这个工具需要配置 IDEA 的自动编译。

- Spring Loaded 插件。

  > Spring Boot 2.X 后，官方宣布不再支持 Spring Loaded 插件 的更新，所以基本可以无视它了。

- [JRebel](https://www.jianshu.com/p/bab43eaa4e14) 插件，需要付费。

关于如何使用 `spring-boot-devtools` 和 Spring Loaded 插件，可以看看 [《Spring Boot 学习笔记：Spring Boot Developer Tools 与热部署》](https://segmentfault.com/a/1190000014488100) 。

# 四、配置文件

两种格式：

- `.properties` 格式。示例如下：

  ```properties
  server.port = 9090
  ```

- `.yaml` 格式。示例如下：

  ```yaml
  server:
      port: 9090
  ```

- 与 Properties 文件相比，如果我们想要在配置文件中添加复杂的属性 YAML 文件就更加**结构化**。从上面的示例，我们可以看出 YAML 具有**分层**配置数据。
- 当然 YAML 在 Spring 会存在一个缺陷，`@PropertySource`注解不支持读取 YAML 配置文件，仅支持 Properties 配置文件。但是使用 [`@Value`](https://blog.csdn.net/lafengwnagzi/article/details/74178374) 注解，来读取 YAML 配置项。

> @PropertySource 例子：
>
> ```properties
> #customize.properties文件
> #超级管理员的用户名
> userName=administrator
> #超级管理员用户密码
> password=admin123
> ```
>
> ```Java
> //实现代码
> @Configuration
> @PropertySource(value = "file:${user.dir}/config/customize.properties", ignoreResourceNotFound = true)
> public class InitCustomizeUrlConfig {
> }
> ```
>
>
> 获取配置文件中的值通过Environment 具体使用方法
>
> ```Java
> @Autowired
> private Environment env;
>    env.getProperty("userName")
> ```
> 参照：<https://blog.csdn.net/u011659172/article/details/51274345>
>
> @Value 例子：
>
> ```yaml
> # application.yaml
> smartTalk: 
>   qa_url: https://nlsapi.aliyun.com/qas
>   qa_manage_url: https://nlsapi.aliyun.com/manage/qas
> ```
>
> ```Java
> // 获取配置内容的类
> @Component
> public class ApiClient {
>  
>     @Value("${smartTalk.qa_manage_url}")
>     private String   qa_manage_url;
>     @Value("${smartTalk.access_key_id}")
>     private String access_key_id;
>     @Value("${smartTalk.access_key_secret}")
>     private String access_key_secret;
>  
>     public String sendRequest(ApiRequest request) {
>         String url = qa_manage_url + "?action=" + request.getAction();
>         return HttpProxy.sendRequest(url, request.getBody(), access_key_id, access_key_secret);
>     }
>  
> }
> ```
>
> 但是要注意使用@Value的类如果被其他类作为对象引用，必须要使用注入的方式，而不能new
>
> 参照：https://blog.csdn.net/lafengwnagzi/article/details/74178374 

# 五、定义多套环境配置

## 1.使用Spring Boot Profiles

### 1.1 使用yml文件

首先,我们先创建一个名为 application.yml的属性文件,如下:

```yaml
server:
  port: 8080
my:
  name: demo
spring:
  profiles:
    active: dev
---
#development environment
spring:
  profiles: dev
server:
  port: 8160
my:
  name: ricky
---
#test environment
spring:
  profiles: test
server:
  port: 8180
my:
  name: test
---
#production environment
spring:
  profiles: prod
server:
  port: 8190
my:
  name: prod
```

​	==application.yml文件分为四部分,使用 --- 来作为分隔符，第一部分通用配置部分，表示三个环境都通用的属性， 后面三段分别为：开发，测试，生产，用spring.profiles指定了一个值(开发为dev，测试为test，生产为prod)，这个值表示该段配置应该用在哪个profile里面==。

​	如果是本地启动，在通用配置里面可以设置调用哪个环境的profil，也就是第一段的spring.profiles.active=XXX， **其中XXX是后面3段中spring.profiles对应的value,通过这个就可以控制本地启动调用哪个环境的配置文件**，例如:

```yaml
spring:
    profiles:
        active: dev
```

> 表示默认 加载的就是开发环境的配置，如果dev换成test，则会加载测试环境的属性，以此类推。
>
> **注意：如果spring.profiles.active没有指定值，那么只会使用没有指定spring.profiles文件的值，也就是只会加载通用的配置。**

#### 启动参数

​	**如果是部署到服务器的话,我们正常打成jar包，启动时通过 `--spring.profiles.active=xxx` 来控制加载哪个环境的配置**，完整命令如下:

```bash
java -jar xxx.jar --spring.profiles.active=test 表示使用测试环境的配置
java -jar xxx.jar --spring.profiles.active=prod 表示使用生产环境的配置
```

#### 使用多个yml配置文件进行配置属性文件

​	也可以使用多个yml来配置属性，将于环境无关的属性放置到application.yml文件里面；**通过与配置文件相同的命名规范，创建application-{profile}.yml文件 存放不同环境特有的配置，例如 application-test.yml 存放测试环境特有的配置属性，application-prod.yml 存放生产环境特有的配置属性。**

​	通过这种形式来配置多个环境的属性文件，**在application.yml文件里面spring.profiles.active=xxx来指定加载不同环境的配置,如果不指定，则默认只使用application.yml属性文件，不会加载其他的profiles的配置。**

### 1.2 使用properties文件

​	如果使用application.properties进行多个环境的配置，原理跟使用多个yml配置文件一致，创建application-{profile}.properties文件 存放不同环境特有的配置，将与环境无关的属性放置到application.properties文件里面，并在application.properties文件中通过spring.profiles.active=xxx 指定加载不同环境的配置。如果不指定，则默认加载application.properties的配置，不会加载带有profile的配置。

## 2.Maven Profile

​	可以通过Maven的profile特性来实现多环境配置打包。

```xml
<profiles>
        <!--开发环境-->
        <profile>
            <id>dev</id>
            <properties>
                <build.profile.id>dev</build.profile.id>
            </properties>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        <!--测试环境-->
        <profile>
            <id>test</id>
            <properties>
                <build.profile.id>test</build.profile.id>
            </properties>
        </profile>
        <!--生产环境-->
        <profile>
            <id>prod</id>
            <properties>
                <build.profile.id>prod</build.profile.id>
            </properties>
        </profile>
    </profiles>

    <build>
        <finalName>${project.artifactId}</finalName>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resources.${build.profile.id}</directory>
                <filtering>false</filtering>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <classifier>exec</classifier>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

​	通过执行 `mvn clean package -P ${profile}` 来指定使用哪个profile。

# 六、核心注解@SpringBootApplication

​	Spring Boot 的核心注解是@SpringBootApplication，源码如下：

```Java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Configuration
@EnableAutoConfiguration
@ComponentScan
public @interface SpringBootApplication {
    Class<?>[] exclude() default {};
    String[] excludeName() default {};
    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackages"
    )
    String[] scanBasePackages() default {};
    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackageClasses"
    )
    Class<?>[] scanBasePackageClasses() default {};
}
```

​	从源代码中得知 **@SpringBootApplication 被 @Configuration、@EnableAutoConfiguration、@ComponentScan 注解所修饰**，换言之 Springboot 提供了统一的注解来替代以上三个注解，简化程序的配置。下面解释一下各注解的功能。

## @Configuration

​	@Configuration 是一个类级注释，指定类是 **Bean 定义**的配置类。@Configuration 类通过 @bean 注解的public 公共方法声明bean。

​	通俗的讲 @Configuration 一般与 @Bean 注解配合使用，用 @Configuration 注解类等价与 XML 中配置 beans，用 @Bean 注解方法等价于 XML 中配置 bean。举例说明：

```xml
<!--xml配置，等价于下面的Java文件，注意在xml文件中用了ref标签-->
<beans>
    <bean id = "userService" class="com.user.UserService">
        <property name="userDAO" ref = "userDAO"></property>
    </bean>
    <bean id = "userDAO" class="com.user.UserDAO"></bean>
</beans>
```

```Java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class Config {
    @Bean
    public UserService getUserService(){
        UserService userService = new UserService();
        userService.setUserDAO(null);
        return userService;
    }
    @Bean
    public UserDAO getUserDAO(){
        return new UserDAO();
    }
}

public class UserService {
    private UserDAO userDAO;

    public UserDAO getUserDAO() {
        return userDAO;
    }

    public void setUserDAO(UserDAO userDAO) {
        this.userDAO = userDAO;
    }
}

public class UserDAO {
}
```

注意：

- 使用 public 修饰 @Bean 注解的方法；
- UserService、UserDAO 类无需声明为 @Component、@Service、@Repository、@Controller；

​	不用 @Autowired 注解直接注入 UserDAO的原因可能是： **@Configuration、@Bean 更注重配置方面的内容，比如我们在 @Bean 方法中可以设置 UserDAO 类对象的特殊信息，比如指定 transactionManager 等。**

## @EnableAutoConfiguration

​	启用 Spring 应用程序上下文的自动配置，试图猜测和配置您可能需要的bean。自动配置类通常采用基于你的 classpath 和已经定义的 beans 对象进行应用。

​	被 @EnableAutoConfiguration 注解的类所在的包有特定的意义，并且作为默认配置使用。例如，当扫描 @Entity类的时候就会使用到它。通常推荐将 @EnableAutoConfiguration 配置在 root 包下，这样所有的子包、类都可以被查找到。

​	Auto-configuration类是常规的 Spring 配置 Bean。它们是通过使用 SpringFactoriesLoader 机制（以 EnableAutoConfiguration 类路径为 key）定位的。通常 auto-configuration beans 是 @Conditional beans（在大多数情况下配合 @ConditionalOnClass 和 @ConditionalOnMissingBean 注解进行使用）。

打开自动配置的功能。如果我们想要关闭某个类的自动配置，可以设置注解的 `exclude` 或 `excludeName` 属性。

> ### SpringFactoriesLoader 机制： 
>
> ​	**SpringFactoriesLoader为Spring工厂加载器，该对象提供了loadFactoryNames方法，入参为factoryClass和classLoader即需要传入工厂类名称和对应的类加载器，方法会根据指定的classLoader，加载该类加器搜索路径下的指定文件，即spring.factories文件，传入的工厂类为接口，而文件中对应的类则是接口的实现类，或最终作为实现类。**
>
> ​	==SpringFactoriesLoader会查询包含 META-INF/spring.factories 文件的JAR。== 当找到spring.factories文件后，SpringFactoriesLoader将查询配置文件命名的属性。EnableAutoConfiguration的 key 值为org.springframework.boot.autoconfigure.EnableAutoConfiguration。根据此 key 对应的值进行 spring 配置。在 spring-boot-autoconfigure.jar文件中，包含一个 spring.factories 文件，部分内容如下：
>
> ![image-20190524093545944](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190524093545944.png)
>
> ```factories
> # Initializers
> org.springframework.context.ApplicationContextInitializer=\
> org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
> org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener
> 
> # Application Listeners
> org.springframework.context.ApplicationListener=\
> org.springframework.boot.autoconfigure.BackgroundPreinitializer
> ...
> ```
>
> ![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/1650e0e47481e59a.png)

## @ComponentScan

​	扫描指定包下的 Bean 们，为 @Configuration注解的类配置组件扫描指令，同时提供与 Spring XML 元素并行的支持。无论是 basePackageClasses() 或是 basePackages() （或其 alias 值）都可以定义指定的包进行扫描。如果指定的包没有被定义，则将从声明该注解的类所在的包进行扫描。

​	注意， 元素有一个 annotation-config 属性（详情：http://www.cnblogs.com/exe19/p/5391712.html），但是 @ComponentScan 没有。这是因为在使用 @ComponentScan 注解的几乎所有的情况下，默认的注解配置处理是假定的。此外，当使用 AnnotationConfigApplicationContext， 注解配置处理器总会被注册，以为着任何试图在 @ComponentScan 级别是扫描失效的行为都将被忽略。

​	通俗的讲，@ComponentScan 注解会自动扫描指定包下的全部标有 @Component注解 的类，并注册成bean，当然包括 @Component 下的子注解@Service、@Repository、@Controller。@ComponentScan 注解没有类似的属性。

参照：https://blog.csdn.net/claram/article/details/75125749 

# 七、自动配置原理

​	主要是通过@EnableAutoConfiguration注解实现的，在@EnableAutoConfiguration注解里面面可以看到，主要是通过引入AutoConfigurationImportSelector这个类实现的自动装配(2.x版本，1.x版本的好像是EnableAutoConfigurationImportSelector这个类，然后这个类继承了AutoConfigurationImportSelector.java)：

```Java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
   String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
   /**
    * Exclude specific auto-configuration classes such that they will never be applied.
    * @return the classes to exclude
    */
   Class<?>[] exclude() default {};
   /**
    * Exclude specific auto-configuration class names such that they will never be
    * applied.
    * @return the class names to exclude
    * @since 1.3.0
    */
   String[] excludeName() default {};
}
```

> @Import其实就是引入一个或多个配置，可以导入普通类，也可以导入配置类。
> @Import用来导入一个或多个类（会被spring容器管理），或者配置类（配置类里的@Bean标记的类也会被spring容器管理）
>
> 如：@Import({User.class,People.class, MyConfig.class})

在AutoConfigurationImportSelector类中，有一个selectImports方法，它返回的数组（类的全类名）都会被纳入到spring容器中，主要通过在selectImports方法中调用getCandidateConfigurations方法。

> **selectImports方法在springboot启动流程——bean实例化前被执行，返回要实例化的类信息列表。如果获取到类信息，spring可以通过类加载器将类加载到jvm中，现在我们已经通过spring-boot的starter依赖方式依赖了我们需要的组件，那么这些组件的类信息在select方法中就可以被获取到。**

```Java
//AutoConfigurationImportSelector.java
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
   if (!isEnabled(annotationMetadata)) {
      return NO_IMPORTS;
   }
   AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
         .loadMetadata(this.beanClassLoader);
   AnnotationAttributes attributes = getAttributes(annotationMetadata);
    // 接收返回的数组(类的全类名)
   List<String> configurations = getCandidateConfigurations(annotationMetadata,
         attributes);
   configurations = removeDuplicates(configurations);
   Set<String> exclusions = getExclusions(annotationMetadata, attributes);
   checkExcludedClasses(configurations, exclusions);
   configurations.removeAll(exclusions);
   configurations = filter(configurations, autoConfigurationMetadata);
   fireAutoConfigurationImportEvents(configurations, exclusions);
   return StringUtils.toStringArray(configurations);
}

// 到classpath下的读取META-INF/spring.factories文件的配置，并返回一个字符串数组。
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
			AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
				getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
		Assert.notEmpty(configurations,
				"No auto configuration classes found in META-INF/spring.factories. If you "
						+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}
// SpringFactoriesLoader.java
public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
        String factoryClassName = factoryClass.getName();
        return (List)loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
    }
```

> #### @EnableAutoConfiguration 作用：
>
> -  从classpath中搜索所有META-INF/spring.factories配置文件然后，将其中org.springframework.boot.autoconfigure.EnableAutoConfiguration  key对应的配置项加载到spring容器
> - 只有spring.boot.enableautoconfiguration为true（默认为true）的时候，才启用自动配置
>
> - @EnableAutoConfiguration还可以进行排除，排除方式有2中，一是根据class来排除（exclude），二是根据class name（excludeName）来排除，其内部实现的关键点有
>    1）ImportSelector 该接口的方法的返回值都会被纳入到spring容器管理中
>    2）SpringFactoriesLoader 该类可以从classpath中搜索所有META-INF/spring.factories配置文件，并读取配置

![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/1650e64b3fe3fa0a.png)

总结：

1. Spring Boot 在启动时扫描项目所依赖的 jar 包，寻找包含`spring.factories` 文件的 jar 包。
2. 根据 `spring.factories` 配置加载 AutoConfigure 类。
3. 根据 `@Conditional` 等条件注解的条件，进行自动配置并将 Bean 注入 Spring IoC 中。

参照：

- [《@EnableAutoConfiguration 注解的工作原理》](https://www.jianshu.com/p/464d04c36fb1) 。
- [《一个面试题引起的 Spring Boot 启动解析》](https://juejin.im/post/5b679fbc5188251aad213110)

# 八、读取配置文件的方式

1. `@Value` 注解，读取配置到属性。最最最常用。

   > 另外，支持和 `@PropertySource` 注解一起使用，指定使用的配置文件。
   >
   > 如： @Value("${info.address}")；其中在配置文件中：info.address=USA

2. `@ConfigurationProperties` 注解，读取配置到类上。

   > 另外，支持和 `@PropertySource` 注解一起使用，指定使用的配置文件。
   >
   > 如：@ConfigurationProperties(prefix ="info")，然后定义相关 的info属性

3. 读取指定文件

   > 资源目录下建立config/db-config.properties:
   >
   > db.username=root
   >
   > db.password=123456
   >
   > 3.1 @PropertySource+@Value注解读取方式
   >
   > @PropertySource(value ={"config/db-config.properties"})+ @Value("${db.username}")
   >
   > **注意：@PropertySource不支持yml文件读取。**
   >
   > 3.2 @PropertySource+@ConfigurationProperties注解读取方式
   >
   > 1. `@ConfigurationProperties(prefix ="db")`
   > 2. `@PropertySource(value ={"config/db-config.properties"})`
   > 3. 在java类中定义相关的属性：private String username;



参考 [《Spring Boot 读取配置的几种方式》](https://aoyouzi.iteye.com/blog/2422837) 。

# 九、与其他框架的集成

## 1.与 Spring MVC 集成

1. 引入 `spring-boot-starter-web` 的依赖。

2. 实现 WebMvcConfigurer 接口，可添加自定义的 Spring MVC 配置。

   > ​	因为 Spring Boot 2 基于 JDK 8 的版本，而 JDK 8 提供 `default` 方法，所以 **Spring Boot 2 废弃了 WebMvcConfigurerAdapter 适配类，直接使用 WebMvcConfigurer 即可。**

```java
// WebMvcConfigurer.java
public interface WebMvcConfigurer {

    /** 配置路径匹配器 **/
    default void configurePathMatch(PathMatchConfigurer configurer) {}
    
    /** 配置内容裁决的一些选项 **/
    default void configureContentNegotiation(ContentNegotiationConfigurer configurer) { }

    /** 异步相关的配置 **/
    default void configureAsyncSupport(AsyncSupportConfigurer configurer) { }

    default void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) { }

    default void addFormatters(FormatterRegistry registry) {
    }

    /** 添加拦截器 **/
    default void addInterceptors(InterceptorRegistry registry) { }

    /** 静态资源处理 **/
    default void addResourceHandlers(ResourceHandlerRegistry registry) { }

    /** 解决跨域问题 **/
    default void addCorsMappings(CorsRegistry registry) { }

    default void addViewControllers(ViewControllerRegistry registry) { }

    /** 配置视图解析器 **/
    default void configureViewResolvers(ViewResolverRegistry registry) { }

    /** 添加参数解析器 **/
    default void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
    }
    /** 添加返回值处理器 **/
    default void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> handlers) { }
    /** 这里配置视图解析器 **/
    default void configureMessageConverters(List<HttpMessageConverter<?>> converters) { }
    /** 配置消息转换器 **/
    default void extendMessageConverters(List<HttpMessageConverter<?>> converters) { }
   /** 配置异常处理器 **/
    default void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) { }
    default void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) { }
    @Nullable
    default Validator getValidator() { return null; }
    @Nullable
    default MessageCodesResolver getMessageCodesResolver() {  return null; }
}
```

在使用 Spring MVC 时，我们一般会做如下几件事情：

1. 实现自己项目需要的拦截器，并在 WebMvcConfigurer 实现类中配置。可参见 [MVCConfiguration](https://github.com/YunaiV/oceans/blob/2a2d3746905f1349e260e88049e7e28346c7648f/bff/webapp-bff/src/main/java/cn/iocoder/oceans/webapp/bff/config/MVCConfiguration.java) 类。
2. 配置 `@ControllerAdvice` + `@ExceptionHandler` 注解，实现全局异常处理。可参见 [GlobalExceptionHandler](https://github.com/YunaiV/oceans/blob/2a2d3746905f1349e260e88049e7e28346c7648f/bff/webapp-bff/src/main/java/cn/iocoder/oceans/webapp/bff/config/GlobalExceptionHandler.java) 类。
3. 配置 `@ControllerAdvice` ，实现 ResponseBodyAdvice 接口，实现全局统一返回。可参见 [GlobalResponseBodyAdvice](https://github.com/YunaiV/oceans/blob/2a2d3746905f1349e260e88049e7e28346c7648f/bff/webapp-bff/src/main/java/cn/iocoder/oceans/webapp/bff/config/GlobalResponseBodyAdvice.java) 。

当然，有一点需要注意，WebMvcConfigurer、ResponseBodyAdvice、`@ControllerAdvice`、`@ExceptionHandler` 接口，都是 Spring MVC 框架自身已经有的东西。

- `spring-boot-starter-web` 的依赖，帮我们解决的是 Spring MVC 的依赖以及相关的 Tomcat 等组件。

## 2.与Spring Security集成

1. 引入 `spring-boot-starter-security` 的依赖。
2. 继承 WebSecurityConfigurerAdapter ，添加**自定义**的安全配置。

```java
@Configuration
@EnableWebSecurity      // 开启Spring Security的功能
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {   //继承WebSecurityConfigurerAdapter，并重写它的方法来设置一些web安全的细节

    /**
     * authorizeRequests():定义哪些URL需要被保护、哪些不需要被保护。
     *      例如下面代码指定了/和/home不需要任何认证就可以访问，其他的路径都必须通过身份验证。
     * 通过formLogin()定义当需要用户登录时候，转到的登录页面。
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/", "/home").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .loginPage("/login")
                .permitAll()
                .and()
                .logout()
                .permitAll();
    }

// 在内存中创建了一个用户，该用户的名称为user，密码为password，用户角色为USER。
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth
                .inMemoryAuthentication()
                .withUser("user").password("password").roles("USER");
    }
    @Bean
    public static PasswordEncoder passwordEncoder(){
        return NoOpPasswordEncoder.getInstance();
    }

}
```

## 3.与Spring Security OAuth2 的集成

参见 [《Spring Security OAuth2 入门》](http://www.iocoder.cn/Spring-Security/OAuth2-learning/) 文章

## 4.与JPA的集成

1. 引入 `spring-boot-starter-data-jpa` 的依赖。
2. 在 application 配置文件中，加入 JPA 相关的少量配置。当然，数据库的配置也要添加进去。
3. 具体编码。

详细的使用，可以参考：[《一起来学 SpringBoot 2.x | 第六篇：整合 Spring Data JPA》](http://www.iocoder.cn/Spring-Boot/battcn/v2-orm-jpa/)

有两点需要注意：

- Spring Boot 2 默认使用的数据库连接池是 [HikariCP](https://github.com/brettwooldridge/HikariCP) ，目前最好的性能的数据库连接池的实现。
- `spring-boot-starter-data-jpa` 的依赖，使用的默认 JPA 实现是 Hibernate 5.X 。

> 数据源配置：
>
> ```properties
> `spring.datasource.url=jdbc:mysql://localhost:3306/chapter5?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true&useSSL=falsespring.datasource.password=rootspring.datasource.username=root#spring.datasource.type# JPA配置spring.jpa.hibernate.ddl-auto=update# 输出日志spring.jpa.show-sql=true# 数据库类型spring.jpa.database=mysql`
> ```
>
> > ddl-auto 几种属性
>
> - **create：** 每次运行程序时，都会重新创建表，故而数据会丢失
> - **create-drop：** 每次运行程序时会先创建表结构，然后待程序结束时清空表
> - **upadte：** 每次运行程序，没有表时会创建表，如果对象发生改变会更新表结构，原有数据不会清空，只会更新（推荐使用）
> - **validate：** 运行程序会校验数据与数据库的字段类型是否相同，**字段不同会报错**
>
> 由于上面我们采用的是`spring.jpa.hibernate.ddl-auto=update`方式，因此这里可以跳过手动建表的操作

## 5.与MyBatis的集成

1. 引入 `mybatis-spring-boot-starter` 的依赖。
2. 在 application 配置文件中，加入 MyBatis 相关的少量配置。当然，数据库的配置也要添加进去。
3. 具体编码。

详细的使用，可以参考：[《一起来学 SpringBoot 2.x | 第七篇：整合 Mybatis》](http://www.iocoder.cn/Spring-Boot/battcn/v2-orm-mybatis/)

## 6.与RabbitMQ的集成

1. 引入 `spring-boot-starter-amqp` 的依赖
2. 在 application 配置文件中，加入 RabbitMQ 相关的少量配置。
3. 具体编码。

详细的使用，可以参考：

- [《一起来学 SpringBoot 2.x | 第十二篇：初探 RabbitMQ 消息队列》](http://www.iocoder.cn/Spring-Boot/battcn/v2-queue-rabbitmq/)
- [《一起来学 SpringBoot 2.x | 第十三篇：RabbitMQ 延迟队列》](http://www.iocoder.cn/Spring-Boot/battcn/v2-queue-rabbitmq-delay/)

## 7.与Kafka的集成

1. 引入 `spring-kafka` 的依赖。
2. 在 application 配置文件中，加入 Kafka 相关的少量配置。
3. 具体编码。

详细的使用，可以参考：

- [《Spring Boot系列文章（一）：SpringBoot Kafka 整合使用》](http://www.54tianzhisheng.cn/2018/01/05/SpringBoot-Kafka/)

## 8.与RocketMQ的集成

1. 引入 `rocketmq-spring-boot` 的依赖。
2. 在 application 配置文件中，加入 RocketMQ 相关的少量配置。
3. 具体编码。

详细的使用，胖友可以参考：

- [《我用这种方法在 Spring 中实现消息的发送和消费》](http://www.iocoder.cn/RocketMQ/start/spring-boot-example)

# 十、日志框架

Spring Boot 支持的日志框架有：

- Logback
- Log4j2
- Log4j
- Java Util Logging

默认使用的是 Logback 日志框架，也是目前较为推荐的，具体配置，可以参见 [《一起来学 SpringBoot 2.x | 第三篇：SpringBoot 日志配置》](http://www.iocoder.cn/Spring-Boot/battcn/v2-config-logs/) 。

因为 Log4j2 的性能更加优秀，也有人在生产上使用，可以参考 [《Spring Boot Log4j2 日志性能之巅》](https://www.jianshu.com/p/f18a9cff351d) 配置。



参考和推荐如下文章：

- 我有面试宝典 [《[经验分享\] Spring Boot面试题总结》](http://www.wityx.com/post/242_1_1.html)
- Java 知音 [《Spring Boot 面试题精华》](https://cloud.tencent.com/developer/article/1348086)
- 祖大帅 [《一个面试题引起的 Spring Boot 启动解析》](https://juejin.im/post/5b679fbc5188251aad213110)
- 大胡子叔叔_ [《Spring Boot + Spring Cloud 相关面试题》](https://blog.csdn.net/panhaigang123/article/details/79587612)
- 墨斗鱼博客 [《20 道 Spring Boot 面试题》](https://www.mudouyu.com/article/26)
- 夕阳雨晴 [《Spring Boot Starter 的面试题》](https://blog.csdn.net/sun1021873926/article/details/78176354)
- 芋道源码