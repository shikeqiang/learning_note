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

# 常用JPA注解

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







