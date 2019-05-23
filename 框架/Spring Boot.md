# 一、Spring Boot的核心功能

- 1、独立运行 Spring 项目

  Spring Boot 可以以 jar 包形式独立运行，运行一个 Spring Boot 项目只需通过 `java -jar xx.jar` 来运行。

- 2、内嵌 Servlet 容器

  Spring Boot 可以选择内嵌 Tomcat、Jetty 或者 Undertow，这样我们无须以 war 包形式部署项目。

  > 在 Spring Boot 未出来的时候，大多数 Web 项目，是打包成 war 包，部署到 Tomcat、Jetty 等容器。

- 3、提供 Starter 简化 Maven 配置

  Spring 提供了一系列的 starter pom 来简化 Maven 的依赖加载。例如，当你使用了 `spring-boot-starter-web` ，会自动加入如下依赖：

![`spring-boot-starter-web` ç pom æä"¶](/Users/jack/Desktop/md/images/01-6804336.png)

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

通过执行 `mvn clean package -P ${profile}` 来指定使用哪个profile。













参照：芋道源码