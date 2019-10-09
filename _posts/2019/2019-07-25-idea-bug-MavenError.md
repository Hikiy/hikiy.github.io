---
layout: post
title:  "IDEA错误记录： Maven错误汇总"
date:   2019-07-25 17:02:00 +0200
categories: IDEA工具
excerpt: 
tagg: idea
---



# Maven错误汇总

## pom.xml中的包没有导入，但是没有报错
新建spring cloud 的注册中心时，发现依赖包并没有导入，搞了好几个小时。

### 解决方案
具体原因没有查明，只是最后解决了

过程：
- 点击idea菜单栏->View->Tools Windows->Maven
- 发现spring的依赖包都是红色波浪
- 但是不知道哪里看发生了什么

解决步骤
- 找到maven放依赖包的地方（File -> Settings-> Build -> Build Tools -> Maven）
- 删除出现波浪的文件
- 右键项目->Maven->Reimport


<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.07.25  
> 更新日期：2019.07.25
