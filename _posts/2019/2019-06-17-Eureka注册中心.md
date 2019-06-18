---
layout: post
title:  "Spring Cloud学习记录：Eureka 注册中心"
date:   2019-06-17 17:04:00 +0200
categories: SpringCloud
excerpt: Eureka是Netflix开源的一款提供服务注册和发现的产品，它提供了完整的Service Registry和Service Discovery实现。也是springcloud体系中最重要最核心的组件之一。
tagg: Spring
---

# Eureka 注册中心

Eureka是Netflix开源的一款提供服务注册和发现的产品，它提供了完整的Service Registry和Service Discovery实现。也是springcloud体系中最重要最核心的组件之一。

**注册中心是分布式系统中最重要的基础部分，也正因为如此，注册中心必须是高可用的。**

在一个分布式服务中，需要客户端A从服务端B获取信息，那么A需要B的地址。但是如果B的节点越来越多，并且动态变化，A将难以找到。所以如果有了中间商注册中心，B来了一个就注册到注册中心，A只需要找注册中心要B即可。

**那么当注册中心给A提供了许多B，那A怎么选择？**

一种就是A通过一些随机、哈希之类的方式选其中一个B进行使用。这是**客户端发现**。这是一种负载均衡策略。
> 特点：实现简单，不用代理。但是需要A要有自己的逻辑来选B

一种是采用代理从很多的B中挑选一个给A使用。这是**服务端发现**
> 特点：B和注册中心对A是透明不可见的，A只需要招代理发送请求即可。例如Nginx、Zookeeper、Kubernetes

**Eureka采用的是客户端发现的方式**

## idea创建新项目
- 选择Spring Initializr(JDK一定要选1.8版本的！！坑！)
- 如果页面打不开将`https://start.spring.io`换成`http://start.spring.io`
- 输入名字后下一步
- 选Spring Cloud Discovery->Eureka Server
- 新建完毕

## 主要依赖

```
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
```

## 入口类中添加@EurekaServer注解

```
@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }

}
```

## 修改配置文件application.yml:

```
server:
  port: 8761
  
eureka:
  client:
    fetch-registry: false
    service-url:
      defaultZone: http://localhost:8761/eureka/
    register-with-eureka: false
    
spring:
  application:
    name: eureka
```

在**开发环境**中，免不了client端的多次重启，为了避免警告，可以添加下面配置，但是**生产环境**中记得要删掉！

```
eureka:
  server:
    enable-self-preservation: false
```

- `eureka.client.register-with-eureka`：表示是否将自己注册到EurekaServer，默认为true。
- `eureka.client.fetch-registry`：表示是否从EurekaServer获取注册信息，默认为true。
- `eureka.client.serviceUrl.defaultZone`：设置与Eureka Server交互的地址，查询服务和注册服务都需要依赖这个地址。默认是http://localhost:8761/eureka ；多个地址可使用 , 分隔。

默认设置下，该服务注册中心也会将自己作为客户端来尝试注册它自己，所以我们需要禁用它的客户端注册行为。

## 启动项目
可以看到Instances currently registered with Eureka栏目中还没有注册的服务。

## 打包
因为注册中心在整体项目中是每次都需要启动的，然而如果每次都到idea去启动的话会相对麻烦，所以将它打成jar包运行会方便很多

### 打包：
命令行定位到eureka项目的目录后输入命令：
```
mvn clean package
```

**PS:这里如果报错：`No compiler is provided in this environment. Perhaps you are running on a JRE rather than a JDK?`**

**原因：**
项目使用版本的jdk和maven环境使用的jdk版本不一样。仔细检查：
- idea项目使用的jdk

> Setting->Build,Execution,Deployment->Compiler->java Compiler：改好版本  
> 右键项目->F4->Project：更改Project SDK和Project language level  
> 右键项目->F4->Project：更改好版本

- 系统环境变量中java的版本，要和idea使用的是同一个JDK！！

### 启动jar包

```
java -jar target\eureka-0.0.1-SNAPSHOT.jar
```
可以通过ctrl+c停止程序

### 后台运行

#### macOS下:
```
nohup java -jar target/eureka-0.0.1-SNAPSHOT.jar > /dev/null 2>&1 &
```
会返回进程PID

也可通过指令获得PID：

```
ps -ef |grep eureka
```

通过指令终止进程：

```
kill -9 PID
```

#### Win下

新建bat批处理文件 并启动：
```
@echo off
start javaw -jar xxxxxxxx-SNAPSHOT.jar
exit
```
建议jar包与批处理文件处于同一目录，启动后会生成log文件，即项目日志

在产生的log文件中查看PID，然后到任务管理器的进程找到PID可终止进程

## Eureka Server高可用：集群

如果注册中心崩了，那服务将不能被发现，为了避免这种情况，可以多启动个Server
和原本的Server互相注册，下面演示简单方法

### 创建新的Application

- idea项目启动按钮的左边有下拉栏，点击并编辑
- 复制一个Application
- 在envirnmenrt->VM options中输入`-Dserver.port=8762`设置新的端口
- 更改原来的Application:envirnmenrt->VM options输入`-Dserver.port=8761`
- 注释掉原本application.yml的端口配置

### 互相注册

#### 方法一：
- 在下拉栏选择application1
- 在配置文件中修改配置注册到application2的端口号
- 运行
- 下拉栏选择applicaiton2
- 在配置文件中修改配置注册到application1的端口号
- 运行

#### 方法二：
- 新建一个application-peer1.yml:

```
server:
  port: 8761
  
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8762/eureka/

spring:
  application:
    name: eureka
```

- 新建一个application-peer2.yml:

```
server:
  port: 8762
  
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/

spring:
  application:
    name: eureka

```

- 打包

```
mvn clean package
```

- 启动两个服务

```
java -jar  target\eureka-0.0.1-SNAPSHOT.jar --spring.profiles.active=peer1
java -jar  target\eureka-0.0.1-SNAPSHOT.jar --spring.profiles.active=peer2
```

### 在client端配置
修改application.yml的配置：

```
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/,http://localhost:8762/eureka/
```

~~然后在8762和8761端口的注册中心都能看到注册了的服务。~~

> 但是测试时并未成功。。8762的端口并未出现注册了的服务，后来测试将8761的注册中心关闭，8762的注册中心才会出现该client，可能spring cloud后面进行了修改

### 更多的注册中心
只要每个注册中心两两互相注册即可

<br /><br /><br /><br />

> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.06.17  
> 更新日期：2019.06.18
