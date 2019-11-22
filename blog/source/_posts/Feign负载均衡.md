---
title: Feign负载均衡
date: 2019-11-17 18:19:36
tags:
 - 微服务
 - SpringCloud
 - Java
---
Feign是一个声明式的WebService客户端，使用Feign能让编写Web Service客户端更加简单。支持JAx-RS标准的注解，支持可拔插式的编码器和解码器，支持Spring MVC标准注解和HttpMessageConverters。可以与Eureka和Ribbon组合使用支持负载均衡。  
<font color=#FF0000 >简单来说就是Feign封装Ribbon，在实际使用一个借口会被多个消费方调用，使用Feign可以封装这些服务提供方的接口，只需要加上一个Feign注解即可，简化了使用ribbon时自动封装服务调用客户端的开发量 </font>  

#### 消费方客户端使用
``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```  

**主启动类示例**  

``` java
@SpringBootApplication(scanBasePackages = "com.jqs.springcloud")//启动时扫描哪些包
@EnableFeignClients(basePackages= {"com.jqs.springcloud"})//扫描所有使用注解@FeignClient定义的feign客户端
public class DeptConsumerFeign_App {
	public static void main(String[] args) {
		SpringApplication.run(DeptConsumerFeign_App .class, args);
	}
}
```  

#### 接口实现
接口实现服务提供者开放的接口

``` java
@FeignClient(value = "MICROSERVICECLOUD-DEPT")
@RequestMapping("/dept")
public interface DeptClientService {
	@GetMapping("/list")
	public List<Dept> list();
}
```  

**调用**  

调用时只需要调用@FeignClient注解的借口即可调用服务提供者的api，且默认的轮询负载均衡，如果有需要可以重写负载均衡算法。  
``` java
@Autowired
private DeptClientService deptClientService;
```  

如果报错误异常，提示找不到@FeignClient注解的vlaue的属性赋值的实例名，则需要在yml配置实例名所指向的实际服务地址(具体我也没找到原因，在debug的源码，发现@Autowired 带有@FeignClient的接口信息时 allServiceList为null。以后在研究)，如
``` yml
MICROSERVICECLOUD-DEPT: ##实例名
  ribbon:
    ## 服务提供者的地址，不是服务注册中心的地址
    listOfServers: http://localhost:8001,http://localhost:8002,http://localhost:8003

## 这个要有，如果不加，只加了上面也没用
ribbon:
  eureka:
    enabled: false
```  