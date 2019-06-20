---
layout: post
title:  "Spring扩展功能：lombok 自动生成 get set 方法"
date:   2019-06-20 16:46:00 +0200
categories: 扩展功能
excerpt: lombok自动生成get、set方法
tagg: Spring
---

# lombok 自动生成 get set 方法

lombok,用于自动生成对象的get set方法，提高开发效率。不过我后来觉得这个功能还是比较鸡肋的，在idea右键生成getter和setter也差不多。

## 使用方法
### 依赖包：

```
        <!--    用于自动生成get set方法    -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
```

### idea安装

> Setting->Plugins:搜索lombok->安装

### 使用

只需在需要自动生成的类上添加注解：`@Data`

```
@Data
public class Author {
    private String name;
}
```

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.06.20  
> 更新日期：2019.06.20
