---
title: SpringCloud Config分布式配置中心
date: 2019-11-20 18:19:36
tags:
 - 微服务
 - SpringCloud
 - Java
---
SpringCloud Config为微服务架构中的微服务提供了集中化的外部配置支持，配置服务器为各自不同微服务应用的所有环境提供了一个中心化的外部配置。  
**服务端：** 称为分布式配置中心，是一个独立的微服务应用，用来连接配置服务器并为客户端提供配置信息，加密/解密信息等访问接口。  
**客户端：** 通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息。    

#### config服务端使用  
需要向集群注册自己，config的客户端才能访问  
**pom加入**
``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```  
**yml加入**  
``` yml
server: 
  port: 3344 
  
spring:
  application:
    name:  microservicecloud-config
  cloud:
    config:
      server:
        git:
          uri: https://github.com/a984958991/microservicecloud-config #GitHub上面的git仓库名字
```  
**主启动类**
``` java
@SpringBootApplication
@EnableConfigServer
public class Config_3344_StartSpringCloudApp
{
	public static void main(String[] args)
	{
		SpringApplication.run(Config_3344_StartSpringCloudApp.class, args);
	}
}
```  
测试访问方式：http://localhost:3344/application-dev.yml  

---  

#### config客户端使用
**pom加入**
``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```  
**加入优先级最高的bootstrap.yml（系统级）**
``` yml
spring:
  cloud:
    config:
      name: microservicecloud-config-client #需要从github上读取的资源名称，注意没有yml后缀名
      profile: test   #本次访问的配置项
      label: master   
      uri: http://config-3344.com:3344  #本微服务启动后先去找3344号服务，通过SpringCloudConfig获取GitHub的服务地址 config服务端的地址
```  

示例配置文件码云地址:  https://gitee.com/jinqiansheng/microservicecloud-config  