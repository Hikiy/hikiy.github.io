---
layout: post
title:  "计算机网络：Get 和 Post区别"
date:   2018-09-30 16:00:00 +0200
categories: 计算机网络
excerpt: 
tagg: network
---

# 计算机网络：Get 和 Post区别

- GET - 从指定的资源请求数据。
- POST - 向指定的资源提交要被处理的数据


## Get

请求的 URL 中发送

- 1.GET 请求可被缓存
- 2.GET 请求保留在浏览器历史记录中
- 3.GET 请求可被收藏为书签
- 7.GET 请求不应在处理敏感数据时使用
- 5.GET 请求有长度限制
- 6.GET 请求只应当用于取回数据


## Post

请求的 HTTP 消息主体中发送

- 1.POST 请求不会被缓存
- 2.POST 请求不会保留在浏览器历史记录中
- 3.POST 不能被收藏为书签
- 4.POST 请求对数据长度没有要求
- 5.POST 比 GET 更安全，因为参数不会被保存在浏览器历史或 web 服务器日志中。


<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2018.09.30  
> 更新日期：2018.09.30

<center>(<font color=red size=2>转载文章请注明作者和出处 </font><a href="https://github.com/Hikiy">Hiki)</a></center>  