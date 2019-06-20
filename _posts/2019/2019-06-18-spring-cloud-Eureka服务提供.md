---
layout: post
title:  "Spring Cloud学习记录：Eureka 服务提供"
date:   2019-06-18 10:18:00 +0200
categories: SpringCloud
excerpt: 
tagg: Spring
---

# Eureka 服务提供

## 服务端注册
### idea新建项目
- 选择Spring Initializr
- 如果页面打不开将`https://start.spring.io`换成`http://start.spring.io`
- 输入名字后下一步
- 选Spring Cloud Discovery->Eureka Discovery
- 新建完毕

### 主要依赖

```
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
```

### 入口类中添加@EurekaDiscoveryClient注解

```
@SpringBootApplication
@EnableDiscoveryClient
public class EurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }

}
```

### 修改配置文件application.yml:

```
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    hostname: clientName

spring:
  application:
    name: client
```
其中，defaultZone是注册中心的URL，instance.hostname是自定义链接名字。即在注册中心点击status下的链接后，url栏显示的名字，一般不用。

这时启动会发现启动后直接关闭，百度发现说说是项目少了依赖包。。这个自动生成的项目果真不靠谱啊。。

### 在pom.xml添加依赖
```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 启动项目
启动项目后，在浏览器打开注册中心的url，看到client已经注册进去了

**PS：有时候会在注册中心看到提示：**

EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.

**这是因为服务不断重启，会导致注册中心的自我保护机制，server端和client端采用心跳机制，server端会不断检查这些client端是否上线，一定时间内统计上线率，上线率低则会报这个警告，不知道这个client到底是否上线，但是宁可信其有不可信其无，当做其上线**

这个问题需要在server端即注册中心解决。

在注册中心的application.yml上加上代码：

```
eureka:
  server:
    enable-self-preservation: false
```



<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.06.18  
> 更新日期：2019.06.18
