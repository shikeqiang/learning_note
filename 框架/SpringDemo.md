# pom.xml

- 自动引⼊入 spring-boot-dependencies
- 自动配置 spring-boot-maven-plugin

金额相关的依赖(在实体类中需要在金额字段上使用Type注解)：

```xml
<dependency>
   <groupId>org.joda</groupId>
   <artifactId>joda-money</artifactId>
   <version>1.0.1</version>
</dependency>
<!--映射-->
<dependency>
	<groupId>org.jadira.usertype</groupId>
    <artifactId>usertype.core</artifactId>
	<version>6.0.1.GA</version>
</dependency>
```

# 慢 SQL日志：

系统属性配置 

- druid.stat.logSlowSql=true
- druid.stat.slowSqlMillis=3000

**Spring Boot** 

- spring.datasource.druid.filter.stat.enabled=true
- spring.datasource.druid.filter.stat.log-slow-sql=true
- spring.datasource.druid.filter.stat.slow-sql-millis=3000

# JPA

## 常用注解

![image-20190812232647436](/Users/jack/Desktop/md/images/image-20190812232647436.png)

![image-20190812232811499](/Users/jack/Desktop/md/images/image-20190812232811499.png)

例子：

```java
@Entity
@Table(name = "T_MENU")
@Builder
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Coffee implements Serializable {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
    @Column
    @Type(type = "org.jadira.usertype.moneyandcurrency.joda.PersistentMoneyAmount",
            parameters = {@org.hibernate.annotations.Parameter(name = "currencyCode", value = "CNY")})
    private Money price;
    @Column(updatable = false)
    @CreationTimestamp
    private Date createTime;
    @UpdateTimestamp
    private Date updateTime;
}
```

```java
@Entity
@Table(name = "T_ORDER")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class CoffeeOrder implements Serializable {
    @Id
    @GeneratedValue
    private Long id;
    private String customer;
    @ManyToMany
    @JoinTable(name = "T_ORDER_COFFEE")
    private List<Coffee> items;
    @Column(nullable = false)
    private Integer state;
    @Column(updatable = false)
    @CreationTimestamp
    private Date createTime;
    @UpdateTimestamp
    private Date updateTime;
}
```

![image-20190813222442457](/Users/jack/Desktop/md/images/image-20190813222442457.png)

![image-20190813230236586](/Users/jack/Desktop/md/images/image-20190813230236586.png)

![image-20190813230400409](/Users/jack/Desktop/md/images/image-20190813230400409.png)

![image-20190813231810303](/Users/jack/Desktop/md/images/image-20190813231810303.png)

==接口中的方法是如何被解释的：==

![image-20190813232020848](/Users/jack/Desktop/md/images/image-20190813232020848.png)

Part类中解析。

# mybatis

## 配置

- mybatis.mapper-locations = classpath*:mapper/**/*.xml 
- mybatis.type-aliases-package = 类型别名的包名 
- mybatis.type-handlers-package = TypeHandler扫描包名 
- mybatis.configuration.map-underscore-to-camel-case = true 

## 定义与扫描

- @MapperScan 配置扫描位置 
- @Mapper 定义接⼝
- 映射的定义—— XML 与注解 

# Spring

## Spring 的应⽤程序上下⽂

![image-20190825224643010](/Users/jack/Desktop/md/images/image-20190825224643010.png)

![image-20190825224931325](/Users/jack/Desktop/md/images/image-20190825224931325.png)

![image-20190825225625882](/Users/jack/Desktop/md/images/image-20190825225625882.png)

```java
@Controller
@RequestMapping("/coffee")
public class CoffeeController {
    @Autowired
    private CoffeeService coffeeService;

    @GetMapping(path = "/", params = "!name")  // 不存在name参数才会匹配上
    @ResponseBody
    public List<Coffee> getAll() {
        return coffeeService.getAllCoffee();
    }

    @RequestMapping(path = "/{id}", method = RequestMethod.GET,
            produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    @ResponseBody
    public Coffee getById(@PathVariable Long id) {// 获取id的值
        Coffee coffee = coffeeService.getCoffee(id);
        return coffee;
    }

    @GetMapping(path = "/", params = "name")
    @ResponseBody
    public Coffee getByName(@RequestParam String name) {
        return coffeeService.getCoffee(name);
    }
}
```

## 定义类型转换和Multipart 上传

![image-20190826225051677](/Users/jack/Desktop/md/images/image-20190826225051677.png)

## 定义校验

- 通过 Validator 对绑定结果进⾏校验 
  - Hibernate Validator 

