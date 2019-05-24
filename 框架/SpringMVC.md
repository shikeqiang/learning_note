# 一、核心组件

## 1.概述：

Spring MVC 一共有九大核心组件，分别是：

- MultipartResolver
- LocaleResolver
- ThemeResolver
- HandlerMapping
- HandlerAdapter
- HandlerExceptionResolver
- RequestToViewNameTranslator
- ViewResolver
- FlashMapManager

初始化如下：

```Java
// DispatcherServlet.java
/** MultipartResolver used by this servlet. */
@Nullable
private MultipartResolver multipartResolver;

/** LocaleResolver used by this servlet. */
@Nullable
private LocaleResolver localeResolver;

/** ThemeResolver used by this servlet. */
@Nullable
private ThemeResolver themeResolver;

/** List of HandlerMappings used by this servlet. */
@Nullable
private List<HandlerMapping> handlerMappings;

/** List of HandlerAdapters used by this servlet. */
@Nullable
private List<HandlerAdapter> handlerAdapters;

/** List of HandlerExceptionResolvers used by this servlet. */
@Nullable
private List<HandlerExceptionResolver> handlerExceptionResolvers;

/** RequestToViewNameTranslator used by this servlet. */
@Nullable
private RequestToViewNameTranslator viewNameTranslator;

/** FlashMapManager used by this servlet. */
@Nullable
private FlashMapManager flashMapManager;

/** List of ViewResolvers used by this servlet. */
@Nullable
private List<ViewResolver> viewResolvers;
    
/**
 * This implementation calls {@link #initStrategies}.
 */
@Override
protected void onRefresh(ApplicationContext context) {
	initStrategies(context);
}
    
/**
 * Initialize the strategy objects that this servlet uses.
 * <p>May be overridden in subclasses in order to initialize further strategy objects.
 */
protected void initStrategies(ApplicationContext context) {
    // 初始化 MultipartResolver
	initMultipartResolver(context);
	// 初始化 LocaleResolver
	initLocaleResolver(context);
	// 初始化 ThemeResolver
	initThemeResolver(context);
	// 初始化 HandlerMappings
	initHandlerMappings(context);
	// 初始化 HandlerAdapters
	initHandlerAdapters(context);
	// 初始化 HandlerExceptionResolvers 
	initHandlerExceptionResolvers(context);
	// 初始化 RequestToViewNameTranslator
	initRequestToViewNameTranslator(context);
	// 初始化 ViewResolvers
	initViewResolvers(context);
	// 初始化 FlashMapManager
	initFlashMapManager(context);
}
```

## 2. MultipartResolver

`org.springframework.web.multipart.MultipartResolver` ，内容类型( `Content-Type` )为 `multipart/*` 的请求的解析器接口。

