pom.xml

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

### 实现原理

定制 **HttpSession**

​	通过定制的 HttpServletRequest 返回定制的 HttpSession：

​		SessionRepositoryRequestWrapper、SessionRepositoryFilter、DelegatingFilterProxy











































































参照：极客时间之《玩转Spring全家桶》



