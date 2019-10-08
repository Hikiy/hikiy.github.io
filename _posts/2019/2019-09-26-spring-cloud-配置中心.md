---
layout: post
title:  "Spring Cloud学习记录：配置中心"
date:   2019-09-26 15:33:00 +0200
categories: SpringCloud
excerpt: 
tagg: Spring
---

# 配置中心

## config-server原理

- config-server 会先从远端git拉取，并且在本地保存一份git。
- 远端git失效的时候可以读取本地git。
- 服务从 config-server 读取配置信息

![](https://note.youdao.com/yws/public/resource/d9c5ca59c2754ea8d0fc632f541c8a53/xmlnote/A5902562DE0945199E8074D645335DFC/21881)


## 一、新建项目

- 因为配置中心也是微服务，所以需要 `Eureka Discovery`
- 配置中心需要 `Config Server`

## 二、给启动类添加注解

```
@SpringBootApplication
@EnableDiscoveryClient
@EnableConfigServer
public class ConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigApplication.class, args);
    }

}
```

## 三、配置git库

配置中心需要从git库中获取代码，需要配置git库

```
spring:
  application:
    name: config
  cloud:
    config:
      server:
        git:
          username: 用户名
          uri: giturl
          password: 密码
          basedir: 从git拉取的文件存放位置，也可以不配置，有默认位置
      label: 读取的分支，可以不写
eureka:
  client:
    service-url:
      defaltZone: http://localhost:8761/eureka/
```

## 四、访问配置文件

浏览器访问git库中的一个文件： `http://localhost:8080/order-a.yml`，可以看到文件内容，再访问的时候会发现控制台输出:

```
Adding property source: file:/C:/Users/Hiki/AppData/Local/Temp/config-repo-7956189288709612648/order.yml
```

访问的文件后缀可以不一样，例如json格式。

### 访问文件的格式

`/{name}-{profiles}.yml`  
`/{label}/{name}-{profiles}`

name : 文件名  
profiles : 环境
label : 分支（默认master分支）

在使用不同环境、不同分支的时候需要在git库的配置文件配置：

```
env:
  dev
label:
  release
```

## 五、配置Config Client

也就是微服务

### 1.依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

### 2.配置文件

```
spring:
  application:
    name: order
  cloud:
    config:
      discovery:
        enabled: true
        service-id: CONFIG(这个是注册在注册中心的配置中心名)
      profile: dev
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
```

### 3.修改`application.yml`配置名

要修改成 `bootstrap.yml` ，只有这样配才会先读取配置


## 六、实现动态更新

### 1.原理：

- config-server 和 微服务通过 消息队列传递信息
- config-server 使用 Sring Bus 后对外提供一个**http接口** ： `/bus-refresh`
- 访问 `/bus-refresh` 后， config-server 会把更新配置的信息发送到 微服务中

从原理上看，肯定是git来访问 `/bus-refresh` 是最合适的

### 2.配置 config-server

#### 添加依赖：

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

#### 配置 application.yml

默认 `/actuator/bus-refresh` 是无法被访问的，需要暴露出来

```
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

启动后就可以访问 `/actuator/bus-refresh` 进行刷新了

### 3.配置微服务

#### 添加依赖:

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

如果需要在一些类中获取配置文件的配置，需要在这个类上添加注解 `@RefreshScope`

```
@Component
@ConfigurationProperties("author")
@RefreshScope
public class AuthorConfig{
    private String name;
    private Integer age;
}
```

### 4.git仓库配置调用

这个需要你的项目有暴露出公网的ip。

#### github上配置

- 选择项目中的 Settings
- 点击左侧 Webhooks
- Add webhook

config组件给Webhooks提供的路由为: `/monitor`。

- 选择json形式

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.09.26  
> 更新日期：2019.09.26