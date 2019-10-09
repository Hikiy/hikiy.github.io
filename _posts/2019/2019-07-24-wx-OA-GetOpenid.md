---
layout: post
title:  "微信公众号开发： 静默获取openid"
date:   2019-07-24 15:11:00 +0200
categories: 微信公众号开发
excerpt: 
tagg: wx
---

# 静默获取openid

静默获取指的是不需要微信用户点击授权，即可获取微信的openid

## openid
是微信用户在**当前公众号**中的**唯一标识**，也就是说在别的公众号上获取的openid是不一样的。

## 步骤
分为两步：

- 获取code
- 通过code获取openid

### 获取code
有两种方式

- 需要用户同意授权的获取code
- 不需要用户同意授权的获取code

#### 获取code的URL
```
https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE#wechat_redirect
```

需要自己填的参数：
- `appid`: 微信公众平台->基本配置 中获取
- `redirect_uri`：回调地址，回调地址必须是微信公众平台->基本配置->服务器配置 中的域名。PS：可以带参数
- `scope`：应用授权作用域，`snsapi_base` （不弹出授权页面，直接跳转，只能获取用户openid），`snsapi_userinfo` （弹出授权页面，可通过openid拿到昵称、性别、所在地。并且， 即使在未关注的情况下，只要用户授权，也能获取其信息 ）

该URL其实是将获取的openid作为参数回调我们给的回调地址中。

**所以这个URL是重定位才能生效！而不是调用！最终重定向到 回调地址`redirect_uri`！在回调的方法中处理code。**

最好的解决方案是，回调地址设置为通过code获取openid的方法。

> 经过测试，code只能使用一次，用code获取openid之后将失效

### 通过code获取openid

#### 通过code获取openid的接口

```
https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code
```

需要自己填的参数：
- `appid`：微信公众平台->基本配置 中获取
- `secret`：微信公众平台->基本配置 中获取
- `code`：从上面获取code的URL获取

这个接口跟获取code不同了，是调用的接口，不是重定向。返回一个JSON。

```
{
    "access_token":"ACCESS_TOKEN",
    "expires_in":7200,
    "refresh_token":"REFRESH_TOKEN",
    "openid":"OPENID",
    "scope":"SCOPE" 
 }
```

返回的JSON中包含：
- `access_token`：网页授权接口调用凭证,注意：此access_token与基础支持的access_token不同
- `expires`：access_token接口调用凭证超时时间，单位（秒）
- `refresh_token`：用户刷新access_token
- `openid`：用户唯一标识，请注意，在未关注公众号时，用户访问公众号的网页，也会产生一个用户和公众号唯一的OpenID
- `scope`：用户授权的作用域，使用逗号（,）分隔。

错误时返回的JSON：
```
{"errcode":40029,"errmsg":"invalid code"}
```

## 综合使用

获取code是重定向，获取openid是接口调用。

所以这两个可以这样结合：
将获取code的重定向到获取openid的接口中

### 代码示例
这里用node.js写的代码，原理是一样的

```
//http://hiki.com/getCode
async getCode(){
    let appid = APPID;
    let redirect = REDIRECT; //回调地址，例如 http://hiki.com/getopenid
    redirect = encodeURI(redirect); //这里是转义，如果REDIRECT是转义好的不需要这一步
    
    //重定向URL
    await ctx.redirect('https://open.weixin.qq.com/connect/oauth2/authorize?appid=${appid}&redirect_uri=${redirect}&response_type=code&scope=snsapi_base&state=STATE#wechat_redirect')
}

//http://hiki.com/getopenid
async getOpenid(){
    //获取作为传过来的code
    let code = this.ctx.query.code;
    
    let appid = APPID;
    let secret = SECRET;
    
    //调用微信的接口获取JSON
    const access_token = await ctx.app.curl('https://api.weixin.qq.com/sns/oauth2/access_token?appid=${appid}&secret=${secret}&code=${code}&grant_type=authorization_code')
    
    let openid = access_token.openid;
}
```

应该也可以都写在一个方法里，然后回调到原方法，在方法中判断是否有code传入即可，不过如果判断不好就是死循环哦!

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.07.24  
> 更新日期：2019.07.24
