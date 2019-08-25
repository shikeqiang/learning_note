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













































参照：芋道源码