# 一、SpringCloud核心功能

- Distributed/versioned configuration 分布式/版本化的配置管理
- Service registration and discovery 服务注册与服务发现
- Routing 路由
- Service-to-service calls 端到端的调用
- Load balancing 负载均衡
- Circuit Breakers 断路器
- Global locks 全局锁
- Leadership election and cluster state 选举与集群状态管理
- Distributed messaging 分布式消息

相关组件：

![Spring Cloudç ç"ä"¶](/Users/jack/Desktop/md/images/4935fcc0a209fd1d4b70cade94986f59.jpeg)

其他等同的组件：

| Netflix  | 阿里    | 其它        |                                                              |
| -------- | ------- | ----------- | ------------------------------------------------------------ |
| 注册中心 | Eureka  | Nacos       | Zookeeper、Consul、Etcd                                      |
| 熔断器   | Hystrix | Sentinel    | Resilience4j                                                 |
| 网关     | Zuul1   | 暂无        | Spring Cloud Gateway                                         |
| 负载均衡 | Ribbon  | Dubbo(未来) | [`spring-cloud-loadbalancer`](https://github.com/spring-cloud/spring-cloud-commons/tree/master/spring-cloud-loadbalancer) |

# 二、微服务

[什么是微服务](https://blog.csdn.net/striveb/article/details/101106443)

## 微服务的优缺点

**1）优点**

- 每一个服务足够内聚,代码容易理解
- 开发效率提高，一个服务只做一件事
- 微服务能够被小团队单独开发
- 微服务是松耦合的，是有功能意义的服务
- 可以用不同的语言开发,面向接口编程
- 易于与第三方集成
- 微服务只是业务逻辑的代码，不会和 HTML、CSS 或者其他界面组合
  - 开发中，两种开发模式
    - 前后端分离
    - 全栈工程师
- 可以灵活搭配,连接公共库/连接独立库

**2）缺点**

- 分布式系统的负责性
- 多服务运维难度，随着服务的增加，运维的压力也在增大
- 系统部署依赖
- 服务间通信成本
- 数据一致性
- 系统集成测试
- 性能监控

## 注册中心

在 Spring Cloud 中，能够使用的注册中心，还是比较多的，如下：

- [`spring-cloud-netflix-eureka-server`](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-server) 和 [`spring-cloud-netflix-eureka-client`](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-server) ，基于 Eureka 实现。
- [`spring-cloud-alibaba-nacos-discovery`](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-nacos-discovery) ，基于 Nacos 实现。
- [`spring-cloud-zookeeper-discovery`](https://github.com/spring-cloud/spring-cloud-zookeeper/tree/master/spring-cloud-zookeeper-discovery) ，基于 Zookeeper 实现。
- … 等等

以上的实现，都是基于 [`spring-cloud-commons`](https://github.com/spring-cloud/spring-cloud-commons) 的 [`discovery`](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-commons/src/main/java/org/springframework/cloud/client/discovery/) 的 [DiscoveryClient](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-commons/src/main/java/org/springframework/cloud/client/discovery/DiscoveryClient.java) 接口，实现统一的客户端的注册发现。

## 服务发现

​	==简单来说，通过注册中心，调用方(Consumer)获得服务方(Provider)的地址，从而能够调用。==

当然，实际情况下，会分成两种注册中心的发现模式：

1. 客户端发现模式
2. 服务端发现模式

> 在 Spring Cloud 中，我们使用前者，即客户端发现模式。

​	在微服务应用中，服务实例的运行环境会动态变化，实例网络地址也是如此。因此，客户端为了访问服务必须使用服务发现机制。

​	**服务注册表是服务发现的关键部分。服务注册表是可用服务实例的数据库，提供管理 API 和查询 API。服务实例使用管理 API 来实现注册和注销，系统组件使用查询 API 来发现可用的服务实例。**

​	服务发现有两种主要模式：客户端发现和服务端发现。==在使用客户端服务发现的系统中，客户端查询服务注册表，选择可用的服务实例，然后发出请求。在使用服务端发现的系统中，客户端通过路由转发请求，路由器查询服务注册表并转发请求到可用的实例。==

​	服务实例的注册和注销也有两种方式。一种是服务实例自己注册到服务注册表中，即自注册模式；另一种则是由其它系统组件处理注册和注销，也就是第三方注册模式。

​	在一些部署环境中，需要使用 Netflix Eureka、etcd、Apache Zookeeper 等服务发现来设置自己的服务发现基础设施。而另一些部署环境则内置了服务发现。例如，Kubernetes 和 Marathon 处理服务实例的注册和注销，它们也在每个集群主机上运行代理，这个代理具有服务端发现路由的功能。

​	HTTP 反向代理和 NGINX 这样的负载均衡器能够用做服务器端的服务发现均衡器。服务注册表能够将路由信息推送到 NGINX，激活配置更新，譬如使用 Cosul Template。NGINX Plus 支持额外的动态配置机制，能够通过 DNS 从注册表中获取服务实例的信息，并为远程配置提供 API。

> 参照： [《为什么要使用服务发现》](https://blog.csdn.net/u013035373/article/details/79414529) 。







































参照：[芋道源码](http://svip.iocoder.cn/Spring-Cloud/Interview/)