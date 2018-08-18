---
title: 'Head Frist Spring Cloud'
date: 2018-02-06 20:32:20
tags: Spring
---
### 服务提供者和服务消费者
- 服务提供者：对应的进行相关数据库操作并向外界提供API接口
- 服务消费者：对应的消费服务提供者提供的API，以实现相应的需求。
- **关键实现：使用RestTemplate作为客户端向相应API发出请求**
    - [RestTemplate详解-参考博文](https://www.cnblogs.com/caolei1108/p/6169950.html)

##### 存在的问题：
- 1、API硬编码问题
- 2、服务发现与注册
    - 服务消费者和服务提供者均需要在服务发现组件中进行注册。
    - 服务发现组件的功能：服务注册表（数据库）的相关维护，服务注册，健康检查（心跳机制）
- 3、服务发现的方式
    - 客户端发现：Eureka 和 Zk
    - 服务端发现： Consul + nginx

### 服务发现和注册
#### Eureka 服务发现 @EnableEurekaServer
- [参考博客：springcloud(二)：注册中心Eureka](http://www.ityouknow.com/springcloud/2017/05/10/springcloud-eureka.html)
- [Spring cloud Eureka 参数配置](https://segmentfault.com/a/1190000008378268)
- **Eureka** 既可以作为 **server**,又可以作为 **client**
- [简单配置 单机系统 eureka-server](https://github.com/zjs1224522500/spring-cloud-examples/blob/master/spring-cloud-eureka/spring-cloud-eureka/src/main/resources/application.properties)
- [高可用配置 集群 多节点 peer1 2 3](https://github.com/zjs1224522500/spring-cloud-examples/blob/master/spring-cloud-eureka/spring-cloud-eureka-cluster/src/main/resources/application.yml)



### 两种服务调用方式
##### Ribbon + RestTemplate
- Ribbon 是一个负载均衡客户端，很好地控制http和tcp的行为
- 对**RestTemplate**使用 **@LoadBalanced**注解，实现负载均衡。
- 可以使用 **@RibbonClient**注解来指定相关配置，并编写配置类自定义负载均衡策略。（通过代码的方式自定义配置）
- 也可以使用配置文件进行负载均衡的相关配置。
- **LoadBalancerClient**可以获取相关客户端信息。

##### Feign
- Feign 是一个声明式的伪Http客户端，它使得写Http客户端变得更简单。使用Feign，只需要创建一个接口并注解。它具有可插拔的注解特性，可使用Feign 注解和JAX-RS注解。Feign支持可插拔的编码器和解码器。Feign默认集成了Ribbon，并和Eureka结合，默认实现了负载均衡的效果。
- **@FeignClients**表明客户端继承了 **Feign**
- 使用 **@FeignClients("spring-application-name")**修饰自定义的**xxxFeignClient**类,又因为SpringCloud集成了SpringMVC的相关特性，默认使用了mvc的**contract**。故可以在接口中定义的方法上使用 mvc 相关注解修饰并进行 url 以及 method的匹配从而模拟出相应的http请求。
- 再在消费者的**Controller**中注入对应的**xxxFeignCLient**，在对应的方法中调用client的相关模拟请求实现需求。
- 对于**FeignClient**也可以使用自定义配置，通过在注解中指定configuration，并编写相关 **Config**类来实现。config类使用 **@Configuration**注解。



### Feign 和 Ribbon 的常见问题：
#### Hystrix相关：
##### Feign和Ribbon在整合了Hystrix后，可能会出现首次调用失败的问题
- 原因： Hystrix默认的超时时间是1秒，如果超过这个时间尚未响应，将会进入fallback代码。而首次请求往往会比较慢（因为Spring的懒加载机制，要实例化一些类），这个响应时间可能就大于1秒了。
- **解决方案**：在配置文件中进行相关配置：
    - hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 5000
    - hystrix.command.default.execution.timeout.enabled: false
    - feign.hystrix.enabled: false