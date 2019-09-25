---
layout: post
title:  "Spring Boot学习记录：多模块构建"
date:   2019-09-25 20:54:00 +0200
categories: SpringBoot
excerpt: 
tagg: Spring
---

# 多模块构建

## 重构

### 1.调整主（父）工程类型（<packaging>)

将父工程 `pom.xml` 文件中的 `<packaging>` 改成 `pom`

```
<packaging>pom</packaging>
```

### 2.创建子模块工程（<module>）

#### idea上创建子模块

- 右键父工程新建 Module
- 选择maven的方式构建，
- 默认选项next就行了

然后会发现新建的子模块中的 `pom.xml` :

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>product</artifactId>
        <groupId>com.hiki</groupId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>client</artifactId>


</project>
```

重点是 `<parent>` 中指定父工程

#### 查看父工程 `pom.xml` 

```
<modules>
    <module>common</module>
    <module>server</module>
    <module>client</module>
</modules>
```

### 3.子模块依赖管理（<dependencyManagement>）

多模块需要互相依赖时（调用别的模块的类型），需要在 `pom.xml` 中添加，例如：

```
    <dependencies>
        <dependency>
            <groupId>com.hiki</groupId>
            <artifactId>common</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>
```

但是值得注意的是，上面的配置需要添加版本号，这样不好管理，所以在父项目中配置，先指明依赖，那么在上面的模块就不需要指明版本号了，因为依赖于父项目:

```
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.hiki</groupId>
                <artifactId>common</artifactId>
                <version>0.0.1-SNAPSHOT</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

### 4.mainClass转移

在多模块中，mainClass可能是在其中一个模块中，打包的时候会无法识别，所以需要配置到主项目的 `pom.xml` 中，但是，如果觉得麻烦，可以将这个build放在主项目中即可，不需要配置mainClass

```
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.hiki.product.ProductApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

主要是这段代码：

```
<configuration>
    <mainClass>com.hiki.product.ProductApplication</mainClass>
</configuration>
```

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.9.25  
> 更新日期：2019.9.25