- @Valid 注解 
- BindingResult 

## MultiPart上传

- 配置 MultipartResolver
  - Spring Boot 自动配置 MultipartAutoConfiguration 

- 配置 MultipartResolver 
- 支持类型 multipart/form-data 
- MultipartFile 类型 

## 视图解析

### 视图解析的实现基础

![image-20190826231127242](/Users/jack/Desktop/md/images/image-20190826231127242.png)

### DispatcherServlet 中的视图解析逻辑

![image-20190826231147271](/Users/jack/Desktop/md/images/image-20190826231147271.png)

使⽤用 **@**ResponseBody 的情况 

- 在 HandlerAdapter.handle() 的中完成了了 Response 输出 
  - RequestMappingHandlerAdapter.invokeHandlerMethod()
    - HandlerMethodReturnValueHandlerComposite.handleReturnValue()
      - RequestResponseBodyMethodProcessor.handleReturnValue()

### 配置 MessageConverter

- Spring Boot ⾃自动查找 HttpMessageConverters 进⾏行行注册 
- 通过 WebMvcConfigurer 的 configureMessageConverters() 

![image-20190826232141403](/Users/jack/Desktop/md/images/image-20190826232141403.png)

### 使用 Thymeleaf

![image-20190826231635292](/Users/jack/Desktop/md/images/image-20190826231635292.png)

## 异常处理

### 核⼼接口 

HandlerExceptionResolver

### 实现类

- SimpleMappingExceptionResolver
- DefaultHandlerExceptionResolver
- ResponseStatusExceptionResolver
- ExceptionHandlerExceptionResolver

### 处理方法：

@ExceptionHandler

### 添加位置

- @Controller / @RestController
- @ControllerAdvice / @RestControllerAdvice(优先级低于上面的两个注解)

```java
@RestControllerAdvice
public class GlobalControllerAdvice {
    @ExceptionHandler(ValidationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Map<String, String> validationExceptionHandler(ValidationException exception) {
        Map<String, String> map = new HashMap<>();
        map.put("message", exception.getMessage());
        return map;
    }
}
```

## 拦截器

核心接口：HandlerInteceptor

​				boolean preHandle()
​				void postHandle()
​				void afterCompletion()

针对 **@ResponseBody** 和 **ResponseEntity** 的情况：ResponseBodyAdvice	

针对异步请求的接⼝：AsyncHandlerInterceptor的void afterConcurrentHandlingStarted()

### 配置方式

常规方法：WebMvcConfigurer.addInterceptors()

**Spring Boot** 中的配置：

- 创建一个带 @Configuration 的 WebMvcConfigurer 配置类
- 不能带 @EnableWebMvc(想彻底⾃己控制 MVC 配置除外，即自定义MVC配置)

```java 
// 日志拦截器
@Slf4j
public class PerformanceInteceptor implements HandlerInterceptor {
    private ThreadLocal<StopWatch> stopWatch = new ThreadLocal<>();

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        StopWatch sw = new StopWatch();
        stopWatch.set(sw);
        sw.start();
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        stopWatch.get().stop();
        stopWatch.get().start();
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        StopWatch sw = stopWatch.get();
        sw.stop();
        String method = handler.getClass().getSimpleName();
        if (handler instanceof HandlerMethod) {
            String beanType = ((HandlerMethod) handler).getBeanType().getName();
            String methodName = ((HandlerMethod) handler).getMethod().getName();
            method = beanType + "." + methodName;
        }
        // 定义日志输出内容及格式
        log.info("{};{};{};{};{}ms;{}ms;{}ms", request.getRequestURI(), method,
                response.getStatus(), ex == null ? "-" : ex.getClass().getSimpleName(),
                sw.getTotalTimeMillis(), sw.getTotalTimeMillis() - sw.getLastTaskTimeMillis(),
                sw.getLastTaskTimeMillis());
        stopWatch.remove();
    }
}
```

## 扩展点

**BeanPostProcessor**：针对 Bean 实例，在 Bean 创建后提供定制逻辑回调

**BeanFactoryPostProcessor**：针对 Bean 定义，在容器器创建 Bean 前获取配置元数据，==Java Config 中需要定义为 static ⽅法==

# Spring Boot

## RestTemplate	

Spring Boot 中没有⾃动配置 RestTemplate 

Spring Boot 提供了 RestTemplateBuilder ：RestTemplateBuilder.build()

### 常⽤方法

**GET** 请求 ：getForObject() / getForEntity()

**POST** 请求 ：postForObject() / postForEntity() 

**PUT** 请求 :put() 

