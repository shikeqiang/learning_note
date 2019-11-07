# 一、简介

​	Hessian和Burlap都是基于HTTP的，他们都解决了RMI所头疼的防火墙渗透问题。但当传递过来的RPC消息中包含序列化对象时，RMI就完胜Hessian和Burlap了。 因为Hessian和Burlap都是采用了私有的序列化机制，而RMI使用的是Java本身的序列化机制。如果数据模型非常复杂，那么Hessian/Burlap的序列化模型可能就无法胜任了。 Spring开发团队意识到RMI服务和基于HTTP的服务之前的空白，Spring的HttpInvoker应运而生。

​	HttpInvoker是常用的[Java](http://lib.csdn.net/base/java)同构系统之间方法调用实现方案，是众多Spring项目中的一个子项目。顾名思义，它通过HTTP通信即可实现两个Java系统之间的远程方法调用，使得系统之间的通信如同调用本地方法一般。

​	Spring HTTP invoker 是 spring 框架中的一个远程调用模型，**执行基于 HTTP 的远程调用（意味着可以通过防火墙）**，并使用 java 的序列化机制在网络间传递对象。**这需要在远端和本地都使用Spring才行**。客户端可以很轻松的像调用本地对象一样调用远程服务器上的对象，这有点类似于 ‍webservice ‍，但又不同于 ‍webservice ‍，区别如下：‍

![image-20190709154853453](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20190709154853453.png)

## 服务器端实现

服务端主入口由`HttpInvokerServiceExporter`实现，它的工作大致流程如下 ：

![æå¡ç"¯å¤çæµç¨](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/SouthEast-2662920.png)

​	HttpInvokerServiceExporter实现了HttpRequestHandler，这使得其拥有处理HTTP请求的能力，按照Spring MVC的架构，它将被注册到HandlerMapping的BeanNameMapping中，这设计到Spring MVC如何处理请求，可以关注我的相关文章。 

​	服务端的重要任务就是读取并解析RemoteInvocation，再返回RemoteInvocationResult，剩下的都只是标准IO流的读写。

## 客户端实现

​	客户端的实现也很好理解，主入口为HttpInvokerProxyFactoryBean, 和Spring用到的众多设计相同，该类的结构使用了模板设计方法，该类提供实现了几个模板方法，整体逻辑由父类HttpInvokerClientInterceptor的实现，主要流程如下：

![å®¢æ·ç"¯å¤çæµç¨](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/SouthEast-20190709170613981.png)

​	我们最关心的是当我们调用接口的方法时，HttpInvoker是如何做到调用到远方系统的方法的，其实HttpInvokerProxyFactoryBean最后返回的是一个代理类（Cglib Proxy或者Jdk Proxy），我们调用接口的任何方法时，都会先执行HttpInvokerClientInterceptor的invoke()方法。

# 二、使用指南

## 服务器端

- 先创建一个实现Serializable接口的实体类，创建接口及实现类；
- 创建服务器端配置文件，通过SimpleUrlHandlerMapping为注册的服务提供URL映射，也就是发布服务

## 配置客户端

- 创建客户端配置文件，在这个配置文件中配置服务器端的路径

  > 注意：
  >
  > 1. Server端与Client端的DTO的包名不同导致反序列化失败。 接口名字和bean名字保持一致 
  > 2. 如果包名或者字段名不同，则会被认为是不同的对象，会反序列化失败，调用也就出错了 

- 可以将接口注册到Spring容器中(通过配置bean标签实现)，并在配置文件中映射接口为本地服务,这样外部就可以通过http接口调用这个接口的实现了

















参照：

<https://blog.csdn.net/zl_momomo/article/list/4?>

<https://blog.csdn.net/xiangxizhishi/article/details/74105477>













