---
layout: post
title:  "ThinkPHP学习记录：redis操作"
date:   2019-05-14 17:27:00 +0200
categories: ThinkPHP
excerpt: 
tagg: PHP
---

# redis

### 普通键值操作
**配置文件**  
redis.php:
```
<?php

return [

    'default' => [
        // 驱动方式
        'type'   => 'Redis',
        // 缓存服务器地址
        'host'   => '',
        // 密码
        'password'   => '',
        // 端口
        'port'   => '',
        // 缓存前缀
        'prefix' => '',
    ],

];
```
config.php:
```
$redis = require('redis.php');
'cache' => $redis['default'],
```
**测试**
```
use think\Cache;

public function test(){
        $result = Cache::set('test','hiki');//成功返回true
        return OutputHelper::success_return($result);
    }
    public function testrm(){
        $result = Cache::rm('test');//成功返回1，没有返回0；
        return OutputHelper::success_return($result);
    }
    public function tests(){
        $result = Cache::get('test');//失败返回false
        return OutputHelper::success_return($result);
    }
```
### 队列操作  
配置：
```
    $redis = new \Redis();
    $redis->connect($this->host, $this->port);
    $redis->auth($this->pwd);
```
测试：
```
$redis->lLen($key);//获取队列长度
$redis->lPush($key, $v['id']);//左push操作
$redis->rPop($key);//右pop操作，左push 右pop为先进先出
```

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.05.14  
> 更新日期：2019.05.14
