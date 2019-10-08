---
layout: post
title:  "博客改进：添加自己的logo SVG转CSS"
date:   2019-06-14 15:57:00 +0200
categories: 博客改进
excerpt: 给自己的博客多点装饰
---

## 博客改进：添加自己的logo SVG转CSS

原本博客的header用着github的图标，查看源码发现是一个class，一直看源码原以为是要去了解`<glyph>`标签，查资料发现这是个很复杂的东西。很久没研究清楚。换了个方式搜索css、字体之类的。发现这东西可以是一个svg转css的过程，这个东西好像是叫字体图标的东西，比较节省性能好像。


### 获得喜欢的svg图标
推荐：https://www.iconfont.cn/ 注意版权问题

### 生成CSS文件

- 访问 https://icomoon.io/ （等待时间巨久，原因未知，或者别的svg转css都可以）
- 点击右上角的IcoMoon App( https://icomoon.io/app/ )
- 点击import Icons上传svg
- 选中上传的图标，点击右下角的Generate Font生成文件
- 点击上方的Preferences进行设置
- 点击右下角的Download下载

### 使用CSS
- 只需要压缩文件中的`style.css`和`fonts`文件夹
- 将它放进css文件夹中（这个博客则是放在`bower_components\octicons\octicons\`中）
- 在需要使用图标的地方编写代码：

```
<span class="icon-logo"></span>
```

- 注意class的名字是生成的时候可以设置的，如果忘记了可以看压缩包中的demo

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.06.14  
> 更新日期：2019.06.14