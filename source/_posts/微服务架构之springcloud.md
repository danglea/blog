---
title: 微服务架构之springcloud
date: 2019-11-17 18:22:38
tags: ['java','微服务']
---

***简介：*** 微服务英文名称Microservice，Microservice架构模式就是将整个Web应用组织为一系列小的Web服务。这些小的Web服务可以独立地编译及部署，并通过各自暴露的API接口相互通讯。它们彼此相互协作，作为一个整体为用户提供功能，却可以独立地进行扩。

<!--more-->

## 1.单体架构和微服务架构

##### 1.单体架构

![](http://dsvip1.vip/QQ%E5%9B%BE%E7%89%8720191216181515.png)

单体架构俗称一个war包打天下，缺点很多不好延伸，性能太低，一个服务维护所有的项目都无法访问。就容易部署。

##### 2.微服务架构

![](http://dsvip1.vip/QQ%E5%9B%BE%E7%89%8720191216182030.png)

每一个模块之间都可以相互的调用，每一个服务都是一个单独的模块

## 2.服务注册中心Eureka

##### 1.父工程引入pom

```java
<dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Finchley.RELEASE<version>
                <type>pom<type>
                <scope>import</scope>   //springcloud版本
</dependency>
 <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.0.1.RELEASE<version> //springboot版本
                <type>pom<type>
                <scope>import</scope>
 </dependency>
```

##### 2.Eureka模块引入

```java
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId></dependency>
```

###### pom配置

```java
server:
  port: 9001 #服务端口
eureka:
  instance:
    hostname: eureka9001  #eureka服务实例名称
  client:
    register-with-eureka: false #false表示不向注册中心注册自己
    fetch-registry: false #false 表示自己就是注册中心，职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://127.0.0.1:${server.port}/eureka/   #单机配置服务地址

```

###### 启动配置

```java
@EnableEurekaServer
```

## 3.服务模块配置

##### 1.Eureka模块引入

```java
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>  //Eureka客户端配置
</dependency>
```

##### 2.客户端配置文件

```java
eureka:
  client:
    service-url:
      defaultZone: http://localhost:9001/eureka
  instance:
    prefer-ip-address: true
    instance-id: springlcoud-eureka8002
```

如果配置集群则模块之间的

```java
spring:
  application:
    name: springcloud-provider     #配置的名字要一样
```

Eureka自带负载均衡，我们可以修改算法

这样我们的服务模块就注册到里面了，我们就可以通过zuul在Eureka中去调用了

## 4.模块之间相互调用Feign

##### 1.添加依赖,也要注册到Eureka服务注册中心里

```java
  <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
 </dependency>
```

##### 2.启动类添加注解

```java
@EnableFeignClients     //用于feign
@EnableDiscoveryClient   //用于feign
```

要写一个调用模块的接口，并添加想要调用的接口

```java
@FeignClient(value = "SPRINGCLOUD-PROVIDER",fallback = deptClientImpl.class) //指明调用模块在Eureka中的名字，如果出错就执行fallback
public interface deptClient {
    @RequestMapping("dept/name/{ids}")
    public Dept findDept(@PathVariable("ids") int ids);
}
```

deptClientImpl是deptClient的实现类，并交给spring管理

fallback属于hystrix熔断器，调用不了就执行deptClient避免发生雪崩效应

```java
feign:
  hystrix:
    enabled: true
```

## 5.微服务网关Zuul

##### 1.介绍

Zuul是Netflix开源的微服务网关，他可以和Eureka,Ribbon,Hystrix等组件配合使 

用。Zuul组件的核心是一系列的过滤器，这些过滤器可以完成以下功能： 

 1.身份认证和安全: 识别每一个资源的验证要求，并拒绝那些不符的请求 

审查与监控： 

2.动态路由：动态将请求路由到不同后端集群 

3.压力测试：逐渐增加指向集群的流量，以了解性能 

4.负载分配：为每一种负载类型分配对应容量，并弃用超出限定值的请求 

5.静态响应处理：边缘位置进行响应，避免转发到内部集群 

6.多区域弹性：跨域AWS Region进行请求路由，旨在实现ELB(ElasticLoad Balancing)使 

用多样化

![](http://dsvip1.vip/QQ%E5%9B%BE%E7%89%8720191220145827.png)

##### 2.依赖添加，也要注册到Eureka

```java
 <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
 </dependency>
 <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
   </dependency>
```

##### 3.基本配置

```java
zuul:
  routes:
    springcloud-provider:
      path: /zuul/**
      serviceId: springcloud-provider          #指定注册eureka注册中心的id
  servlet-path: /
```

##### 4.启动类添加

```java
@EnableZuulProxy 
@SpringBootApplication
```