​	例如，文件上传请求，MultipartResolver 会将 HttpServletRequest 封装成 MultipartHttpServletRequest ，这样从 MultipartHttpServletRequest 中获得上传的文件。具体的使用示例，参见 [《spring-boot 上传文件 MultiPartFile 获取不到文件问题解决》](https://blog.csdn.net/happy_cheng/article/details/54178392)

​	关于内容类型( `Content-Type` )为 `multipart/*` ，可以看看 [《HTTP 协议之 multipart/form-data 请求分析》](https://blog.csdn.net/five3/article/details/7181521) 文章。

### MultipartResolver 接口，代码如下：

```Java
// MultipartResolver.java
public interface MultipartResolver {
	/**
     * 是否为 multipart 请求
	 */
	boolean isMultipart(HttpServletRequest request);

	/**
     * 将 HttpServletRequest 请求封装成 MultipartHttpServletRequest 对象
	 */
	MultipartHttpServletRequest resolveMultipart(HttpServletRequest request) throws MultipartException;
	/**
     * 清理处理 multipart 产生的资源，例如临时文件
     *
	 */
	void cleanupMultipart(MultipartHttpServletRequest request);
}
```

## 3. LocaleResolver

`org.springframework.web.servlet.LocaleResolver` ，**本地化( 国际化 )解析器接口**。代码如下：

```Java
// LocaleResolver.java
public interface LocaleResolver {
	/**
     * 从请求中，解析出要使用的语言。例如，请求头的 "Accept-Language"
	 */
	Locale resolveLocale(HttpServletRequest request);
	/**
     * 设置请求所使用的语言
	 */
	void setLocale(HttpServletRequest request, @Nullable HttpServletResponse response, @Nullable Locale locale);

}
```

具体的使用示例，参见 [《SpringMVC学习系列（8） 之 国际化》](https://www.cnblogs.com/liukemng/p/3750117.html) 。

## 4. ThemeResolver

`org.springframework.web.servlet.ThemeResolver` ，**主题解析器接口**。代码如下：

```Java
// ThemeResolver.java
public interface ThemeResolver {
	/**
     * 从请求中，解析出使用的主题。例如，从请求头 User-Agent ，判断使用 PC 端，还是移动端的主题
	 */
	String resolveThemeName(HttpServletRequest request);

	/**
     * 设置请求，所使用的主题。
	 */
	void setThemeName(HttpServletRequest request, @Nullable HttpServletResponse response, @Nullable String themeName);
}
```

具体的使用示例，参见 [《如何使用 Spring MVC 主题》](https://blog.csdn.net/neweastsun/article/details/79213867) 。

**因为现在的前端，基本和后端做了分离，所以这个功能已经越来越少用了。**

## 5. HandlerMapping

`org.springframework.web.servlet.HandlerMapping` ，处理器匹配接口，**根据请求( `handler` )获得其的处理器( `handler`)和拦截器们( HandlerInterceptor 数组 )。**代码如下：

```Java
// HandlerMapping.java
public interface HandlerMapping {
	String PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE = HandlerMapping.class.getName() + ".pathWithinHandlerMapping";
	String BEST_MATCHING_PATTERN_ATTRIBUTE = HandlerMapping.class.getName() + ".bestMatchingPattern";
	String INTROSPECT_TYPE_LEVEL_MAPPING = HandlerMapping.class.getName() + ".introspectTypeLevelMapping";
	String URI_TEMPLATE_VARIABLES_ATTRIBUTE = HandlerMapping.class.getName() + ".uriTemplateVariables";
	String MATRIX_VARIABLES_ATTRIBUTE = HandlerMapping.class.getName() + ".matrixVariables";
	String PRODUCIBLE_MEDIA_TYPES_ATTRIBUTE = HandlerMapping.class.getName() + ".producibleMediaTypes";
	/**
     * 获得请求对应的处理器和拦截器们
	 */
	@Nullable
	HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
}
```

- **返回的对象类型是 HandlerExecutionChain ，它包含处理器( `handler` )和拦截器们( HandlerInterceptor 数组 )。**简单代码如下：

  ```Java
  // HandlerExecutionChain.java
  /**
   * 处理器
   */
  private final Object handler;
  /**
   * 拦截器数组
   */
  @Nullable
  private HandlerInterceptor[] interceptors;
  ```

  - 注意，处理器的类型可能和我们想的不太一样，是个 **Object** 类型。

## 6. HandlerAdapter

`org.springframework.web.servlet.HandlerAdapter` ，处理器适配器接口。代码如下：

```Java
// HandlerAdapter.java
public interface HandlerAdapter {
	/**
     * 是否支持该处理器
	 */
	boolean supports(Object handler);

	/**
     * 执行处理器，返回 ModelAndView 结果
	 */
	@Nullable
	ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
	/**
     * 返回请求的最新更新时间。
     *
     * 如果不支持该操作，则返回 -1 即可
	 */
	long getLastModified(HttpServletRequest request, Object handler);
}
```

- 因为，处理器 `handler` 的类型是 Object 类型，需要有一个调用者来实现 `handler` 是怎么被使用，怎么被执行。而 HandlerAdapter 的用途就在于此。

## 7. HandlerExceptionResolver

`org.springframework.web.servlet.HandlerExceptionResolver` ，**处理器异常解析器接口，将处理器( `handler` )执行时发生的异常，解析( 转换 )成对应的 ModelAndView 结果。**代码如下：

```Java
// HandlerExceptionResolver.java
public interface HandlerExceptionResolver {
	/**
     * 解析异常，转换成对应的 ModelAndView 结果
	 */
	@Nullable
	ModelAndView resolveException(
			HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex);
}
```

## 8. RequestToViewNameTranslator

`org.springframework.web.servlet.RequestToViewNameTranslator` ，请求到视图名的转换器接口。代码如下：

```Java
// RequestToViewNameTranslator.java
public interface RequestToViewNameTranslator {
	/**
     * 根据请求，获得其视图名
	 */
	@Nullable
	String getViewName(HttpServletRequest request) throws Exception;
}
```

## 9. ViewResolver

`org.springframework.web.servlet.ViewResolver` ，**实体解析器接口，根据视图名和国际化，获得最终的视图 View 对象。**代码如下：

```Java
// ViewResolver.java
public interface ViewResolver {
	/**
     * 根据视图名和国际化，获得最终的 View 对象
	 */
	@Nullable
	View resolveViewName(String viewName, Locale locale) throws Exception;
}
```

​	**ViewResolver 的实现类比较多，例如说，InternalResourceViewResolver 负责解析 JSP 视图，FreeMarkerViewResolver 负责解析 Freemarker 视图。**

## 10. FlashMapManager

`org.springframework.web.servlet.FlashMapManager` ，F**lashMap 管理器接口，负责重定向时，保存参数到临时存储中**。代码如下：

```Java
// FlashMapManager.java
public interface FlashMapManager {
	/**
     * 恢复参数，并将恢复过的和超时的参数从保存介质中删除
	 */
	@Nullable
	FlashMap retrieveAndUpdate(HttpServletRequest request, HttpServletResponse response);
	/**
     * 将参数保存起来
	 */
	void saveOutputFlashMap(FlashMap flashMap, HttpServletRequest request, HttpServletResponse response);

}
```

默认情况下，这个临时存储会是 Session 。也就是说：

- 重定向前，保存参数到 Seesion 中。
- 重定向后，从 Session 中获得参数，并移除。

具体使用示例，参见 [《Spring MVC Flash Attribute 的讲解与使用示例》](https://www.oschina.net/translate/spring-mvc-flash-attribute-example) 一文。

实际场景下，使用的非常少，特别是前后端分离之后。

# 二、DispatchServlet处理请求的过程

FROM [《Spring MVC 原理探秘 —— 一个请求的旅行过程》](https://www.tianxiaobo.com/2018/06/29/Spring-MVC-%E5%8E%9F%E7%90%86%E6%8E%A2%E7%A7%98-%E4%B8%80%E4%B8%AA%E8%AF%B7%E6%B1%82%E7%9A%84%E6%97%85%E8%A1%8C%E8%BF%87%E7%A8%8B/)

![img](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/15300766829012.jpg)

## ① **发送请求**

​	用户向服务器发送 HTTP 请求，请求被 Spring MVC 的调度控制器 DispatcherServlet 捕获。

## ② **映射处理器**

​	DispatcherServlet 根据请求 URL ，调用 HandlerMapping 获得该 Handler 配置的所有相关的对象（包括 **Handler** 对象以及 Handler 对象对应的**拦截器**），最后以 HandlerExecutionChain 对象的形式返回。

- 即 HandlerExecutionChain 中，包含对应的 **Handler** 对象和**拦截器**们。

> 此处，对应的方法如下：
>
> ```Java
> // HandlerMapping.java 
> @Nullable
> HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
> ```

## ③ **处理器适配**

​	**DispatcherServlet 根据获得的 Handler，选择一个合适的HandlerAdapter 。**（附注：如果成功获得 HandlerAdapter 后，此时将开始执行拦截器的 `preHandler(...)` 方法）。

​	**提取请求 Request 中的模型数据，填充 Handler 入参，开始执行Handler（Controller)。 在填充Handler的入参过程中，根据你的配置，Spring 将帮你做一些额外的工作**：

- HttpMessageConverter ：会将请求消息（如 JSON、XML 等数据）转换成一个对象。
- 数据转换：对请求消息进行数据转换。如 String 转换成 Integer、Double 等。
- 数据格式化：对请求消息进行数据格式化。如将字符串转换成格式化数字或格式化日期等。
- 数据验证： 验证数据的有效性（长度、格式等），验证结果存储到 BindingResult 或 Error 中。

Handler(Controller) 执行完成后，向 DispatcherServlet 返回一个 ModelAndView 对象。

> 此处，对应的方法如下：
>
> ```Java
> // HandlerAdapter.java
> @Nullable
> ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
> ```

## ⑤ **解析视图**

根据返回的 ModelAndView ，选择一个适合的 ViewResolver（必须是已经注册到 Spring 容器中的 ViewResolver)，解析出 View 对象，然后返回给 DispatcherServlet。

> 此处，对应的方法如下：
>
> ```Java
> // ViewResolver.java
> @Nullable
> View resolveViewName(String viewName, Locale locale) throws Exception;
> ```

## ⑥ ⑦ **渲染视图** + **响应请求**

**ViewResolver 结合 Model 和 View，来渲染视图，并写回给用户( 浏览器 )。**

> 此处，对应的方法如下：
>
> ```Java
> // View.java
> void render(@Nullable Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception;
> ```

------

具体可以参照：

- [《精尽 Spring MVC 源码分析 —— 组件一览》](http://svip.iocoder.cn/Spring-MVC/Components-intro/)
- [《精尽 Spring MVC 源码分析 —— 请求处理一览》](http://svip.iocoder.cn/Spring-MVC/DispatcherServlet-process-request-intro/)

​	**但是对于目前主流的架构，前后端已经进行分离了，所以 Spring MVC 只负责 Model 和 Controller 两块**，而将 View 移交给了前端。所以，在上图中的步骤 ⑤ 和 ⑥ 两步，已经不在需要。

​	现在一般变成，在步骤 ③ 中，如果 Handler(Controller) 执行完后，**如果判断方法有 `@ResponseBody` 注解，则直接将结果写回给用户( 浏览器 )。**

​	**但是 HTTP 是不支持返回 Java POJO 对象的，所以需要将结果使用 [HttpMessageConverter](http://svip.iocoder.cn/Spring-MVC/HandlerAdapter-5-HttpMessageConverter/) 进行转换后，才能返回。例如说，大家所熟悉的 [FastJsonHttpMessageConverter](https://github.com/alibaba/fastjson/wiki/%E5%9C%A8-Spring-%E4%B8%AD%E9%9B%86%E6%88%90-Fastjson) ，将 POJO 转换成 JSON 字符串返回。**

还可以参照下面几张流程图：

> FROM [《SpringMVC - 运行流程图及原理分析》](https://blog.csdn.net/J080624/article/details/77990164)
>
> **流程示意图**：
>
> ![流程示意图](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/01-20190407121237875.png)
>
> **代码序列图**：
>
> ![代码序列图](http://static2.iocoder.cn/images/Spring/2022-02-21/02.png)
>
> ------
>
> FROM [《看透 Spring MVC：源代码分析与实践》](https://item.jd.com/11807414.html) P123
>
> **流程示意图**：
>
> ![《流程示意图》](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/03-20190407121237909.png)

# 三、相关注解

![image-20190417190824494](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190417190824494.png)

## @Controller 

`@Controller` 注解，它将一个类标记为 Spring Web MVC **控制器** Controller 。

## @RestController 和 @Controller 的区别

`@RestController` 注解，在 `@Controller` 基础上，增加了 `@ResponseBody` 注解，更加适合目前前后端分离的架构下，**提供 Restful API ，返回例如 JSON 数据格式。**当然，返回什么样的数据格式，根据客户端的 `"ACCEPT"` 请求头来决定。

## @RequestMapping

**`@RequestMapping` 注解，用于将特定 HTTP 请求方法映射到将处理相应请求的控制器中的特定类/方法。此注释可应用于两个级别：**

- 类级别：映射请求的 URL。
- 方法级别：映射 URL 以及 HTTP 请求方法。

## @RequestMapping 和 @GetMapping 区别

- `@RequestMapping` 可注解在类和方法上；`@GetMapping` 仅可注册在方法上。
- `@RequestMapping` 可进行 GET、POST、PUT、DELETE 等请求方法；`@GetMapping` 是 `@RequestMapping` 的 GET 请求方法的特例，目的是为了提高清晰度。

## 返回 JSON 格式使用什么注解？

==可以使用 **@ResponseBody** 注解，或者使用包含 `@ResponseBody` 注解的 **@RestController** 注解。==

当然，还是需要配合相应的支持 JSON 格式化的 HttpMessageConverter 实现类。例如，Spring MVC 默认使用 MappingJackson2HttpMessageConverter 。

## @PathVariable 注解与@RequestParam的区别

​	`@PathVariable` 注解，是 Spring MVC 中有用的注解之一，它允许您从URI 读取值，比如查询参数。它在使用 Spring 创建 RESTful Web 服务时特别有用，因为在 REST 中，资源标识符是 URI 的一部分。

​	@RequestParam注解和@PathVariable注解的区别，从字面上可以看出前者是获取请求里边携带的参数；后者是获取请求路径里边的变量参数。

**(例如：127.0.0.1/user/{userId}?userName=zhangshan,userId是路径上的变量，userName才是请求参数信息)**

> **1.@RequestParam注解**
>
> @RequestParam有三个参数：
>
> value：参数名；
>
> required：是否必需，默认为true，表示请求参数中必须包含该参数，如果不包含抛出异常。
>
> defaultValue：默认参数值，如果设置了该值自动将required设置为false，如果参数中没有包含该参数则使用默认值。
>
> 示例：@RequestParam(value = "userId", required = false, defaultValue = "1")
>
> **2.@PathVariable注解**
>
> ​	当使用@RequestMapping URI占位符映射时，Url中可以通过一个或多个{xxxx}占位符映射，通过@PathVariable可以绑定占位符参数到方法参数中。
>
> 例如：@PathVariable("userId") Long userId,@PathVariable("userName") String userName
>
> (注：Long类型可以根据需求自己改变String或int，Spring会自动做转换)
>
> @RequestMapping(“/user/{userId}/{userName}/query")
>

参照 [《Spring MVC 的 @RequestParam 注解和 @PathVariable 注解的区别》](https://blog.csdn.net/cx361006796/article/details/52829759) 。

## @RequestBody

​	@requestBody注解常用来处理content-type不是默认的application/x-www-form-urlcoded编码的内容，比如说：application/json或者是application/xml等。一般情况下来说常用其来处理application/json类型。

​	通过@requestBody可以将请求体中的JSON字符串绑定到相应的bean上，当然，也可以将其分别绑定到对应的字符串上。

​	前端通过post传数据过来,要用RequestBody注解。

#  四、WebApplicationContext 

**WebApplicationContext 是实现ApplicationContext接口的子类，专门为 WEB 应用准备的。**

- 它允许从相对于 Web 根目录的路径中**加载配置文件**，**完成初始化 Spring MVC 组件的工作**。
- 从 WebApplicationContext 中，可以获取 ServletContext 引用，整个 Web 应用上下文对象将作为属性放置在 ServletContext 中，以便 Web 应用环境可以访问 Spring 上下文。

关于这一块，如果想要详细了解，可以看看如下两篇文章：

- [《精尽 Spring MVC 源码分析 —— 容器的初始化（一）之 Root WebApplicationContext 容器》](http://svip.iocoder.cn/Spring-MVC/context-init-Root-WebApplicationContext/)
- [《精尽 Spring MVC 源码分析 —— 容器的初始化（二）之 Servlet WebApplicationContext 容器》](http://svip.iocoder.cn/Spring-MVC/context-init-Servlet-WebApplicationContext/)

# 五、Spring MVC 的异常处理

​	Spring MVC 提供了异常解析器 HandlerExceptionResolver 接口，将处理器( `handler` )执行时发生的异常，解析( 转换 )成对应的 ModelAndView 结果。代码如下：

```Java
// HandlerExceptionResolver.java
public interface HandlerExceptionResolver {
    /**
     * 解析异常，转换成对应的 ModelAndView 结果
     */
    @Nullable
    ModelAndView resolveException(
            HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex);
}
```

- **也就是说，如果异常被解析成功，则会返回 ModelAndView 对象。**
- 详细的源码解析，见 [《精尽 Spring MVC 源码解析 —— HandlerExceptionResolver 组件》](http://svip.iocoder.cn/Spring-MVC/HandlerExceptionResolver/) 。

​	一般情况下，我们使用 `@ExceptionHandler` 注解来实现过异常的处理，可以先看看 [《Spring 异常处理 ExceptionHandler 的使用》](https://www.jianshu.com/p/12e1a752974d) 。

# 六、Spring MVC的优点

1. 使用真的真的真的非常**方便**，无论是添加 HTTP 请求方法映射的方法，还是不同数据格式的响应。
2. 提供**拦截器机制**，可以方便的对请求进行拦截处理。
3. 提供**异常机制**，可以方便的对异常做统一处理。
4. 可以任意使用各种**视图**技术，而不仅仅局限于 JSP ，例如 Freemarker、Thymeleaf 等等。
5. 不依赖于 Servlet API (目标虽是如此，但是在实现的时候确实是依赖于 Servlet 的，当然仅仅依赖 Servlet ，而不依赖 Filter、Listener )。

# 七、Spring MVC 设定重定向和转发 

- 结果转发：在返回值的前面加 `"forward:/"` 。
- 重定向：在返回值的前面加上 `"redirect:/"` 。

当然，目前前后端分离之后，我们作为后端开发，已经很少有机会用上这个功能了。

## 八、Spring MVC 的 Controller 是不是单例？

绝绝绝大多数情况下，Controller 是**单例**。

那么，Controller 里一般不建议存在**共享的变量**。

## 九、Spring MVC 和 Struts2 的异同？

1. 入口不同
   - Spring MVC 的入门是一个 Servlet **控制器**。
   - Struts2 入门是一个 Filter **过滤器**。
2. 配置映射不同，
   - Spring MVC 是基于**方法**开发，传递参数是通过**方法形参**，一般设置为**单例**。
   - Struts2 是基于**类**开发，传递参数是通过**类的属性**，只能设计为**多例**。

3. 视图不同

- Spring MVC 通过参数解析器是将 Request 对象内容进行解析成方法形参，将响应数据和页面封装成 **ModelAndView** 对象，最后又将模型数据通过 **Request** 对象传输到页面。其中，如果视图使用 JSP 时，默认使用 **JSTL** 。
- Struts2 采用**值栈**存储请求和响应的数据，通过 **OGNL** 存取数据。

当然，更详细的也可以看看 [《面试题：Spring MVC 和 Struts2 的区别》](http://www.voidcn.com/article/p-ylualwcj-c.html) 一文。

# 十、Spring MVC 拦截器

`org.springframework.web.servlet.HandlerInterceptor` ，拦截器接口。代码如下：

```Java
// HandlerInterceptor.java
/**
 * 拦截处理器，在 {@link HandlerAdapter#handle(HttpServletRequest, HttpServletResponse, Object)} 执行之前
 */
default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
		throws Exception {
	return true;
}
/**
 * 拦截处理器，在 {@link HandlerAdapter#handle(HttpServletRequest, HttpServletResponse, Object)} 执行成功之后
 */
default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
		@Nullable ModelAndView modelAndView) throws Exception {
}
/**
 * 拦截处理器，在 {@link HandlerAdapter#handle(HttpServletRequest, HttpServletResponse, Object)} 执行完之后，无论成功还是失败
 *
 * 并且，只有该处理器 {@link #preHandle(HttpServletRequest, HttpServletResponse, Object)} 执行成功之后，才会被执行
 */
default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
		@Nullable Exception ex) throws Exception {
}
```

- 一共有三个方法，分别为：
  - `preHandle(...)` 方法，调用 Controller 方法之前执行。
  - `postHandle(...)` 方法，调用 Controller 方法之后执行。
  - `afterCompletion(…)`方法，处理完 Controller 方法返回结果之后执行。
    - 例如，页面渲染后。
    - **当然，要注意，无论调用 Controller 方法是否成功，都会执行**。
- 举个例子：
  - 当俩个拦截器都实现放行操作时，执行顺序为 `preHandle[1] => preHandle[2] => postHandle[2] => postHandle[1] => afterCompletion[2] => afterCompletion[1]` 。
  - 当第一个拦截器 `preHandle(...)` 方法返回 `false` ，也就是对其进行拦截时，第二个拦截器是完全不执行的，第一个拦截器只执行 `preHandle(...)` 部分。
  - 当第一个拦截器 `preHandle(...)` 方法返回 `true` ，第二个拦截器 `preHandle(...)` 返回 `false` ，执行顺序为 `preHandle[1] => preHandle[2] => afterCompletion[1]` 。
- 总结来说：
  - `preHandle(...)` 方法，按拦截器定义顺序调用。若任一拦截器返回 `false` ，则 Controller 方法不再调用。
  - `postHandle(...)` 和 `afterCompletion(...)` 方法，按拦截器定义逆序调用。
  - `postHandler(...)` 方法，在调用 Controller 方法之后执行。
  - `afterCompletion(...)` 方法，只有该拦截器在 `preHandle(...)` 方法返回 `true` 时，才能够被调用，且一定会被调用。为什么“且一定会被调用”呢？即使 `afterCompletion(...)` 方法，按拦截器定义**逆序**调用时，前面的拦截器发生异常，后面的拦截器还能够调用，即无视异常。

关于这块，可以看看如下两篇文章：

- [《Spring MVC 多个拦截器执行顺序及拦截器使用方法》](https://blog.csdn.net/amaxiaochen/article/details/77210880) 文章，通过**实践**更加理解。
- [《精尽 Spring MVC 源码分析 —— HandlerMapping 组件（二）之 HandlerInterceptor》](http://svip.iocoder.cn/Spring-MVC/HandlerMapping-2-HandlerInterceptor/) 文章，通过**源码**更加理解。

## Spring MVC 的拦截器可以做哪些事情？

拦截器能做的事情非常非常非常多，例如：

- 记录访问日志。
- 记录异常日志。
- 需要登陆的请求操作，拦截未登陆的用户。
- …

## Spring MVC 的拦截器和 Filter 过滤器有什么差别？

看了文章 [《过滤器( Filter )和拦截器( Interceptor )的区别》](https://blog.csdn.net/xiaodanjava/article/details/32125687) ，感觉对比的怪怪的。艿艿觉得主要几个点吧：

- **功能相同**：拦截器和 Filter都能实现相应的功能，谁也不比谁强。
- **容器不同**：拦截器构建在 Spring MVC 体系中；Filter 构建在 Servlet 容器之上。
- **使用便利性不同**：拦截器提供了三个方法，分别在不同的时机执行；过滤器仅提供一个方法，当然也能实现拦截器的执行时机的效果，就是麻烦一些。

另外，再补充一点小知识。我们会发现，拓展性好的框架，都会提供相应的拦截器或过滤器机制，方便的我们做一些拓展。例如：

- Dubbo 的 Filter 机制。
- Spring Cloud Gateway 的 Filter 机制。
- Struts2 的拦截器机制。

# 十一、REST

​	==REST 代表着抽象状态转移，它是根据 HTTP 协议从客户端发送数据到服务端，例如：服务端的一本书可以以 XML 或 JSON 格式传递到客户端。==

​	参照： [《怎样用通俗的语言解释 REST，以及 RESTful？》](https://www.zhihu.com/question/28557115) 

​	**资源是指数据在 REST 架构中如何显示的。将实体作为资源公开 ，它允许客户端通过 HTTP 方法如：[GET](http://javarevisited.blogspot.sg/2012/03/get-post-method-in-http-and-https.html), [POST](http://www.java67.com/2014/08/difference-between-post-and-get-request.html),[PUT](http://www.java67.com/2016/09/when-to-use-put-or-post-in-restful-web-services.html), DELETE 等读，写，修改和创建资源。**

## REST 操作

REST 接口是通过 HTTP 方法完成操作。

- 一些HTTP操作是安全的，如 GET 和 HEAD ，它不能在服务端修改资源
- 换句话说，PUT,POST 和 DELETE 是不安全的，因为他们能修改服务端的资源。

所以，是否安全的界限，在于**是否修改**服务端的资源。

## 幂等操作

​	**有一些HTTP方法，如：GET，不管你使用多少次它都能产生相同的结果，在没有任何一边影响的情况下，发送多个 GET 请求到相同的[URI](http://www.java67.com/2013/01/difference-between-url-uri-and-urn.html) 将会产生相同的响应结果。因此，这就是所谓幂等操作。**

​	换句话说，[POST方法不是幂等操作](http://javarevisited.blogspot.sg/2016/05/what-are-idempotent-and-safe-methods-of-HTTP-and-REST.html) ，因为如果发送多个 POST 请求，它将在服务端创建不同的资源。但是，假如你用PUT更新资源，它将是幂等操作。甚至多个 PUT 请求被用来更新服务端资源，将得到相同的结果。

​	[REST](http://javarevisited.blogspot.sg/2015/08/difference-between-soap-and-restfull-webservice-java.html) 是可扩展的和可协作的。它既不托管一种特定的技术选择，也不定在客户端或者服务端。你可以用 [Java](http://javarevisited.blogspot.sg/2017/11/top-5-free-java-courses-for-beginners.html), [C++](http://www.java67.com/2018/02/5-free-cpp-courses-to-learn-programming.html), [Python](http://www.java67.com/2018/02/5-free-python-online-courses-for-beginners.html), 或 [JavaScript](http://www.java67.com/2018/04/top-5-free-javascript-courses-to-learn.html) 来创建 RESTful Web 服务，也可以在客户端使用它们。

## REST 用哪种 HTTP 方法呢?

REST 能用任何的 HTTP 方法，但是，最受欢迎的是：

- 用 GET 来检索服务端资源
- 用 POST 来创建服务端资源
- [用 PUT 来更新服务端资源](http://javarevisited.blogspot.sg/2016/04/what-is-purpose-of-http-request-types-in-RESTful-web-service.html#axzz56WGunSwy)
- 用 DELETE 来删除服务端资源。

恰好，这四个操作，对上我们日常逻辑的 CRUD 操作。

## 删除的 HTTP 状态返回码

在删除成功之后，您的 REST API 应该返回什么状态代码，并没有严格的规则。它可以返回 200 或 204 没有内容。

- 一般来说，如果删除操作成功，响应主体为空，返回 [204](http://www.netingcn.com/http-status-204.html) 。
- 如果删除请求成功且响应体不是空的，则返回 200 。

## REST API 是无状态的吗?

​	REST API 应该是无状态的，因为它是基于 HTTP 的，它也是无状态的。

​	REST API 中的请求应该包含处理它所需的所有细节。它**不应该**依赖于以前或下一个请求或服务器端维护的一些数据，例如会话。

## RestTemplate 的优势是什么?

​	在 Spring Framework 中，RestTemplate 类是 [模板方法模式](http://www.java67.com/2012/09/top-10-java-design-pattern-interview-question-answer.html) 的实现。跟其他主流的模板类相似，如 JdbcTemplate 或 JmsTempalte ，它将在客户端简化跟 RESTful Web 服务的集成。正如在 RestTemplate 例子中显示的一样，你能非常容易地用它来调用 RESTful Web 服务。

> 一般使用 [OkHttp](http://square.github.io/okhttp/) 作为 HTTP 库，因为更好的性能，使用也便捷，并且无需依赖 Spring 库。

## HttpMessageConverter 在 Spring REST 中代表什么?

​	**HttpMessageConverter 是一种[策略接口](http://www.java67.com/2014/12/strategy-pattern-in-java-with-example.html) ，它指定了一个转换器，它可以转换 HTTP 请求和响应。Spring REST 用这个接口转换 HTTP 响应到多种格式，例如：JSON 或 XML 。**

​	每个 HttpMessageConverter 实现都有一种或几种相关联的MIME协议。Spring 使用 `"Accept"` 的标头来确定客户端所期待的内容类型。

​	然后，它将尝试找到一个注册的 HTTPMessageConverter ，它能够处理特定的内容类型，并使用它将响应转换成这种格式，然后再将其发送给客户端。

参照 [《Spring 中 HttpMessageConverter 详解》](https://leokongwq.github.io/2017/06/14/spring-MessageConverter.html) 。

## 如何创建 HttpMessageConverter 的自定义实现来支持一种新的请求/响应？

​	我们仅需要创建自定义的 AbstractHttpMessageConverter 的实现，并使用 WebMvcConfigurerAdapter 的 `extendMessageConverters(List<HttpMessageConverter<?>> converters)` 方法注中册它，该方法可以生成一种新的请求 / 响应类型。

具体的示例，可以参照 [《在 Spring 中集成 Fastjson》](https://github.com/alibaba/fastjson/wiki/%E5%9C%A8-Spring-%E4%B8%AD%E9%9B%86%E6%88%90-Fastjson) 文章。



























