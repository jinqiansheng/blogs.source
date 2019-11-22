---
title: Zuul路由网关
date: 2019-11-19 18:19:36
tags:
 - 微服务
 - SpringCloud
 - Java
---
提供代理、路由、过滤三大功能。  
路由：负责将外部请求转发的具体的微服务实例上，是实现外部统一访问入口的基础。  
过滤：对请求的处理过程进行干预，实现校验、服务聚合等功能。  
它将自身注册为Eureka服务治理下的应用，同时从Eureka中获取其他微服务的消息。  

#### 使用
**pom加入**
``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```  
**yml加入**
``` yml
zuul: 
  #ignored-services: microservicecloud-dept  #原真实服务名忽略(指定服务名","分隔)
  prefix: /jqs #统一的公共前缀
  ignored-services: "*" #忽略所有的真实服务名忽略
  routes: 
    mydept.serviceId: microservicecloud-dept #服务的实例名
	mydept.path: /mydept/** #映射后的访问名
```  
**主启动类**
``` java
@SpringBootApplication
@EnableZuulProxy
public class Zuul_9527_StartSpringCloudApp
{
	public static void main(String[] args)
	{
		SpringApplication.run(Zuul_9527_StartSpringCloudApp.class, args);
	}
}
```  