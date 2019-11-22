---
title: Eureka服务注册于发现
date: 2019-11-15 18:19:36
tags:
 - 微服务
 - SpringCloud
 - Java
---
<font color=#FF0000 >Eureka用于实现服务注册和发现，</font>是Netflix的一个子模块，也是核心模块之一。Eureka是一个极具Rest的服务，用于定位服务，以实现云端中间层服务发现和故障转移。服务注册于发现对于微服务架构来说是非常重要的，有了服务发现与注册，只需要使用服务器的标识符，就可以访问到服务，而不需要修改服务调用的配置文件。功能类似于dubbo的注册中心，比如Zookeeper。</br>
Eureka采用了C-S的设计架构，Eureka Server 作为服务注册功能的服务器，是服务的注册中心，而系统中的其他微服务，使用Eureka的客户端连接到Eureka Server并维持心跳连接。这样系统的维护人员就可以通过Eureka Server来监控系统中各个微服务是否正常运行。SpringCloud的一些其他的模块(比如Zuul)就可以通过Eureka Server来发现系统中的其他微服务，并执行相关逻辑。</br>
**<font color=#FF0000 >简单来说就是负责发现和注册微服务，并通过Eureka Server维持各个微服务之间的心跳连接。</font>**

#### 两大组件：
**Eureka Server：**
提供注册服务,各个节点启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所以可用服务节点的信息，服务节点的信息可以在界面中直观的看到。
  
**Eureka Client：** 是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置、使用轮询负载算法的负载均衡器。再应用启动后，将会向Eureka Server发送心跳（默认周期是30秒）。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）

#### Eureka对比Zookeeper的优点：
在CAP的理论中,一个分布式系统不能同时满足C（一致性）、A（可用性）、P（分区容错性）。由于分区容错性P在分布式系统中必须要保证，因此只能在A和C之间进行权衡。

