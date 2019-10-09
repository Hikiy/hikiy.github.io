---
layout: post
title:  "博客bug：Page build failed"
date:   2019-10-09 10:55:00 +0200
categories: 博客改进
excerpt: Github Page build failed
---

# 博客Page build failed

Github Page build failed

该bug发生在国庆后，我也不知道具体发生了什么，可能github的page进行了改动。在国庆之前是正常运行的，但是再国庆之后，新commit的博客发生了错误。github还会发送错误邮箱：

> The page build failed for the `master` branch with the following error:  
> Page build failed. For more information, see https://help.github.com/en/articles/troubleshooting-jekyll-build-errors-for-github-pages-sites#troubleshooting-build-errors.  
For information on troubleshooting Jekyll see:  
> 
>  https://help.github.com/articles/troubleshooting-jekyll-builds
>
> If you have any questions you can contact us by replying to this email.

![](https://hiki-blog.oss-cn-shenzhen.aliyuncs.com/githubio/bug/bug_20191009102748.png)
![](https://hiki-blog.oss-cn-shenzhen.aliyuncs.com/githubio/bug/bug_20191009104823.png?Expires=1570593017&OSSAccessKeyId=TMP.hVHzoQPphgabRGiuVhq38Pt5Qtj3VwWhUubzWVbjNiJtAaDBYAkqKaSoMTVC12gsYiRsCuaRU3BMpz5irzyTZVdAffYKFm3sbZkqdWHtK1iYeLEmSgKpHJRURgZbmc.tmp&Signature=pvzbyaT3D5FFxO%2FAbR2HqtpiLvk%3D)

## 解决流程

### 1.排除本身jekyll的问题

执行命令本地启动：

```
bundle exec jekyll serve
```

因为之前都是成功的，显然本地运行成功了。

### 2.排查是不是Github版本不支持

可以到官网查看：

https://pages.github.com/versions/

确实有些版本是更新的，但是更新后依然没有解决问题。。

### 3.排查新文章问题

暴力点就是**直接删掉新文章**

结果还是**失败**了。。这说明是以前文章的问题。

**删除了最后一次成功的文章**，依然**失败**。

这时候真的有点绝望，既然不是最新的这几个文章问题，那是哪里出错了？

以后还要写文章，不能放弃，我必须排查出来！

备份博客，**删除大量文章，只保留最初两个文章**

**成功了！**

但是这样就头大了，说明是这之间的文章有问题。。因为已经有很多文章了，一个个排查真的会死人

于是我按照月份一个个查，最后缩小了范围，锁定在一篇文章中（真的花了两天）。


### 4.找到问题

怎么说呢。问题再一个很奇妙的地方，出现在这里：

<!DOCTYPE configuration>

这是我贴的一个配置代码。本来很长的，但是只有这一段有问题，这一段是上下添加了 ` ``` ` 符号才会出问题

![](https://hiki-blog.oss-cn-shenzhen.aliyuncs.com/githubio/bug/bug_20191009105153.png?Expires=1570593133&OSSAccessKeyId=TMP.hVHzoQPphgabRGiuVhq38Pt5Qtj3VwWhUubzWVbjNiJtAaDBYAkqKaSoMTVC12gsYiRsCuaRU3BMpz5irzyTZVdAffYKFm3sbZkqdWHtK1iYeLEmSgKpHJRURgZbmc.tmp&Signature=ejLiijl3iaGham7Xoaxs8Fv0rF8%3D)

无论我测试 <!TYPE configuration> 还是 <!DOCTYPE config> 都没有问题！ 

**只有是 <!DOCTYPE configuration> 的时候会出问题**

把这段代码删除即可。

后来看到Github Pages Dependency versions 最后更新时间再 2019.10.07 。也许就是版本更新，有些bug出现了

![](https://hiki-blog.oss-cn-shenzhen.aliyuncs.com/githubio/bug/bug_20191009110106.png?Expires=1570594869&OSSAccessKeyId=TMP.hVHzoQPphgabRGiuVhq38Pt5Qtj3VwWhUubzWVbjNiJtAaDBYAkqKaSoMTVC12gsYiRsCuaRU3BMpz5irzyTZVdAffYKFm3sbZkqdWHtK1iYeLEmSgKpHJRURgZbmc.tmp&Signature=cU3Biodw84sTRAP3h4EBV7w1UMI%3D)

未找到解决方案。。希望有人能解决

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.10.09  
> 更新日期：2019.10.09