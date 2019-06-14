---
layout: post
title:  "egg.js学习记录：快速入门"
date:   2019-04-26 19:17:00 +0200
categories: eggjs
tagg: nodeframwork
excerpt: 构建egg.js项目
---

# 快速入门
先[安装node.js](https://nodejs.org)，然后直接cmd，生成项目：
```
$ mkdir egg-example && cd egg-example
$ npm init egg --type=simple
$ npm i
```
PS:`npm i `即`npm install`

启动项目：
```
$ npm run dev
$ open localhost:7001
```
PS： 在公司拉代码后不要忘了`npm install`安装环境

**一些web的写法写在了**`编程语言/node.js`**的[笔记](https://github.com/Hikiy/Notes/tree/master/%E7%BC%96%E7%A8%8B%E8%AF%AD%E8%A8%80/Node.js)里**

----
- 通过上面方法新建的项目目录：

```
project
  |
  └app
  |  └controller
  |  |     └home.js
  |  └router.js
  |
  └config
  |  └config.default.js
  |  └plugin.js
  |
  └test
  |  └app
  |    └controller
  |      └home.js
  |
  └.autod.conf.js
  └.eslintignore
  └.eslintrc
  └.gitignore
  └package.json
  └READNE.md
```

- /app规范：

```
project
  |
  └app
     └controller
     └helper
     └middleware
     └model
     └public
     └schedule
     └service
     |
     └router.js

```
- 官方规范

```
egg-project
├── package.json
├── app.js (可选)
├── agent.js (可选)
├── app
|   ├── router.js
│   ├── controller
│   |   └── home.js
│   ├── service (可选)
│   |   └── user.js
│   ├── middleware (可选)
│   |   └── response_time.js
│   ├── schedule (可选)
│   |   └── my_task.js
│   ├── public (可选)
│   |   └── reset.css
│   ├── view (可选)
│   |   └── home.tpl
│   └── extend (可选)
│       ├── helper.js (可选)
│       ├── request.js (可选)
│       ├── response.js (可选)
│       ├── context.js (可选)
│       ├── application.js (可选)
│       └── agent.js (可选)
├── config
|   ├── plugin.js
|   ├── config.default.js
│   ├── config.prod.js
|   ├── config.test.js (可选)
|   ├── config.local.js (可选)
|   └── config.unittest.js (可选)
└── test
    ├── middleware
    |   └── response_time.test.js
    └── controller
        └── home.test.js
```

- Docker

```
FROM node:8.16.0-alpine

RUN mkdir -p /app

WORKDIR /app

COPY ./ /app/

RUN npm i --production --registry=https://registry.npm.taobao.org

EXPOSE 8000

CMD npm run docker
```
> 其中`RUN npm i --production --registry=https://registry.npm.taobao.org`为淘宝镜像

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.04.26  
> 更新日期：2019.06.13
