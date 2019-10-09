---
layout: post
title:  "Spring Boot学习记录:Mybatis"
date:   2019-05-15 16:19:00 +0200
categories: SpringBoot
excerpt: 
tagg: Spring
---

# Spring Boot核心
2019年5月8日15:56:29

### 基本设置
**入口类和@SpringBootApplication**

@SpringBootApplicaiton，Spring Boot的核心注解，是个组合注解，源码：
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM,
				classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
	@AliasFor(annotation = EnableAutoConfiguration.class)
	Class<?>[] exclude() default {};

	@AliasFor(annotation = EnableAutoConfiguration.class)
	String[] excludeName() default {};

	@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};

	@AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
	Class<?>[] scanBasePackageClasses() default {};

}
```
主要组合：@Configuration @EnableAutoConfiguration @ComponentScan
@EnableAutoConfiguration：根据类路径中的jar包依赖自动配置项目
例如spring-boot-start-web依赖，会自动添加tomcat和Spring MVC的依赖，并自动配置

SpringBoot会自动扫描@SpringBootApplication所在类同级包和下级包的bean

**关闭特定的自动配置**
```
@SpringBootApplication(exclude ={DataSourceAutoConfigruation.class})
```
**DIY banner**   
这个就是在src/main/resource 中新建个banner.txt 然后把banner弄上去。没啥用。。

- 关闭banner在入口文件main中
```
SpringApplication app = new SpringApplication(TestApplication.class, args);
app.setShowBanner(false);
app.run();
```
但是我配置的SpringBoot根本没有setShowBanner这个方法。具体原因就先不去解决了

**配置文件**  
src/main/resource源文件夹中的application.properties

**starter pom**  
用于简化企业级开发绝大多数场景

**xml配置**
Spring Boot提倡零配置，但实际项目有时候必须用到xml则需要使用注释`@ImportResource("classpath:some-context.xml","classpath	:another-context.xml")`

**常规属性配置**
在application.properties中配置的值，直接使用`@Value`  
例如application.properties中配置：
```
book.name=spring boot

@Value("{$book.name}")
private String bookName;
```
但是上面的方法在有很多参数的时候显得很麻烦，这时候使用注`解@ConfigurationProperties`  
例：  
```
在application.properties中：
author.name=hiki
author.age=22
```
在需要的类上
```
@ConfigurationProperties(prefix="author")
public class AuthorSettings(){
private String name;
private Long age;
}
```
如果是要指定properties位置则：
```
@ConfigurationProperties(prefix="author",locations = {"classpath:config/author.properties"}))
```
**日志配置**
```
Spring Boot默认使用logback为日志框架
配置日志级别：
logging.file=D:/mylog/log.log
配置日志文件，格式为logging.level.包名=级别
logging.level.org.springframework.web=DEBUG

```
**Profile配置|多环境配置**  
Profile是Spring用来针对不同环境对不同配置提供支持。例如生产环境(prod)和开发环境(dev)  
用法：  
- 在`application.properties`所在文件夹新建properties,分别命名为`application-prod.properties`和`application-dev.properties`
- 在`application.properties`增加配置使用dev环境：
```
spring.profiles.active=dev
```

<br /><br /><br /><br />
> [项目代码](https://github.com/Hikiy/SpringBootLearn)  
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.05.08  
> 更新日期：2019.10.09
