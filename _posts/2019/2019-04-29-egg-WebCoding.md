---
layout: post
title:  "egg.js学习记录：Web写法"
date:   2019-04-29 15:00:00 +0200
categories: eggjs
excerpt: Web各层写法
tagg: nodeframwork
---

## 中间件写法

### 1.app/middleware中新建一个js，例如hiki_middleware：
```
'use strict';

const Output = require('../helper/output_helper');

/**
 * 测试拦截中间件
 *
 * @author Hiki
 */

module.exports = () => {
    return async function(ctx, next) {
        let requestQuery = ctx.query;

        if ( requestQuery.hiki == undefined || requestQuery.hikisin == undefined) {
            ctx.body = Output.miss_params();
        } else {
            if( requestQuery.hiki == 'hiki' && requestQuery.hikisin == 'cool'  ){
                await next();
            } else {
                ctx.body = Output.success_return('nonono');
            }
        }
    }

}
```
**注意```return```要加```async```,中间件通过后要```await next()```**
### 2.在配置文件中配置中间件：
```
config.middleware = [
        'hikiMiddleware',
    ];
```

## Controller层写法
app/controller/hikitest_controller:
```
'use strict';

const Controller = require('egg').Controller;
const Output = require('../helper/output_helper');

class HikitestController extends Controller{
    async gethikiname(){
        const { ctx } = this;
        //const ctx = this.ctx;

        let name = ctx.query.name;
        let hikiname = 'hiki';

        if( name != undefined ){
            hikiname = 'no,his name is hiki';
        }

        ctx.body = Output.success_return(hikiname);
    }
}

module.exports = HikitestController;
```
**注意末尾需要输出**

## MySQL用法
### 安装与配置
**安装egg-mysql插件：**
```
$ npm i --save egg-mysql
```
**在`config/plugin.js`开启插件：**
```
exports.mysql = {
  enable: true,
  package: 'egg-mysql',
};
```
**在`config/config.${env}.js`配置单数据库连接信息：**
```
exports.mysql = {
  // 单数据库信息配置
  client: {
    // host
    host: 'mysql.com',
    // 端口号
    port: '3306',
    // 用户名
    user: 'test_user',
    // 密码
    password: 'test_password',
    // 数据库名
    database: 'test',
  },
  // 是否加载到 app 上，默认开启
  app: true,
  // 是否加载到 agent 上，默认关闭
  agent: false,
};
```
**使用方法**
```
await app.mysql.query(sql,vaules);
```
**配置多数据库连接信息：**
```
exports.mysql = {
  clients: {
    // clientId, 获取client实例，需要通过 app.mysql.get('clientId') 获取
    db1: {
      // host
      host: 'mysql.com',
      // 端口号
      port: '3306',
      // 用户名
      user: 'test_user',
      // 密码
      password: 'test_password',
      // 数据库名
      database: 'test',
    },
    db2: {
      // host
      host: 'mysql2.com',
      // 端口号
      port: '3307',
      // 用户名
      user: 'test_user',
      // 密码
      password: 'test_password',
      // 数据库名
      database: 'test',
    },
    // ...
  },
  // 所有数据库配置的默认值
  default: {

  },

  // 是否加载到 app 上，默认开启
  app: true,
  // 是否加载到 agent 上，默认关闭
  agent: false,
};
```
使用方法：  
```
const client1 = app.mysql.get('db1');
await client1.query(sql, values);

const client2 = app.mysql.get('db2');
await client2.query(sql, values);
```
### 编写CRUD语句
**insert**
```
const result = await this.app.mysql.insert('posts', { title: 'Hello World' });
// 在 post 表中，插入 title 为 Hello World 的记录

=> INSERT INTO `posts`(`title`) VALUES('Hello World');
//判断插入成功
const insertSuccess = result.affectedRows === 1;
```
**get和select**
- 查询一条记录
```
const post = await this.app.mysql.get('posts',{ id:12 });
=> SELECT * FROM `posts` WHERE `id` = 12 LIMIT 0, 1;
```
- 查询全表
```
const post = await this.app.mysql.select('posts');
-> SELECT * FROM `posts`;
```

