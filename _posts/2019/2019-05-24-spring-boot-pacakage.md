---
layout: post
title:  "Spring Boot学习记录：打包"
date:   2019-05-24 15:46:00 +0200
categories: SpringBoot
excerpt: 
tagg: Spring
---

# 打包

## jar包
### maven打包：
```
cd 项目跟目录（和pom.xml同级）
mvn clean package
## 或者执行下面的命令
## 排除测试代码后进行打包
mvn clean package  -Dmaven.test.skip=true
```
打包完成后 jar 包会生成到 target 目录下，命名一般是 项目名+版本号.jar  

#### 启动 jar 包命令
```
java -jar  target/springbootlearn-1.0.0.jar
```
控制台关闭，服务就不能访问。

#### 后台运行方式
```
nohup java -jar target/springbootlearn-1.0.0.jar &
```

#### 选择启动时的配置文件
```
java -jar app.jar --spring.profiles.active=dev
```

#### 设置jvm参数
```
java -Xms10m -Xmx80m -jar app.jar &
```

### gradle打包
```
gradle build
java -jar build/libs/mymodule-0.0.1-SNAPSHOT.jar
```

<br /><br /><br /><br />
> [项目代码](https://github.com/Hikiy/SpringBootLearn)  
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.05.24  
> 更新日期：2019.05.24
