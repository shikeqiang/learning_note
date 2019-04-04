# DispatcherServlet源码解析

## 一、springMVC整体流程

先来看看springMVC的工作流程图：![image-20181219160754756](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20181219160754756-5206874.png)

下面来看看每一步都是在做什么：

第一步：用户发起request请求，请求至DispatcherServlet前端控制器。

第二步：DispatcherServlet前端控制器请求HandlerMapping处理器映射器查找Handler。

DispatcherServlet：前端控制器，相当于中央调度器，各各组件都和前端控制器进行交互，降低了各组件之间耦合度。

第三步：HandlerMapping处理器映射器，根据url及一些配置规则（xml配置、注解配置）查找Handler，将Handler返回给                          DispatcherServlet前端控制器。

第四步：DispatcherServlet前端控制器调用适配器执行Handler，有了适配器通过适配器去扩展对不同Handler执行方式（比如：                原始servlet开发，注解开发）

第五步：适配器执行Handler。Handler是后端控制器，当成模型。

第六步：Handler执行完成返回ModelAndView

ModelAndView是springmvc的一个对象，对Model和view进行封装。

第七步：适配器将ModelAndView返回给DispatcherServlet

第八步：DispatcherServlet调用视图解析器进行视图解析，解析后生成view。视图解析器根据逻辑视图名解析出真正的视图。

View：springmvc视图封装对象，提供了很多view，比如：jsp、freemarker、pdf、excel。

第九步：ViewResolver视图解析器给前端控制器返回view

第十步：DispatcherServlet调用view的渲染视图的方法，将模型数据填充到request域 。

第十一步：DispatcherServlet向用户响应结果(jsp页面、json数据。)

在这里面有几个组件要说一下：

DispatcherServlet：前端控制器，由springmvc提供
HandlerMappting：处理器映射器，由springmvc提供
HandlerAdapter：处理器适配器，由springmvc提供
Handler：处理器，需要程序员开发
ViewResolver：视图解析器，由springmvc提供
View：真正视图页面需要由程序编写
 由上面springMVC的工作流程图我们可以知道，其实springMVC的核心就是DispatcherServlet，所以接下来主要就是通过源码来看看DispatcherServlet是怎么工作的。

## 二、服务流程

一般而言，Servlet有一个服务方法 doService来为HTTP请求提供服务(如参照前面的servlet源码解析)， DispatcherServlet也是如此，它的 doService 方法如下面代码所示：

```Java
//将DispatcherServlet特定的请求转发给doDispatch处理
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
	if (logger.isDebugEnabled()) {
		String resumed = WebAsyncUtils.getAsyncManager(request).hasConcurrentResult() ? " resumed" : "";
		logger.debug("DispatcherServlet with name '" + getServletName() + "'" + resumed +
							" processing " + request.getMethod() + " request for [" + getRequestUri(request) + "]");
	}
	// 快照处理，使用快照可以更快响应用户请求
	Map<String, Object> attributesSnapshot = null;
	if (WebUtils.isIncludeRequest(request)) {
		attributesSnapshot = new HashMap<String, Object>();
		Enumeration<?> attrNames = request.getAttributeNames();
		while (attrNames.hasMoreElements()) {
			String attrName = (String) attrNames.nextElement();
			if (this.cleanupAfterInclude || attrName.startsWith("org.springframework.web.servlet")) {
				attributesSnapshot.put(attrName, request.getAttribute(attrName));
			}
		}
	}
	// 设置Web IOC容器
	request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
    //设置国际化
	request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
    //主题属性
	request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
    //主题源属性
	request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());
	FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
	if (inputFlashMap != null) {
		request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
	}
	request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
	request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
	try {
        //处理分发
		doDispatch(request, response);
	}
	finally {
		if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
			// Restore the original attribute snapshot, in case of an include.
			if (attributesSnapshot != null) {
				restoreAttributesAfterInclude(request, attributesSnapshot);
			}
		}
	}
}

```

