---
layout: post
title:  "node.js学习记录：let和const使用理解"
date:   2019-04-27 19:57:00 +0200
categories: nodejs
tagg: node.js
excerpt: 关于let和const命令的使用理解
---

# 关于let和const命令的使用理解

## let
用来声明变量。用法类似于var，但不同的是所声明的变量，只在let 命令所在的代码块内有效。  
**let不允许在同一个作用域内对同一个变量重复声明。**
```
// 报错

function testRedefined(){
let a = 10;
let a = 20;
}
```
## const
const声明一个只读的常量。一旦声明，常量的值就不能改变。  

const声明的变量和let有类似的限制，命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用，同样在一个相同的作用域内不能重复声明。  

**const实际保证的并不是变量的值不得改动，而是变量指向的那个内存地址不得改动。**

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.04.27  
> 更新日期：2019.04.27
