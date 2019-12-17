---
layout: post
title:  "算法：快慢指针妙用"
date:   2019-12-17 17:41:00 +0200
categories: 算法
excerpt: 快慢指针是指，在解决一些问题的时候，快指针比慢指针多走几步，用于判断特殊情况
---

# 算法：快慢指针妙用

快慢指针是指，在解决一些问题的时候，快指针比慢指针多走几步，用于判断特殊情况

## 一、找循环体

用来判断一个过程是否无限循环。

### 1.链表中的环

![](https://note.youdao.com/yws/public/resource/285f2a95b7993458b819328026a11e83/xmlnote/A9B1BFEF8BB0465981A6F748A31996E1/23017)

- 遍历链表
- 快指针如果最终等于慢指针则有环

### 2.逻辑无限循环（快乐数）

在leetcode做算法题遇到的快乐数问题： https://leetcode-cn.com/problems/happy-number/

> 编写一个算法来判断一个数是不是“快乐数”。
>
> 一个“快乐数”定义为：对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和，然后重复这个过程直到这个数变为 1，也可能是无限循环但始终变不到 1。如果可以变为 1，那么这个数就是快乐数。

```
输入: 19
输出: true
解释:
1² + 9² = 82
8² + 2² = 68
6² + 8² = 100
1² + 0² + 0² = 1
```

**使用快慢指针的思想，其中一个平方和每次做两次，与另一个平方和作对比。当进入循环是查看是否等于1即可。**

```
//快慢指针，当相等的时候说明进入循环，判断是否是等于1引起的循环
    public boolean isHappy(int n) {
        int fast = n;
        int slow = n;

        do{
            slow = sum(slow);
            fast = sum(fast);
            fast = sum(fast);
        }while( fast != slow );

        return slow == 1;
    }

    public int sum(int n){
        int sum = 0;
        while(n > 0){
            int bit = n%10;
            sum += bit * bit;
            n = n/10;
        }
        return sum;
    }
```

## 二、找中间值

在链表中，寻找中间值也可以使用快慢指针。

- 快指针每次比慢指针多走一步。
- 当快指针到结尾时，慢指针所在位置即中间值。

## 三、寻找倒数第n个值

快指针比慢指针提前走n-1步即可

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.12.17  
> 更新日期：2019.12.17