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

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.6.20  
> 更新日期：2019.6.20