> CAP原则又称CAP定理，指的是在一个分布式系统中， Consistency（一致性）、 		Availability（可用性）、Partition tolerance（分区容错性），三者不可得兼。
> 
> **一致性（C）：**
> 在分布式系统中的所有数据备份，在同一时刻是否同样的值。（等同于所有节点访问同一份最新的数据副本）
> 
> **可用性（A）：**
> 在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。（对数据更新具备高可用性）
> 
> **分区容忍性（P）：**
> 以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。
> 
>  ![ACP](https://s2.ax1x.com/2019/11/22/MoWOII.png)
> CAP原则的精髓就是要么AP，要么CP，要么AC，但是不存在CAP。如果在某个分布式系统中数据无副本， 那么系统必然满足强一致性条件， 因为只有独一数据，不会出现数据不一致的情况，此时C和P两要素具备，但是如果系统发生了网络分区状况或者宕机，必然导致某些数据不可以访问，此时可用性条件就不能被满足，即在此情况下获得了CP系统，但是CAP不可同时满足 。
> 因此在进行分布式架构设计时，必须做出取舍。当前一般是通过分布式缓存中各节点的最终一致性来提高系统的性能，通过使用多节点之间的数据异步复制技术来实现集群化的数据一致性。通常使用类似 memcached 之类的 NOSQL 作为实现手段。虽然 memcached 也可以是分布式集群环境的，但是对于一份数据来说，它总是存储在某一台 memcached 服务器上。如果发生网络故障或是服务器死机，则存储在这台服务器上的所有数据都将不可访问。由于数据是存储在内存中的，重启服务器，将导致数据全部丢失。当然也可以自己实现一套机制，用来在分布式 memcached 之间进行数据的同步和持久化，但是实现难度是非常大的 。  



**<font color=#FF0000 >Zookeeper保证的是CP</font>**
服务注册功能对一致性要求高，当master节点因为网络故障和其他节点失去联系时，剩余节点会重新进行Leader选举。这个时间持续30~120s，选举期间整个zk集群为不可用的，导致选举期间服务注册瘫痪。  
**<font color=#FF0000 >Eureka则是AP</font>**
优先保证可用性，Eureka各个节点都是平等的，如果某几个几点故障，只要有一台Eureka在，就能保证注册服务可用，只不过查到的信息可能不是最新的，而Eureka的自我保护机制，如果在15分钟内超过85的节点都没有正常心跳，那么Eureka就认为客户端与注册中心出现网络故障，会出现以下几种情况
1. Eureka不在从注册列表中移除因为长时间没有收到心跳而应该过期的服务
2. Eureka仍然能够接受新的服务注册和查询请求，但不会同步到其他节点，只保证当前节点依然可用
3. 当网络稳定时，当前实例新注册的信息会同步到其他节点

**<font color=#FF0000 >Eureka可以很好的应对网络故障导致的部分节点失去联系的情况，而不会像Zookeeper那样使整个注册服务瘫痪。</font>**

#### EurekaServer使用
**pom引入jar包**  

``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```  

**yml加入**  

``` yml
eureka: 
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client: 
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url: 
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ 
```  

**启动类加入注解启用**  

``` java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServer_App {
	public static void main(String[] args) {
		SpringApplication.run(EurekaServer_App.class, args);
	}
}
```  
---
#### EurekaClient 使用
**pom引入jar包**  

``` xml
<!-- 将微服务provider侧注册进eureka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```  

**yml加入**  

``` yml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url: 
      defaultZone: http://localhost:7001/eureka #EurekaServer的地址
  instance:
    instance-id: microservicecloud-dept8001 #别名
    prefer-ip-address: true     #访问路径可以显示IP地址  
```  

**启动类加入注解注册**

``` java
@SpringBootApplication
@EnableEurekaClient
public class EurekaClient_App {
	public static void main(String[] args) {
		SpringApplication.run(EurekaClient_App .class, args);
	}
}
```  
---
#### 服务提供者加入info监控信息
**父maven项目加入**  

``` xml
<build>
    <finalName>microservicecloud</finalName>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
            <configuration>
                <delimiters>
                    <delimit>$</delimit>
                </delimiters>
            </configuration>
        </plugin>
    </plugins>
</build>
```  

**pom加入**

``` xml
<!-- actuator监控信息完善 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```  

**yml加入**

``` yml
info: #点击别名页面所显示的说明
   app.name: atguigu-microservicecloud 
   company.name: www.atguigu.com
   build.artifactId: $project.artifactId$
   build.version: $project.version$  
```  
---
#### Eureka自我保护机制
某时刻某一微服务不可用了，eureka不会立刻清理，依旧会对该微服务的信息进行保存。  
默认情况下，如果EurekaServer在一定时间内没有接收到某个微服务实例的心跳，EurekaServer将会注销该实例，（默认90秒），但是当网络分区故障发生时，微服务与EurekaServer之间无法正常通信，Eureka会通过自我保护机制解决这个问题，一旦进入该模式，EurekaServer就会保护服务注册表中的信息，不再删除服务祖册表中的数据（也就是不会注销任何微服务）。当网络故障恢复后，该EurekaServer节点会自动退出自我保护模式。  
自我保护模式中，EureKa Server会保护服务注册表中的信息，不再注销任何服务实例。当它收到心跳数重新恢复到阀值以上时，该EurekaServer 节点会自动退出自我保护模式。  

**<font color=#FF0000 >总的来说就是在触发EurekaServer的保护模式后，它宁可保留错误的服务注册信息，也不会盲目注销任何可能健康的服务实例。</font>**

#### 禁用自我保护模式
**Eureka Server端**  

``` properties
eureka.server.enable-self-preservation #设为false，关闭自我保护
eureka.server.eviction-interval-timer-in-ms #清理间隔（单位毫秒，默认是60*1000）
```  

**Eureka Client端**  

``` properties
eureka.client.healthcheck.enabled #开启健康检查（需要spring-boot-starter-actuator依赖）
eureka.instance.lease-renewal-interval-in-seconds #续约更新时间间隔（默认30秒）
eureka.instance.lease-expiration-duration-in-seconds #续约到期时间（默认90秒
```  

#### Discovery服务发现
用于发现服务提供者
**主启动类上引用**  

``` java
@SpringBootApplication
@EnableDiscoveryClient //服务发现
public class EurekaClient_App
{
    public static void main(String[] args)
    {
        SpringApplication.run(EurekaClient_App.class, args);
    }
}
```  

**调用代码使用示例**  

``` java
@Autowired
private DiscoveryClient client;
public Object discovery()
    {
        List<String> list = client.getServices();
        System.out.println("**********" + list);

        List<ServiceInstance> srvList = client.getInstances("MICROSERVICECLOUD-DEPT");
        for (ServiceInstance element : srvList) {
            System.out.println(element.getServiceId() + "\t" + element.getHost() + "\t" + element.getPort() + "\t"
                    + element.getUri());
        }
        return this.client;
    }
```  

#### 集群配置
多个EurekaServer可以互相牵手形成集群，某一个注册中心因为特殊原因发生故障服务提供者无法注册时会向集群中的别的EurekaServer注册，当故障解除后集群中的EurekaServer再进行同步注册信息。  
**在EurekaServer yml中配置牵手**  

``` properties
eureka.client.service-url.defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
```  

**在Eureka client中向集群的所有EurekaServer注册 yml配置如下**  

``` properties
eureka.client.service-url.defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```  