**DELETE** 请求 :delete() 

### 构造 URI

UriComponentsBuilder

构造相对于当前请求的 **URI**：ServletUriComponentsBuilder

构造指向 **Controller** 的 **URI**：MvcUriComponentsBuilder

### 传递 **HTTP Header**	

- RestTemplate.exchange()
- RequestEntity\<T> / ResponseEntity\<T>

### 类型转换

JsonSerializer / JsonDeserializer/@JsonComponent

### 解析泛型对象

RestTemplate.exchange() / ParameterizedTypeReference\<T>(可以自定义泛型类型)

```java
ParameterizedTypeReference<List<Coffee>> ptr =
      new ParameterizedTypeReference<List<Coffee>>() {};
ResponseEntity<List<Coffee>> list = restTemplate
      .exchange(coffeeUri, HttpMethod.GET, null, ptr);
list.getBody().forEach(c -> log.info("Coffee: {}", c));
```

### HTTP库

通⽤接⼝：ClientHttpRequestFactory

默认实现：SimpleClientHttpRequestFactory	

**Apache HttpComponents**：HttpComponentsClientHttpRequestFactory

**Netty**：Netty4ClientHttpRequestFactory

**OkHttp**：OkHttp3ClientHttpRequestFactory

### 优化底层请求策略

连接管理：PoolingHttpClientConnectionManager / KeepAlive 策略

超时设置：connectTimeout / readTimeout

SSL校验：证书检查策略

连接复用的默认实现：org.apache.http.impl.client.DefaultConnectionKeepAliveStrategy

```java
@Bean
public HttpComponentsClientHttpRequestFactory requestFactory() {
   //连接池管理器
   PoolingHttpClientConnectionManager connectionManager =
         new PoolingHttpClientConnectionManager(30, TimeUnit.SECONDS);
   connectionManager.setMaxTotal(200);
   connectionManager.setDefaultMaxPerRoute(20);
   // 定制httpClient
   CloseableHttpClient httpClient = HttpClients.custom()
         .setConnectionManager(connectionManager)
         .evictIdleConnections(30, TimeUnit.SECONDS)
         .disableAutomaticRetries() //自动重试
         // 有 Keep-Alive 认里面的值，没有的话永久有效
         //.setKeepAliveStrategy(DefaultConnectionKeepAliveStrategy.INSTANCE)
         // 换成自定义的
         .setKeepAliveStrategy(new CustomConnectionKeepAliveStrategy())
         .build();
   HttpComponentsClientHttpRequestFactory requestFactory =
         new HttpComponentsClientHttpRequestFactory(httpClient);
   return requestFactory;
}
```

### WebClient

​	这是⼀个以 Reactive ⽅式处理 HTTP 请求的⾮阻塞式的客户端，

⽀持的底层 **HTTP** 库：

Reactor Netty - ReactorClientHttpConnector

Jetty ReactiveStream HttpClient - JettyClientHttpConnector

#### 基本用法

创建WebClient：

WebClient.create() / WebClient.builder()

发起请求：get() / post() / put() / delete() / patch()

获得结果：retrieve() / exchange()

处理HTTP Status：onStatus()

应答正文：bodyToMono() / bodyToFlux()

```java
// 构造WebClient
@Bean
public WebClient webClient(WebClient.Builder builder) {
   return builder.baseUrl("http://localhost:8080").build();
}
```

## HTTP状态码

![image-20190831123846685](/Users/jack/Desktop/md/images/image-20190831123846685.png)

## 分布式会话

常见的解决方案：

粘性会话 Sticky Session

会话复制 Session Replication

集中会话 Centralized Session

### **Spring Session**

- 简化集群中的⽤户会话管理
- ⽆需绑定容器特定解决方案

支持的存储：Redis、MongoDB、JDBC、Hazelcast

#### 实现原理

定制 **HttpSession**

​	通过定制的 HttpServletRequest 返回定制的 HttpSession(可以屏蔽不同存储之间的差异)：

​		SessionRepositoryRequestWrapper、SessionRepositoryFilter、DelegatingFilterProxy

### 基于 Redis 的 HttpSession

引入依赖：spring-session-data-redis

基本配置：

@EnableRedisHttpSession、提供 RedisConnectionFactory、实现 AbstractHttpSessionApplicationInitializer (DelegatingFilterProxy)

Spring Boot 对 Spring Session 的⽀持:

![image-20190901000225979](/Users/jack/Desktop/md/images/image-20190901000225979.png)

## WebFlux

