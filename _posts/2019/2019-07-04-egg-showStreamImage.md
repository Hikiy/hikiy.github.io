---
layout: post
title:  "egg.js学习记录：图片流显示到前端"
date:   2019-07-04 10:21:00 +0200
categories: eggjs
excerpt: 其实也不是什么非常难的东西，就是需要理解header。
tagg: nodeframwork
---

# 图片流显示到前端

其实也不是什么非常难的东西，就是需要理解header。

返回给前端的时候是有header的，header决定了前端显示数据的方式。如果要直接显示图片，那就需要改header头。node.js的header中的`content-type`是 `application/json` ，如果要图片显示需要改为 `image/jpeg`


```
'use strict';
const Controller = require('egg').Controller;

class CommonController extends Controller{
    /**
     * 通过OSS中的文件名显示图片
     * @author Hiki
     * 
     * @param filename 
     */
    async getPicture(){
        const ctx = this.ctx;
        let filename = ctx.query.filename;
        let stream = await this.service.uploadService.getFileStream(filename);
        ctx.set('Content-type','image/jpeg');
        ctx.body = stream.stream;
    }
}
module.exports = CommonController;
```

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.07.04  
> 更新日期：2019.07.04

<center>(<font color=red size=2>转载文章请注明作者和出处 </font><a href="https://github.com/Hikiy">Hiki)</a></center>  