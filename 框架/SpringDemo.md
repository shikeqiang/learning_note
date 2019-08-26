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

参照：极客时间之《玩转Spring全家桶》



