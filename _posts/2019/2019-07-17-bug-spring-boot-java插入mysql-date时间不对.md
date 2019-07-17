---
layout: post
title:  "Spring Boot报错： java 插入mysql date 时间不对"
date:   2019-07-17 11:03:00 +0200
categories: 报错记录
excerpt: 使用java的new Date新建的时间插入到mysql中，发现mysql的时间不一样。
tagg: Spring
no-post-nav: true
---

# java 插入mysql date 时间不对

使用java的new Date新建的时间插入到mysql中，发现mysql的时间不一样。

## 解决方案
这是时区的问题。

将时区改为CTT（Asia&Shanghai）
```
url: jdbc:mysql://127.0.0.1:3306/table?characterEncoding=utf-8&useSSL=false&serverTimezone=CTT
```

<br /><br /><br /><br />

> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.07.17  
> 更新日期：2019.07.17
