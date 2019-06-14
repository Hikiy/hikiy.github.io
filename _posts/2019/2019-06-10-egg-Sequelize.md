---
layout: post
title:  "egg.js学习记录：Sequelize"
date:   2019-06-10 15:48:00 +0200
categories: eggjs
excerpt: 在 Node.js 社区中，sequelize是一个广泛使用的 ORM 框架，它支持 MySQL、PostgreSQL、SQLite 和 MSSQL 等多个数据源。
tagg: nodeframwork
---

在 Node.js 社区中，sequelize是一个广泛使用的 ORM 框架，它支持 MySQL、PostgreSQL、SQLite 和 MSSQL 等多个数据源。

> [egg.js的Sequelize官方文档](https://eggjs.org/zh-cn/tutorials/sequelize.html)  
> [Sequelize官方文档](http://docs.sequelizejs.com/)  
> [Sequelize中文文档](https://github.com/demopark/sequelize-docs-Zh-CN)

## 初始化
安装并配置 egg-sequelize 插件（它会辅助我们将定义好的 Model 对象加载到 app 和 ctx 上）和 mysql2 模块：

- 安装
```
npm install --save egg-sequelize mysql2
```

- 在 config/plugin.js 中引入 egg-sequelize 插件
```
exports.sequelize = {
  enable: true,
  package: 'egg-sequelize',
};
```

- `config/config.default.js`中编写sequelize配置：
```
config.sequelize = {
        dialect: 'mysql',
        host: '',
        port: '',
        username: '',
        password: '',
        database: '',
        define: {
            timestamps: false, // 禁用sequelize默认时间戳设置
            freezeTableName: true, // 禁用sequelize默认给表名设置复数
        },
    };
```
## 编写代码
### Model

```
'use strict';

module.exports = app => {
    const { INTEGER } = app.Sequelize;

    const OrderGroupbuyTagsModel = app.model.define('order_groupbuy_tags', {
        ogtid: { type: INTEGER, primaryKey: true, autoIncrement: true },
        partner_code: INTEGER,
        gbid: INTEGER,
        leader_uid: INTEGER,
        status: INTEGER,
        created: INTEGER,
        updated: INTEGER,
    });

    return OrderGroupbuyTagsModel;
};

```

### 查询

```
const Service = require('egg').Service;
const Op = require('sequelize').Op;
.
.
//用来关联两张表的
this.ctx.model.GroupbuyModel.hasMany(this.ctx.model.OrderGroupbuyTagsModel, { foreignKey: 'gbid', sourceKey: 'gbid' });
this.ctx.model.OrderGroupbuyTagsModel.belongsTo(this.ctx.model.GroupbuyModel, { foreignKey: 'gbid', targetKey: 'gbid' });

let groupbuyTimeoutList = await this.ctx.model.OrderGroupbuyTagsModel.findAll({
    //属性
    attributes: [ 'ogtid', 'gbid', 'groupbuy.amount_limit' ],

    //连接表
    include: {
        model: this.ctx.model.GroupbuyModel,
        attributes: [],
    },

    where: {
        created: { [Op.lt]: Math.round(new Date().getTime()/1000) - 1200 }, //20分钟
        status: 1,
    },

    limit: 20,
    order: [
        [ 'ogtid', 'ASC' ]
    ],
    //原始查询
    raw: true,
});
```

- 其中`hasMany`和`belongsTo`是用于**关联**两个表

> belongsTo - 属于。本例中意思是OrderGroupbuyTagsModel属于GroupbuyModel。外键在OrderGroupbuyTagsModel上
>
> hasMany - 拥有许多。本例中意思是GroupbuyModel有很多个OrderGroupbuyTagsModel。
>
> hasOne - 拥有一个。



- 其中`[Op.lt]`为**操作符**,可用于创建更复杂比较的符号运算符:

```
const Op = Sequelize.Op

[Op.and]: {a: 5}           // 且 (a = 5)
[Op.or]: [{a: 5}, {a: 6}]  // (a = 5 或 a = 6)
[Op.gt]: 6,                // id > 6
[Op.gte]: 6,               // id >= 6
[Op.lt]: 10,               // id < 10
[Op.lte]: 10,              // id <= 10
[Op.ne]: 20,               // id != 20
[Op.eq]: 3,                // = 3
[Op.not]: true,            // 不是 TRUE
[Op.between]: [6, 10],     // 在 6 和 10 之间
[Op.notBetween]: [11, 15], // 不在 11 和 15 之间
[Op.in]: [1, 2],           // 在 [1, 2] 之中
[Op.notIn]: [1, 2],        // 不在 [1, 2] 之中
[Op.like]: '%hat',         // 包含 '%hat'
[Op.notLike]: '%hat'       // 不包含 '%hat'
[Op.iLike]: '%hat'         // 包含 '%hat' (不区分大小写)  (仅限 PG)
[Op.notILike]: '%hat'      // 不包含 '%hat'  (仅限 PG)
[Op.startsWith]: 'hat'     // 类似 'hat%'
[Op.endsWith]: 'hat'       // 类似 '%hat'
[Op.substring]: 'hat'      // 类似 '%hat%'
[Op.regexp]: '^[h|a|t]'    // 匹配正则表达式/~ '^[h|a|t]' (仅限 MySQL/PG)
[Op.notRegexp]: '^[h|a|t]' // 不匹配正则表达式/!~ '^[h|a|t]' (仅限 MySQL/PG)
[Op.iRegexp]: '^[h|a|t]'    // ~* '^[h|a|t]' (仅限 PG)
[Op.notIRegexp]: '^[h|a|t]' // !~* '^[h|a|t]' (仅限 PG)
[Op.like]: { [Op.any]: ['cat', 'hat']} // 包含任何数组['cat', 'hat'] - 同样适用于 iLike 和 notLike
[Op.overlap]: [1, 2]       // && [1, 2] (PG数组重叠运算符)
[Op.contains]: [1, 2]      // @> [1, 2] (PG数组包含运算符)
[Op.contained]: [1, 2]     // <@ [1, 2] (PG数组包含于运算符)
[Op.any]: [2,3]            // 任何数组[2, 3]::INTEGER (仅限PG)
[Op.col]: 'user.organization_id' // = 'user'.'organization_id', 使用数据库语言特定的列标识符, 本例使用 PG
```

- 其中`raw`设置为`true`为使用**原始查询**。有时候,你可能会期待一个你想要显示的大量数据集,而无需操作. 对于你选择的每一行,Sequelize 创建一个具有更新,删除和获取关联等功能的实例.如果你有数千行,则可能需要一些时间. 如果你只需要原始数据,并且不想更新任何内容,你可以这样做来获取原始数据.

```
// 你期望从数据库的一个巨大的数据集,
// 并且不想花时间为每个条目构建DAO？
// 你可以传递一个额外的查询参数来取代原始数据:
Project.findAll({ where: { ... }, raw: true })
```

**重命名**

将`bar`重命名为`baz`：

**聚合**
```
Model.findAll({
  attributes: [[sequelize.fn('COUNT', sequelize.col('hats')), 'no_hats']]
});
```
必须给它一个别名,以便能够从模型中访问它. 在上面的例子中,你可以使用 instance.get('no_hats') 获得帽子数量.

**排除一些指定的表字段**
```
Model.findAll({
  attributes: { exclude: ['bar'] }
});
```

```
Model.findAll({
  attributes: ['foo', ['bar', 'baz']]
});
```
### 更新

```
await this.ctx.model.OrderGroupbuyTagsModel.update({
    status: orderGroupbuyTagsStatus,
    updated: Math.round(new Date().getTime()/1000)
}, {
    where: {
        ogtid: ogtid
    }
});
```

### 新增

```
await this.ctx.model.RefundsModel.create({
    partner_code: orderInfo[i].partner_code,
    oid: orderInfo[i].oid,
    status: 1,
    created: Math.round(new Date().getTime()/1000),
    updated: Math.round(new Date().getTime()/1000),
});
```

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.06.10  
> 更新日期：2019.06.10
