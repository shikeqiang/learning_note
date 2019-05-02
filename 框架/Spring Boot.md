# 一、Spring Boot核心功能

- 1、独立运行 Spring 项目

  Spring Boot 可以以 jar 包形式独立运行，运行一个 Spring Boot 项目只需要通过 `java -jar xx.jar` 来运行。

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































































参照：芋道源码