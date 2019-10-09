---
layout: post
title:  "Java并发：多线程的合理配置"
date:   2019-07-11 15:50:00 +0200
categories: 并发
excerpt: 创建多少线程合理呢？
tagg: Java
---

# 多线程的合理配置

多线程就不多说了，很容易理解怎么回事
![](https://note.youdao.com/yws/public/resource/aab7147570c5edbb8c0c3eda4018495b/xmlnote/9B43E18934984BE8A917DFF5340DC803/18349)

## 应用场景

### I/O密集型
大部分情况下，I/O 操作执行的时间相对于 CPU 计算来说都非常长，这种场景我们一般都称为 I/O 密集型计算

### CPU密集型
，CPU 密集型计算大部分场景下都是纯 CPU 计算。

## 创建多少线程合理
I/O 密集型程序和 CPU 密集型程序，计算最佳线程数的方法是不同的。

对于 CPU 密集型的计算场景，理论上**线程的数量 =CPU 核数**就是最合适的。不过在工程上，线程的数量一般会设置为**CPU 核数 +1**

**最佳线程数 =CPU 核数 * [ 1 +（I/O 耗时 / CPU 耗时）]**

测试耗时比可以用apm工具

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.07.11  
> 更新日期：2019.07.11