**update**
```
// 修改数据，将会根据主键 ID 查找，并更新
const row = {
  id: 123,
  name: 'fengmk2',
  otherField: 'other field value',    // any other fields u want to update
  modifiedAt: this.app.mysql.literals.now, // `now()` on db server
};
const result = await this.app.mysql.update('posts', row); // 更新 posts 表中的记录

=> UPDATE `posts` SET `name` = 'fengmk2', `modifiedAt` = NOW() WHERE id = 123 ;

// 判断更新成功
const updateSuccess = result.affectedRows === 1;

// 如果主键是自定义的 ID 名称，如 custom_id，则需要在 `where` 里面配置
const row = {
  name: 'fengmk2',
  otherField: 'other field value',    // any other fields u want to update
  modifiedAt: this.app.mysql.literals.now, // `now()` on db server
};

const options = {
  where: {
    custom_id: 456
  }
};
const result = await this.app.mysql.update('posts', row, options); // 更新 posts 表中的记录

=> UPDATE `posts` SET `name` = 'fengmk2', `modifiedAt` = NOW() WHERE custom_id = 456 ;

// 判断更新成功
const updateSuccess = result.affectedRows === 1;
```
**delete**
```
const result = await this.app.mysql.delete('posts', {
  author: 'fengmk2',
});

=> DELETE FROM `posts` WHERE `author` = 'fengmk2';
```
### 直接sql语句
```
const postId = 1;
const results = await this.app.mysql.query('update posts set hits = (hits + ?) where id = ?', [1, postId]);

=> update posts set hits = (hits + 1) where id = 1;
```


​    
## 事务
### 手动控制
```
const conn = await app.mysql.beginTransaction(); // 初始化事务

try {
  await conn.insert(table, row1);  // 第一步操作
  await conn.update(table, row2);  // 第二步操作
  await conn.commit(); // 提交事务
} catch (err) {
  // error, rollback
  await conn.rollback(); // 一定记得捕获异常后回滚事务！！
  throw err;
}
```

## 路由配置
app/router.js:
```
'use strict';

/**
 * @param {Egg.Application} app - egg application
 */
module.exports = app => {
    router.get('/gethikiname', controller.hikitestController.gethikiname);
};

```

## HTTP请求
### GET请求
- Controller层

```
'use strict';

const Controller = require('egg').Controller;
const Output = require('../helper/output_helper');

class HikitestController extends Controller{
    async gethikiname(){
        // const { ctx } = this;
        const ctx = this.ctx;

        let name = ctx.query.name;
        let hikiname = 'hiki';

        if( name != undefined ){
            hikiname = 'no,his name is hiki';
        }

        ctx.body = Output.success_return(hikiname);
    }
}

module.exports = HikitestController;
```

- 路由配置

```
'use strict';

/**
 * @param {Egg.Application} app - egg application
 */
module.exports = app => {
    const { router, controller } = app;
    router.get('/gethikiname', controller.hikitestController.gethikiname);
};
```
### POST请求

- Controller层

```
'use strict';

const Controller = require('egg').Controller;
const Output = require('../helper/output_helper');

class HikitestController extends Controller{
    async gethikiname(){
        // const { ctx } = this;
        const ctx = this.ctx;

        let name = ctx.request.body.name;
        let hikiname = 'hiki';

        if( name != undefined ){
            hikiname = 'no,his name is hiki';
        }

        ctx.body = Output.success_return(hikiname);
    }
}

module.exports = HikitestController;
```

- 路由配置

```
'use strict';

/**
 * @param {Egg.Application} app - egg application
 */
module.exports = app => {
    const { router, controller } = app;
    router.post('/gethikiname', controller.hikitestController.gethikiname);
};
```

- 然后测试报错

```
`invalid csrf token`
```
- 原因是egg这个框架在安全方面做了处理，这个csrf是用来判断这个post请求是否来自原url。
- 在config.default.js 中禁用 csrf
```
config.security = {
        csrf:{
            enable:false
        }
    }
```

- 然后用postman发送post请求发现post传过去的值为undefined
- 需要使用另一个格式传值，使用x-www-form-urlencoded传值，或者说headers加上`[{"key":"Content-Type","value":"application/x-www-form-urlencoded"}]`
- 成功

<br /><br /><br /><br />

> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.04.29  
> 更新日期：2019.06.24
