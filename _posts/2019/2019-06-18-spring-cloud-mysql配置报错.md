---
layout: post
title:  "Spring Boot报错：Mysql错误 The server time zone value '�й���׼ʱ��' is unrecognized "
date:   2019-06-18 17:25:00 +0200
categories: 报错记录
excerpt: Mysql配置报错：The server time zone value '�й���׼ʱ��' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver
tagg: Spring
no-post-nav: true
---

# Mysql配置报错：The server time zone value '�й���׼ʱ��' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver

在学习Spring Cloud的过程中，配置mysql数据库后启动项目报错：
```
The server time zone value '�й���׼ʱ��' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver
```
因为时区问题，所以在配置文件中要配置好时区`serverTimezone=UTC`：
```
spring:
  application:
    name: product
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456
    url: jdbc:mysql://127.0.0.1:3306/springcloud?characterEncoding=utf-8&useSSL=false&serverTimezone=UTC
```

准确来说归为Spring Boot的报错。

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.06.18  
> 更新日期：2019.06.18
