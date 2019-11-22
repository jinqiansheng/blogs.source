---
title: Hystrix断路器
date: 2019-11-18 18:19:36
tags:
 - 微服务
 - SpringCloud
 - Java
---
用于服务熔断和服务降级，能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。  
当某个服务单元发生故障之后，通过断路器的故障监控，向调用方返回一个符合预期的，可处理的备选响应，而不是长时间的等待或抛出异常。  
#### 服务熔断
**pom加入**

``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```  

**主启动类**

``` java
@SpringBootApplication
@EnableEurekaClient//本服务启动后会自动注册进eureka服务中
@EnableDiscoveryClient//服务发现
@EnableCircuitBreaker//对hystrixR熔断机制的支持
public class DeptProvider8001_Hystrix_App {

	public static void main(String[] args) {
		SpringApplication.run(DeptProvider8001_Hystrix_App.class, args);
	}
}
```  

**调用**
``` java
@GetMapping("getByid/{id}")
@HystrixCommand(fallbackMethod = "processHystrix_Get")
//出现异常执行执行"processHystrix_Get"方法
public Dept get(@PathVariable("id") Integer id)
{
    Optional<Dept> op = deptList.stream().filter(e -> e.getDeptno() == id).findFirst();
    if(op.isPresent()) {
        return op.get();
    }else {
        throw new RuntimeException("该ID：" + id + "没有没有对应的信息");
    }
}
public Dept processHystrix_Get(@PathVariable("id") Integer id)
{
    return new Dept().setDeptno(id).setDname("该ID：" + id + "没有没有对应的信息,null--@HystrixCommand")
				.setDbSource("没得信息哇");
}
```  

以上处理方式属于简易型的服务熔断。实际项目中熔断和降级是共同处理，我的理解就是使用通过熔断的机制达到服务降级的目的。  

---   

#### 服务降级
服务的降级处理是在客户端完成，服务端不做处理
**yml加入**
``` yml
feign: 
  hystrix: 
    enabled: true
```  

**在@FeignClinent注解中加上fallbackFactory属性值**    

``` java
@FeignClient(value = "MICROSERVICECLOUD-DEPT",fallbackFactory=DeptClientServiceFallbackFactory.class)
@RequestMapping("/dept")
public interface DeptClientService {
	@GetMapping("/list")
	public List<Dept> list();
	
	@GetMapping("getByid/{id}")
	public Dept get(@PathVariable("id") Integer id);
}
```  

**实现fallbackFactory属性值中的类**  

``` java
@Component//必须加否则无效果
//实现FallbackFactory<T> 接口 T为service
public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService>{
	@Override
	public DeptClientService create(Throwable cause) {
		return new DeptClientService() {
			@Override
			public List<Dept> list() {
				return null;
			}
			@Override
			public Dept get(Integer id) {
				return new Dept().setDeptno(id).setDname("该ID：" + id + "没有没有对应的信息,Consumer客户端提供的降级信息,此刻服务Provider已经关闭");
			}
		};
	}
}
```  
---  

#### hystrixDashboard服务监控
Hystrix会通过hystrix-mettics-event-stream项目持续记录所有通过Hystrix发起的请求执行信息，并以报表的形式展示。  
**POM加入**  

``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
</dependency>
```  

**主启动类**  

``` java
@SpringBootApplication
@EnableHystrixDashboard
public class DeptConsumer_DashBoard_App
{
	public static void main(String[] args)
	{
		SpringApplication.run(DeptConsumer_DashBoard_App.class, args);
	}
}
``` 
---

**被监控的服务提供者pom需要加入actuator**
``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```  
输入hystrixDashboard微服务的地址查看如：http://localhost:9001/hystrix 查看图形化界面，再根据提示查看特定微服务的访问图形化统计  

**实心圆：** 通过颜色变化代表健康程度，分别从绿色<黄色<橙色<红色递减。同时流量越大实心圆越大  
**曲线：** 可以记录指定毫秒内流量的上升和下降趋势
