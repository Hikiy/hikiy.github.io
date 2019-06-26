---
layout: post
title:  "解决方案：登录验证 logintoken方案"
date:   2019-05-08 17:17:00 +0200
categories: 解决方案
excerpt: 采用egg.js框架下的node.js实现
---

# 登录验证 logintoken方案

采用egg.js框架下的node.js实现

## 整体思路

### 登录

- 用户登录时，通过密码加密方案（请看[另一篇内容](https://hikiy.github.io/%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/2019/05/08/solution-%E5%AF%86%E7%A0%81%E5%8A%A0%E5%AF%86%E6%96%B9%E6%A1%88.html)）验证密码是否正确。
- 密码正确，随机生成login_token字符串。
- 以 专用名+login_token 命名redis的key。将需要的用户信息设置到value中，并添加过期时间

登录接口：
```
async login(username, password){
        /**
        /*先通过密码验证
        **/

        let strs = "jalsdjflasjdfladsjflajsdlfjalfjalsdfjjfl";
        let length = 32;
        let token = "";

        //从strs取随机32个字符
        for (let i = 0; i < length; i++) {
            token += strs.charAt(Math.floor(Math.random() * strs.length));
        }

        let user = {
            uid: key[0].dataValues.uid
        };
        let userinfo = JSON.stringify(user);

        await this.app.redis.set("HIKI_LOGIN_TOKEN:" + token, userinfo);
        await this.app.redis.expire("HIKI_LOGIN_TOKEN:" + token, 43200); //半天

        result.logintoken = token;
        return result;
    }
```

### 登录拦截

在egg.js中有中间件。可以使用中间件进行登录验证，所以：
- 在中间件中获取get参数login_token
- 从redis中查询该login_token的字段
- 如果存在，则获取value值，以便使用

中间件：
```
module.exports = () => {

    /**
     * 登录验证拦截
     * 
     * @author Hiki
     */
    return async function(ctx, next) {
        let requestQuery = ctx.query;
        let token = requestQuery.login_token;

        if( ctx.url == "/login" ){
            //如果是登录请求则通过
            await next();
        }else{
            if ( token == undefined ) {
                //未添加login_token报错
                ctx.body = Output.miss_params();
            } else {
                let user = JSON.parse(await ctx.app.redis.get("HIKI_LOGIN_TOKEN:" + token));
                if( user == undefined ){
                    //login_token验证失败
                    ctx.body = Output.token_error();
                }else{
                    ctx.uid = user.uid;
                    await next();
                }
            }
        }
    }

}
```

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.06.26  
> 更新日期：2019.06.26
