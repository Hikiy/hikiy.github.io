---
layout: post
title:  "egg.js学习记录：Sequelize和mongoose冲突"
date:   2019-06-21 15:49:00 +0200
categories: eggjs
excerpt: 
tagg: nodeframwork
---

# Sequelize和mongoose冲突

## 问题

公司给任务要连接mysql和mongodb，mysql当时已经使用了sequelize，新加mongoose配置，然而发现冲突，搞了整整一天。

在配置文件 `config.default.js` 添加好配置：
```
config.mongoose = {
        url: 'mongodb://....',
        options: {},
    };
```

在 `plugin.js` 配置mongoose:

```
    mongoose: {
        enable: true,
        package: 'egg-mongoose',
    },

```

这时候启动项目 `npm run dev` ，就报错了：

```
nodejs.TypeError: Cannot assign to read only property 'model' of object '#<Application>'
    at AppWorkerLoader.loadToApp (D:\workspace\nodejs\ecomm-superadmin\node_modules\egg-core\lib\loader\egg_loader.js:379:39)
    at loadModelToApp (D:\workspace\nodejs\ecomm-superadmin\node_modules\egg-mongoose\lib\mongoose.js:111:14)
    at app.beforeStart (D:\workspace\nodejs\ecomm-superadmin\node_modules\egg-mongoose\lib\mongoose.js:50:7)
    at Object.callFn (D:\workspace\nodejs\ecomm-superadmin\node_modules\egg-core\lib\utils\index.js:44:42)
    at process.nextTick (D:\workspace\nodejs\ecomm-superadmin\node_modules\egg-core\lib\lifecycle.js:248:13)
    at process._tickCallback (internal/process/next_tick.js:61:11)
    at Function.Module.runMain (internal/modules/cjs/loader.js:757:11)
    at startup (internal/bootstrap/node.js:283:19)
    at bootstrapNodeJSCore (internal/bootstrap/node.js:622:3)
```

## 解决方案
### 原因
最后发现问题所在是sequelize与mongoose的冲突。冲突在model层。因为他们默认构建model都是在`app\model`下,当两个都往这里构建时就冲突了。

### 解决思路
需要将sequelize和mongoose用的model分开来

查找文档，只在sequelize文档中看到了如何指定model文件夹。可以在文档的多数据源配置中找到:

```
exports.sequelize = {
  datasources: [
    {
      delegate: 'model', // load all models to app.model and ctx.model
      baseDir: 'model', // load models from `app/model/*.js`
      database: 'biz',
      // other sequelize configurations
    },
    {
      delegate: 'admninModel', // load all models to app.adminModel and ctx.adminModel
      baseDir: 'admin_model', // load models from `app/admin_model/*.js`
      database: 'admin',
      // other sequelize configurations
    },
  ],
};
```

这个baseDir就是我们要找的指定model位置了。

mongoose使用的model还是放在原本的app\model下即可，sequelize使用的model放在新的目录下。

这个egg.js还是有坑的

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.06.21  
> 更新日期：2019.06.21