- ⽤于构建基于 Reactive 技术栈之上的 Web 应用程序
- 基于 Reactive Streams API ，运行在非阻塞服务器上（netty、jetty）
- 对于⾮阻塞 Web 应用及函数式编程的需要

性能：

- 请求的耗时并不会有很大的改善
- 仅需少量固定数量的线程和较少的内存即可实现扩展

![image-20190901000748194](/Users/jack/Desktop/md/images/image-20190901000748194.png)

两种编程模型：基于注解的控制器、函数式 Endpoints

![image-20190901001014615](/Users/jack/Desktop/md/images/image-20190901001014615.png)

## 自动装配

- 基于添加的 JAR 依赖自动对 Spring Boot 应⽤程序进行配置
- 主要是spring-boot-autoconfiguration这个jar包

开启自动配置：@EnableAutoConfiguration(exclude = Class<?>[])  //括号里可以排除掉某些不加载的类 、

​							@SpringBootApplication  启动类可以配置的注解，包含了上面的注解

### 实现原理

**@EnableAutoConfiguration**：

- AutoConfigurationImportSelector
- META-INF/spring.factories：org.springframework.boot.autoconfigure.EnableAutoConfiguration

#### 条件注解

@Conditional、@ConditionalOnClass(存在特定的类时生效)、@ConditionalOnBean(存在特定的Bean时生效)、@ConditionalOnMissingBean(不存在特定的Bean时生效)、@ConditionalOnProperty

#### 日志

命令行(IDEA的参数里面)加上 --debug，结果是由**ConditionEvaluationReportLoggingListener**：Positive matches、Negative matches、Exclusions、Unconditional classes

#### 自定义配置

编写 **Java Config**：   @Configuration

添加条件：@Conditional

定位自动配置：META-INF/spring.factories

#### 自动装配执行顺序：

@AutoConfigureBefore、@AutoConfigureAfter、@AutoConfigureOrder

### 自定义starter

autoconfigure 模块，包含自动配置代码

starter 模块，包含指向自动配置模块的依赖及其他相关依赖

### 外化配置加载顺序

![image-20190901110553151](/Users/jack/Desktop/md/images/image-20190901110553151.png)

![image-20190901110627071](/Users/jack/Desktop/md/images/image-20190901110627071.png)

![image-20190901110722969](/Users/jack/Desktop/md/images/image-20190901110722969.png)

@Configuration 类上的 @PropertySource

SpringApplication.setDefaultProperties() 设置的默认属性

## PropertySource

添加 **PropertySource**：

- \<context:property-placeholder>
- PropertySourcesPlaceholderConfigure：PropertyPlaceholderConfigurer
- @PropertySource
- @PropertySources

@ConfigurationProperties：

- 可以将属性绑定到结构化对象上
- ⽀持 Relaxed Binding
- ⽀持安全的类型转换
- @EnableConfigurationProperties

### 定制 PropertySource

![image-20190901114019518](/Users/jack/Desktop/md/images/image-20190901114019518.png)

```java
//加载到environment
@Slf4j
public class YapfEnvironmentPostProcessor implements EnvironmentPostProcessor {
    private PropertiesPropertySourceLoader loader = new PropertiesPropertySourceLoader();
    @Override
    public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
        MutablePropertySources propertySources = environment.getPropertySources();
        Resource resource = new ClassPathResource("yapf.properties");
        try {
            PropertySource ps = loader.load("YetAnotherPropertiesFile", resource)
                    .get(0);
            propertySources.addFirst(ps);
        } catch (Exception e) {
            log.error("Exception!", e);
        }
    }
}
```

## Actuator

​	可以监控并管理应用程序，主要是通过HTTP/JMX访问，依赖是spring-boot-starter-actuator。

常用Endpoint：

> Endpoint主要是用来监控应用服务的运行状况，并集成在Mvc中提供查看接口。

![image-20190901185343292](/Users/jack/Desktop/md/images/image-20190901185343292.png)

![image-20190901185500141](/Users/jack/Desktop/md/images/image-20190901185500141.png)

### 访问及配置

通过HTTP：/actuator/\<id>

端口与路径：

- management.server.address=
- management.server.port=
- management.endpoints.web.base-path=/actuator
- management.endpoints.web.path-mapping.\<id>=路径

开启 **Endpoint**:

- management.endpoint.\<id>.enabled=true
- management.endpoints.enabled-by-default=false

暴露 **Endpoint**：

- management.endpoints.jmx.exposure.exclude=
- management.endpoints.jmx.exposure.include=*
- management.endpoints.web.exposure.exclude=
- management.endpoints.web.exposure.include=info, health

