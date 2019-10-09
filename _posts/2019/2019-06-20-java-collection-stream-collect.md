---
layout: post
title:  "Java知识补充：collection.stream()以及collect()方法"
date:   2019-06-20 16:00:00 +0200
categories: java
excerpt: 
tagg: Java
---

# collection.stream()以及collect()方法

这是在网上学习的时候看到的方法：

```
List<Integer> categoryTypeList = productInfoList
                        .stream()
                        .map(ProductInfo::getCategoryType)
                        .collect(Collectors.toList());
```

这条语句主要实现从类型为 `<ProductInfo>` 的List: `productInfoList` 中获取 `productInfo` 的属性 `CategoryType` 的集合。

原解决方案应该为：
- 遍历 `productInfoList` 获取 `ProductInfo` 对象
- 从每个 `ProdcutInfo` 中获取 `CategoryType`
- 把获取的 `CategoryType` 存入新的List当中。

本解决方案更为高效简介。

stream()优点
>
> - 1.无存储。stream不是一种数据结构，它只是某种数据源的一个视图，数据源可以是一个数组，Java容器或I/O channel等。
> - 2.为函数式编程而生。对stream的任何修改都不会修改背后的数据源，比如对stream执行过滤操作并不会删除被过滤的元素，而是会产生一个不包含被过滤元素的新stream。
> - 3.惰式执行。stream上的操作并不会立即执行，只有等到用户真正需要结果的时候才会执行。
> - 4.可消费性。stream只能被“消费”一次，一旦遍历过就会失效，就像容器的迭代器那样，想要再次遍历必须重新生成。

更多示例：
```

List<String> list= Arrays.asList("a", "b", "c", "d");

List<String> collect =list.stream().map(String::toUpperCase).collect(Collectors.toList());
System.out.println(collect); //[A, B, C, D]


List<Integer> num = Arrays.asList(1,2,3,4,5);
List<Integer> collect1 = num.stream().map(n -> n * 2).collect(Collectors.toList());
System.out.println(collect1); //[2, 4, 6, 8, 10]
```
更多的知识还未补充，若翻到可以先到别人写的博客先看:[stream介绍，以及lambda表达式的使用](https://blog.csdn.net/lidai352710967/article/details/82496783)

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.06.20  
> 更新日期：2019.06.20
