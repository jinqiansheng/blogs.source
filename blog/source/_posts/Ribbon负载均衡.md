---
title: Ribbon负载均衡
date: 2019-11-16 18:19:36
tags:
 - 微服务
 - SpringCloud
 - Java
---
Spring Cloud Ribbon是基于Netflix Ribbon实现的一套<font color=#FF0000 >客户端负载均衡工具。  </font>  
主要提供客户端的软件负载均衡算法，将Netflix的中间层服务连接在一起，提供一系列完善的配置项，如连接超时、重试等。  
简单来说就是在配置文件中列出Load Balancer(简称LB)后面所有的极其，Ribbon自动帮你基于某种规则去连接，并可以自定义负载均衡算法。

#### 使用
Ribbon是一套客户端的负载均衡工具，需要在消费方使用
**pom引入相关jar包**  

``` xml
<!-- Ribbon相关 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```  

**yml加入**

``` yml
eureka:
  client:
    register-with-eureka: false #不注册自身
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/  #eureka集群
```  

**主启动类**

``` java
@SpringBootApplication
@EnableEurekaClient
public class Consumer_App
{
    public static void main(String[] args)
    {
        SpringApplication.run(Consumer_App.class, args);
    }
}
```  
---
在ConfigBean声明的Bean  
RestTemplate需要加入@LoadBalanced注解，restTemplate就可以通过微服务名进行访问,默认采用轮询的负载均衡算法
``` java
@Configuration
public class ConfigBean {
	@Bean
	@LoadBalanced
	public RestTemplate getRestTemplate() {
		return new RestTemplate();
	}
     /**
     *定义负载均衡算法，不写默认为轮询
     **/
     @Bean
    public IRule myRule()
    {
        return new RandomRule();
    }
}
```  
---
#### Ribbon核心组件IRule
默认的7种负载均衡的算法  

|名称|说明|
|:---|:---|
|RoundRobinRule|轮询算法（是Ribbon默认的负载均衡机制）|  
|RandomRule|随机访问算法|  
|RetryRule|先按照 RoundRobinRule 的策略访问服务，如果访问的服务宕机或者出现异常的情况，则在指定时间内会进行重试，访问其它可用的服务|  
|BestAvailableRule|首先会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务访问|  
|ZoneAvoidanceRule|默认规则,复合判断server所在区域的性能和server的可用性选择服务器|  
|AvailabilityFilteringRule|首先会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，还有并发的连接数量超过阈值的服务，然后对剩余的服务列表按照轮询策略进行访问|  
|WeightedResponseTimeRule|根据平均响应时间计算所有服务的权重，响应时间越快服务权重越大被选中的概率越高。刚启动时如果统计信息不足，则使用RoundRobinRule策略，等统计信息足够，会切换到WeightedResponseTimeRule|  

---

#### Ribbon自定义负载均衡算法
使用自定义Ribbon 需要再主启动类上加入@RibbonClient  
<font color=#FF0000>注意：自定义的负载均衡算法配置类不能放在@ComponentScan所扫描的当前包下以及包下，
否则自定义的这个配置类就会被所有的Ribbon客户端共享。</font>    

**主启动加入@RibbonClient注解**  

``` java
@SpringBootApplication
@EnableEurekaClient
//在启动该微服务的时候就能去加载我们的自定义Ribbon配置类，从而使配置生效
//name = 微服务名
//configuration = 自定义负载均衡算法的类
@RibbonClient(name="MICROSERVICECLOUD-DEPT",configuration=MySelfRule.class)
public class Consumer_App
{
    public static void main(String[] args)
    {
        SpringApplication.run(Consumer_App.class, args);
    }
}
```  

自定义写法与调用Ribbon自带的7种类似，但需要自己写类去实现AbstractLoadBalancerRule接口，具体写法可参照自带的7种负载均衡算法去实现（比如参照轮询或者随机这种比较简单的负载均衡算法的源码去实现自己想要的）。  

``` java
@Configuration
public class MySelfRule {

	@Bean
	public IRule myRule()
	{
		return new RoundRobinRule_T();//自定义负载均衡
	}
}
```  