由上面的代码和注释可以看到，doService先对HTTP请求进行一些处理，然后设置一些属性，最后所有的流程都集中在了 doDispatch 方法中。所以接下来我们继续看doDispatch方法：

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;
		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
		try {
            //模型和视图
			ModelAndView mv = null;
            //错误处理
			Exception dispatchException = null;
			try {
                //文件上传处理解析器
				processedRequest = checkMultipart(request);
                //是否为文件上传请求
				multipartRequestParsed = (processedRequest != request);
				// 获得匹配的执行链，决定处理请求的handler
				mappedHandler = getHandler(processedRequest);
                //没有处理器错误
				if (mappedHandler == null || mappedHandler.getHandler() == null) {
					noHandlerFound(processedRequest, response);
					return;
				}
 
				// 找到对应的处理器适配器(HandlerAdapter)
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
 
				// Process last-modified header, if supported by the handler.
                ///判断是 http 的 get 方法还是 post 方法或其他
				String method = request.getMethod();
                //如果是GET方法的处理
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (logger.isDebugEnabled()) {
						logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
					}
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}
                //执行拦截器的事前方法，如果返回 false，则流程结束
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}
 
				// Actually invoke the handler.
                //执行处理器，返回 ModelAndView
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
 
				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}
                //如果视图为空，给予设置默认视图的名称
				applyDefaultViewName(processedRequest, mv);
                //执行处理器拦截器的事后方法
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
                //记录异常
				dispatchException = ex;
			}
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Throwable ex) {
            //记录异常
			 dispatchException = new NestedServletException(”Handler dispatch failed ”, err) ;
		}
        //处理请求结果，显然这里已经通过处理器得到最后的结果和视图 
        //如果是逻辑视图，则解析名称，否则就不解析，最后渲染视图
            processDispatchResult(processedRequest, response,mappedHandler, mv, dispatchException);
        }
        catch (Exception ex) {
            //异常处理，拦截器完成方法
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
            //错误处理
			triggerAfterCompletionWithError(processedRequest, response, mappedHandler, new NestedServletException (”Handler processing failed ",err)) ;
		}
		finally {
            //处理资源的释放
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					//拦截器完成方法mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
                //文件请求资源释放
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}
```

通过源码可以看出其流程是 :
(l)通过请求找到对应的执行链，执行链包含了拦截器和开发者控制器。

(2)通过处理器找到对应的适配器。

(3)执行拦截器的事前方法，如果返回 false，则流程结束，不再处理 。

(4)通过适配器运行处理器，然后返回模型和视图。

(5)如果视图没有名称 ，则 给出默认的视图名称。

(6)执行拦截器的事后方法。

(7)处理分发请求得到的数据模型和视图的渲染。

## 三、处理器和执行器

在上面，我们已经了解了一个请求到达springMVC后，DispatcherServlet是怎么处理这个请求的，接下来就是看看处理器和拦截器，即DispatcherServlet的getHandler方法：

```java
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
	for (HandlerMapping hm : this.handlerMappings) {
			if (logger.isTraceEnabled()) {
				logger.trace(
						"Testing handler map [" + hm + "] in DispatcherServlet with name '" + getServletName() + "'");
			}
			HandlerExecutionChain handler = hm.getHandler(request);
			if (handler != null) {
				return handler;
			}
		}
		return null;
	}
```

在启动期间， SpringMVC已经初始化了处理器映射--HandlerMappings，所以这里先根据请求找到对应的 HandlerMapping，找到 HandlerMapping 后，就把相关处理的内容转化成一个 HandlerExecutionChain 对象。

接下来就来看看HandlerExecutionChain的结构是什么样的：

```java
public class HandlerExecutionChain {
 
	private static final Log logger = LogFactory.getLog(HandlerExecutionChain.class);
    //处理器，自己写的控制器(或其方法)
	private final Object handler;
    //拦截器数组
	private HandlerInterceptor[] interceptors;
    //拦截器列表
	private List<HandlerInterceptor> interceptorList;
    //当前拦截器下标，当使用数组时有效
	private int interceptorIndex = -1;
    ...
}
```

由源码可以看出，主要是定义一些拦截器相关的属性，可以在进入控制器之前 ， 运行对应的拦截器，这样就可以在进入处理器前做一些逻辑了。如果在做电商系统的时候，用户可以再未登录状态浏览商品，并加入购物车，但是如果要结算的时候，这个时候就要拦截请求，要求用户登录。这个时候就可以通过拦截器实现。接下来就来看看拦截器Handlerlnterceptor的相关方法：

```java
public interface HandlerInterceptor {
    boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
	    throws Exception;
    void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
			throws Exception;
    void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception;
```

依据 doDispatch 方法，当preHandle方法返回为 true 的时候才会继续运行，否则就会结束流程，而在数据模型渲染视图之前调用 postHandle方法， 在流程的 finally语 句中还会调用 afterCompletion方法， 这就是处理器拦截器的内容。如果要自己做一个拦截器的话，可以实现Handlerlnterceptor接口，然后重写这三个方法，你可以在这三个方法里面加入相应的业务逻辑进行处理。

## 四、视图渲染

在第二点的时候，我们知道processDispatchResult对模型和视图的处理，接下来我们来看看SpringMVC是怎么实现视图渲染。

```java
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
			HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {
 
		boolean errorView = false;
        //处理器发生异常
		if (exception != null) {
            //视图和模型定义方面的异常
			if (exception instanceof ModelAndViewDefiningException) {
				logger.debug("ModelAndViewDefiningException encountered", exception);
				mv = ((ModelAndViewDefiningException) exception).getModelAndView();
			}
            //处理器的异常
			else {
				Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
				mv = processHandlerException(request, response, handler, exception);
				errorView = (mv != null);
			}
		}
 
		//处理器是否返回视图和模型
		if (mv != null && !mv.wasCleared()) {
            //渲染视图
			render(mv, request, response);
			if (errorView) {
				WebUtils.clearErrorRequestAttributes(request);
			}
		}
		else {
            //视图为 null，或者已经被处理过
			if (logger.isDebugEnabled()) {
				logger.debug("Null ModelAndView returned to DispatcherServlet with name '" + getServletName() +
						"': assuming HandlerAdapter completed request handling");
			}
		}
        //是否存在并发处理
		if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
			// Concurrent handling started during a forward
			return;
		}
        //完成方法
		if (mappedHandler != null) {
			mappedHandler.triggerAfterCompletion(request, response, null);
		}
	}
```

由上面的源码可以看出，首先是先判断是否异常，如果有异常就再进一步处理是什么异常，如果没有异常就通过render方法进行进一步处理，相关逻辑也在注释中表明了，接下来就来看看render这个方法：

```java
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
		// Determine locale for request and apply it to the response.
//国际化
		Locale locale = this.localeResolver.resolveLocale(request);
		response.setLocale(locale);
		View view;
//是否是逻辑视图，如果是需要转化为真实路径下的视图
		if (mv.isReference()) {
			// We need to resolve the view name.
//转化逻辑视图为真实视图
			view = resolveViewName(mv.getViewName(), mv.getModelInternal(), locale, request);
			if (view == null) {
				throw new ServletException("Could not resolve view with name '" + mv.getViewName() +
						"' in servlet with name '" + getServletName() + "'");
			}
		}
//非逻辑视图
		else {
			// No need to lookup: the ModelAndView object contains the actual View object.
			view = mv.getView();
			if (view == null) {
				throw new ServletException("ModelAndView [" + mv + "] neither contains a view name nor a " +
						"View object in servlet with name '" + getServletName() + "'");
			}
		}
 
		// Delegate to the View object for rendering.
		if (logger.isDebugEnabled()) {
			logger.debug("Rendering view [" + view + "] in DispatcherServlet with name '" + getServletName() + "'");
		}
		try {
//渲染视图， view 为视圈， mv.getModelInternal ()为数据模型
			view.render(mv.getModelInternal(), request, response);
		}
		catch (Exception ex) {
			if (logger.isDebugEnabled()) {
				logger.debug("Error rendering view [" + view + "] in DispatcherServlet with name '" +
						getServletName() + "'", ex);
			}
			throw ex;
		}
	}
```

首先是国际化的设置，然后判断是否是逻辑视图，如果是就转化它为 真实视图，或者 直接获取视图即可，最后进入视图的 render方法，将模型和视图中的数据模型传递到该方 法中去，这样就完成了视图的渲染，展示给用户。