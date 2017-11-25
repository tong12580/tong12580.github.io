title: springboot填坑之 -- spring cloud基于ip的discovery服务注册中心配置
thumbnail: https://jokers-1252021562.cosgz.myqcloud.com/images/technology/timg.jpeg
date: 2017/07/29 20:07:01
tags: 
    - springBoot
    - springCloud
    - discovery
categories:
    - spring
---

## spring cloud基于ip的discovery服务注册中心配置

`SpringBoot`  `springCloud`

### 简述

 > Spring Cloud为开发人员提供了快速构建分布式系统中一些常见模式的工具（例如配置管理，服务发现，断路器，智能路由，微代理，控制总线，一次性令牌，全局锁，领导选举，分布式会话，集群状态）。分布式系统的协调导致锅炉板模式，并且使用Spring Cloud开发人员可以快速站起来实现这些模式的服务和应用程序。他们将在任何分布式环境中运行良好，包括开发人员自己的笔记本电脑，裸机数据中心，以及Cloud Foundry等托管平台。-- [引用自springCloud官网][1] 

----

spring cloud 的注册中心的配置(包括其他的微服务的配置) 都是基于host进行配置的 会产生极大的不变 -- 特别是在基于不同网段, 不同网关的 docker 容器之中 host的地址可能会不断变化,且 host不如ip容易维护.

综上所述: 本文将 聊一聊基于ip的注册中心的配置;

### 代码示例

----

```
server:
  port: 8761
eureka:
  instance:
    lease-expiration-duration-in-seconds: 30 //1
    lease-renewal-interval-in-seconds: 10 //2
    prefer-ip-address: true //3
    ip-address: 114.114.114.xxx //4
  server:
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 1200000
  client:
    register-with-eureka: false //5
    fetch-registry: false //6
    service-url:
      defaultZone: http://${eureka.instance.ip-address}:${server.port}/eureka/  //7
```

----

### 现在 来说明下:

> * 首先  1, 2  用来检测 服务是否存在 是否过期的
> 
>  (1) 指注册中心在接收到最后一个心跳之后等待的时间（秒），然后才能从此视图中删除此实例，并禁止此实例的流量。 
>  (2) 指注册的客户端服务需要向注册中心发送心跳以指示它仍然存在的频率（以秒为单位）。如果在leaseExpirationDurationInSeconds中指定的时间段内未收到心跳线，则eureka服务器将从其视图中删除该实例，因此不允许此实例的流量。
>
> * 其次  (3)用来 开启 是否使用ip识别服务 当该变量为 ```true``` 时 此时 将会使用您的 内网ip去注册服务, 当您的服务注册中心是基于内网的服务,那么 配置服务到这一步即可.但是如果您的各项自服务,不再同一个网段中时, 请继续配置(4)

> * 当您配置 (4) 时 即为手动配置ip地址注册服务, 此时 [3] 的配置将无效, 如果ip为指定注册中心所在的公网ip,那么 该注册中心将在公网可见.

> *  当为单注册中心时, 做为注册中心,本服务当然不能注册自己 (5) ,(6) 配置为 false 服务才可以正常启动,当为多注册中心时(5) (6) 可不配置,但是(7)必须配置为***非己*** 的url地址


----

[1]: http://projects.spring.io/spring-cloud/
