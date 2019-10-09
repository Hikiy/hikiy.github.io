---
layout: post
title:  "Spring Boot学习记录:Mybatis"
date:   2019-05-15 16:19:00 +0200
categories: SpringBoot
excerpt: 
tagg: Spring
---

# Mybatis

有两种模式：

- **注解模式**
- **XML模式**  

注解模式开发快，但是要动态SQL还是要XML模式。

### Maven

```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
	<dependency>
		<groupId>org.mybatis.spring.boot</groupId>
		<artifactId>mybatis-spring-boot-starter</artifactId>
		<version>2.0.0</version>
	</dependency>
     <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
```

### application.properties

```
spring.datasource.url=jdbc:mysql://localhost:3306/springboot?serverTimezone=GMT&useSSL=false
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```




<br /><br /><br /><br />
> [项目代码](https://github.com/Hikiy/SpringBootLearn)  
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.05.15  
> 更新日期：2019.05.23
