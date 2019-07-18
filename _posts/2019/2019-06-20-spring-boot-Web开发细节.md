---
layout: post
title:  "Spring Boot学习记录：Web开发细节"
date:   2019-06-20 15:12:00 +0200
categories: SpringBoot
excerpt: 
tagg: Spring
---

# Web开发细节

## Json返回歧义

有时候在返回给前端Json时，有些字段可能在后端会引起歧义。例如返回Json:
```
{
    "name" : "Hiki"
}
```
但是在后端这个 `name` 容易引起歧义，需要使用 `authorName` 。

### 解决方案：

那么我们就需要给 `authorName` 字段加上注解 `@JsonProperty("name")` ：

```
    @JsonProperty("name")
    private String categoryName;
```

------------↓2019.07.17↓----------------
## 增加日志
有时候错误需要日志支持

### @Slf4j注解
在需要日志的类上添加注解@Slf4j

```
@Slf4j
public class HikiController {
    public void add(){
        log.error();
    }
}
```

## 参数验证
有更方便的方法进行参数验证

### 使用@Valid注解

1.为表单新建类：

```
public class HikiForm {
    @NotEmpty(message = "姓名必填")
    private String name;

    @NotNull(message = "手机不能为null")
    private String phone;
}
```

2.在Controller验证上添加@Valid注解

```
@RequestMapping("/add")
    public void add(@Valid HikiForm hikiForm, BindingResult bindingResult){
        if( bindingResult.hasErrors() ){
            String message = bindingResult.getFieldError().getDefaultMessage();
        }
    }
```

其中`bindingResult.getFieldError().getDefaultMessage()`获取的就是表单中注释的message；

------------↓2019.07.18↓----------------
## Json与对象的转换

使用Google提供的Gson类

### 依赖
因为Spring Boot有引入，所以无需版本号
```
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
        </dependency>
```

### 将Json转成对象
使用Gson的fromJson()。第一个参数为需要转换的Json，第二个参数为转换成的类型

这里用到了TypeToken来获取类型。因为可能会是失败，所以使用try catch块圈住
```
    Gson gson = new Gson();
    try{
        someList = gson.fromJson(somejson,new TypeToken<List<Some>>(){}.getType());
    }catch (Exception e){
        ...
    }
```

### 将对象转成Json
使用Gson的toJson()

<br /><br /><br /><br />

> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.6.20  
> 更新日期：2019.7.18