## Health Indicator

目的：检查应⽤程序的运行状态

状态：

- DOWN - 503
- OUT_OF_SERVICE - 503
- UP - 200
- UNKNOWN - 200

### Spring Boot ⾃带的 Health Indicator

机制：

- 通过 HealthIndicatorRegistry 收集信息
- HealthIndicator 实现具体检查逻辑

配置项：

- management.health.defaults.enabled=true|false
- management.health.\<id>.enabled=true
- management.endpoint.health.show-details=never|when-authorized|always

![image-20190901201313867](/Users/jack/Desktop/md/images/image-20190901201313867.png)

### 自定义 Health Indicator

方法：

- 实现 HealthIndicator 接⼝
- 根据自定义检查逻辑返回对应 Health 状态：Health 中包含状态和详描述信息

```java
// 可以监测/查看咖啡是否还有
@Component
public class CoffeeIndicator implements HealthIndicator {
    @Autowired
    private CoffeeService coffeeService;

    @Override
    public Health health() {
        long count = coffeeService.getCoffeeCount();
        Health health;
        if (count > 0) {
            health = Health.up()
                    .withDetail("count", count)
                    .withDetail("message", "We have enough coffee.")
                    .build();
        } else {
            health = Health.down()
                    .withDetail("count", 0)
                    .withDetail("message", "We are out of coffee.")
                    .build();
        }
        return health;
    }
}
```

## Micrometer

​		这是个监控指标的度量类库。

特性：

- **多维度度量**：支持Tag(可以打上标签)
- ==预置大量探针：缓存、类加载器器、GC、CPU 利用率、线程池......==
- 与Spring深度整合

支持多种监控系统：

- Dimensional：AppOptics, Atlas, Azure Monitor, Cloudwatch, Datadog, Datadog StatsD, Dynatrace, Elastic, Humio, Influx, KairosDB, New Relic, Prometheus, SignalFx, Sysdig StatsD, Telegraf StatsD, Wavefront
- Hierarchical：Graphite, Ganglia, JMX, Etsy StatsD

**核心接口：Meter**

内置实现：

- Gauge, TimeGauge
- Timer, LongTaskTimer, FunctionTimer
- Counter, FunctionCounter
- DistributionSummary

```java
@Service
@Transactional
@Slf4j
public class CoffeeOrderService implements MeterBinder {
    @Autowired
    private CoffeeOrderRepository orderRepository;

    private Counter orderCounter = null;

    public CoffeeOrder get(Long id) {
        return orderRepository.getOne(id);
    }

    public CoffeeOrder createOrder(String customer, Coffee...coffee) {
        CoffeeOrder order = CoffeeOrder.builder()
                .customer(customer)
                .items(new ArrayList<>(Arrays.asList(coffee)))
                .state(OrderState.INIT)
                .build();
        CoffeeOrder saved = orderRepository.save(order);
        log.info("New Order: {}", saved);
        orderCounter.increment();//统计订单总量，后面可以做监测
        return saved;
    }

    public boolean updateState(CoffeeOrder order, OrderState state) {
        if (state.compareTo(order.getState()) <= 0) {
            log.warn("Wrong State order: {}, {}", state, order.getState());
            return false;
        }
        order.setState(state);
        orderRepository.save(order);
        log.info("Updated Order: {}", order);
        return true;
    }

    @Override
    public void bindTo(MeterRegistry meterRegistry) {
        this.orderCounter = meterRegistry.counter("order.count");
    }
}
```

### Micrometer in Spring Boot 2.x

一些 **URL**：

- /actuator/metrics
- /actuator/prometheus

⼀些配置项：

- management.metrics.export.*
- management.metrics.tags.*
- management.metrics.enable.*
- management.metrics.distribution.*
- management.metrics.web.server.auto-time-requests(耗时监控)

核心度量项：JVM、CPU、⽂件句柄数、⽇志、启动时间

其他度量项：

- Spring MVC、Spring WebFlux
- Tomcat、Jersey JAX-RS
- RestTemplate、WebClient
- 缓存、数据源、Hibernate
- Kafka、RabbitMQ

⾃定义度量指标：

- 通过 MeterRegistry 注册 Meter
- 提供 MeterBinder Bean 让 Spring Boot ⾃动绑定
- 通过 MeterFilter 进行定制

# maven

查看依赖：

- mvn dependency:tree
- IDEA Maven Helper 插件

统一管理依赖：

- dependencyManagement
- Bill of Materials - bom























参照：极客时间之《玩转Spring全家桶》



