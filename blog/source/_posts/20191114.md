---
title: SpringCloud概述
date: 2019-11-14 18:19:36
tags:
 - 微服务
 - SpringCloud
 - Java
---
分布式微服务架构下的一站式解决方案，是各个微服务架构落地技术的几何体，俗称微服务全家桶。

#### SpringCloud和SrpingBoot的关系

1. SpringBoot专注于快速方便开发单个个体微服务
2. SpringCloud是关注全局的微服务协调治理框架，它将SpringBoot开发的一个个单体微服务整合并管理起来，为各个微服务之间提供配置管理、服务发现、断路器、路由、微代理、实践总线、全局锁、决策竞选、分布式会话等等集成服务
3. SpringBoot可以离开SpringCloud独立使用开发项目，但是SpringCloud离不开SpringBoot，属于依赖关系

#### Dubbo和SpringCloud的区别：

**技术栈对比**

| 技术栈    | Dubbo             | SpringCloud                  |
|:---:| :---: | :---: |
| 服务注册中心 | Zookeeper(第三方) | Srping Cloud Netflix Eureka  |
| 服务调用方式 | RPC               | Rest API                     |
| 服务监控 | Dubbo-monitor     | Spring Boot Admin            |
| 断路器    | 不完善         | Spring Cloud Netflix Hystrix |
| 服务网关 | 无               | Spring Cloud Netflix Zuul    |
| 分布式配置 | 无               | Spring Cloud Config          |
| 服务跟踪 | 无               | Spring Cloud Sleuth          |
| 消息总线 | 无               | Spring Cloud Bus             |
| 数据流    | 无               | Spring Cloud Stream          |
| 批量任务 | 无               | Spring Cloud Task            |
| ……       |                   |                              |

**区别：**
1. SpringCloud抛弃了Dubbo的RPC通信，采用的是基于HTTP的Rest方式，严格来说，这两种方式各有优劣。虽然一定程度上来说，后者牺牲了服务调用的性能，但也避免了上面提到的原生RPC带来的问题。而且Rest相比RPC更为灵活，服务提供方和调用方的依赖只依靠一纸契约，不存在代码级别的强依赖，这在强调快速烟花的微服务环境下，显得更加灵活。
2. SpringCloud提供了分布式的一站式服务对自身各个组件有大量的兼容测试，而Dubbo构建的微服务架构，自由度很高，但是需要开发人员对各组件的基础有足够的了解，从构建项目的成本来讲SpringCloud要优于Dubbo。
3. 社区支持和更新力度，Dubbo停止更新了5年左右，SpringCloud是Spring的拳头项目，更新力度较高，社区也相对活跃，学习成本低于Dubbo。

 > **HTTP：** 是指从客户端到服务端的请求消息，请求是使用具有标准定义的通用接口定向到资源的，这些语义能够被中间件和提供服务的来源机器进行解释。这样会使一个应用支撑分层的转换和间接层，并且独立于消息的来源，这对于一个Internet规模，多个组织，无法控制的可伸缩的信息系统来说，是非常有用的。主要用于对外的异构环境，浏览器接口调用，app接口调用，第三方接口调用等
 > 
 > **PRC：** 是远程过程调用协议，根据语言的API来定义的，而不是根据基于网络的应用来定义的。主要用于公司内  
> 
 > |&emsp;&emsp;&emsp;&emsp;|HTTP|RPC|
 > |:---:|:---|:---|
 > | 传输协议 | 基于HTTP协议 | 可以基于TCP协议，也可以基于HTTP协议；|
 > | 传输效率 | 如果是基于HTTP1.1的协议，请求中会包含很多无用的内容，如果是基于HTTP2.0的协议，那么简单的封装一下是可以作为一个RPC来使用的，这时标准RPC框架更多的是服务治理。 | 使用自定义的TCP协议，可以让请求报文体积更小，或者使用HTTP2协议，也可以很好的减少报文的体积，提高传输效率。 |
 > | 性能消耗 | 大部分是通过json来实现的，字节大小和序列化耗时都比thrift要更消耗性能 | 可以基于thrift实现高效的二进制传输 |
 > | 负载均衡 | 需要配置Nginx，HAProxy来实现 | 基本都自带负载均衡策略 |
 > | 服务治理 | 需要事先通知，修改Nginx，HAProxy配置  | 能做到自动通知，不影响上游 |






#### 开发参考：
中文API：[https://www.springcloud.cc/spring-cloud-dalston.html](https://www.springcloud.cc/spring-cloud-dalston.html)  
中文社区：[https://www.springcloud.cn](https://www.springcloud.cn)  
中文网：[https://www.springcloud.cc](https://www.springcloud.cc)
