---
layout: post
title:  "node.js学习记录：Promise函数"
date:   2019-04-27 16:34:00 +0200
categories: nodejs
tagg: node.js
excerpt: Promise是一个构造函数，方法有all、reject、resolve这几个，原型上有then、catch等方法。
---

# Promise函数

Promise是一个构造函数，方法有all、reject、resolve这几个，原型上有then、catch等方法。

## 构造方法
```
var promise = new Promise(function(resolve, reject){

//异步操作

setTimeout(function(){

            console.log('执行完毕！');

            resolve('xxx');

        }, 2000);

    }
);
```
Promise的构造函数接收一个函数，并且传入两个参数：resolve，reject，分别表示异步操作执行成功后的回调函数和异步操作执行失败后的回调函数.  
resolve是将Promise的状态置为fullfiled，reject是将Promise的状态置为rejected。  

一般将Promise包在一个函数中，以为new的时候，里面的代码就执行了。如：
```
function runAsync(){

    var promise = new Promise(function(resolve, reject){

        setTimeout(function(){

            console.log('执行完毕);

            resolve('xxx');

        }, 2000);

    });
    return promise;
}
```

## then
`then` 是`promise`对象的方法，意思就是，`promise`对象处理完后，执行`then`方法里的方法。
```
runAsync().then(function(data){

    console.log(data);

    //这里用传过来的数据做其他操作

});

```
`then`方法可以接收两个参数，第一个是`resolve`的回调，第二个是`reject`的回调，具体看下面

## reject用法  
```
function getNumber(){

    var promise = new Promise(function(resolve, reject){

        //异步操作

        setTimeout(function(){

            var num = Math.ceil(Math.random()*10); //生成1-10的随机数

            if(num<=5){

                resolve(num);

            }

            else{

                reject('数字大了');

            }

        }, 2000);

    });
    return promise;
}
```
使用时判断：
```
getNumber().then(

    function(data){

        console.log('resolved');

        console.log(data);
    },

    function(reason, data){

        console.log('rejected');

        console.log(reason);
    }
);
```
`reason`是`reject`传入的参数

## catch
和`then`方法的第二个参数一样，用来指定reject回调。用法：
```
getNumber().then(
    function(data){

        console.log('resolved');

        console.log(data);

    }
).catch(function(reason){

    console.log('rejected');

    console.log(reason);

});
```
和`then`方法的第二个参数不同的地方在于，如果`then`方法的第一个参数（回调）抛出异常时，也会执行catch中的方法而不会卡死。

## all
Promise的all方法提供并行执行异步操作的能力，并且在所有一步操作执行完成后才执行回调：
```
Promise.all([runAsync1(), runAsync2(), runAsync3()]).then(
    function(results){
        console.log(results);
    }
);
```
结果：
```
异步1完成

异步2完成

异步3完成

[“xxx1”,”xxx2”,”xxx3”]
```

## race
和all方法基本一致，但是回调函数只作用于最快完成的那个操作。
```
Promise.race([runAsync1(), runAsync2(), runAsync3()]).then(
    function(results){
        console.log(results);
    }
);
```
将runAsync1的延迟改为1秒结果：
```
异步1完成

xxx1

异步2完成

异步3完成
```
race给某个异步请求设置超时时间，并且在超时后执行相应的操作，代码如下：
```
//请求某个图片资源

function requestImg(){

    var promise = new Promise(
        function(resolve, reject){
            var img = new Image();
            img.onload = function(){
                resolve(img);
            }
            img.src = 'xxx';
        }
    );
    return promise;
}


//延时函数，用于给请求计时

function timeout(){

    var promise = new Promise(
        function(resolve, reject){
            setTimeout(
                function(){
                    reject('图片请求超时');
                }, 5000);
    });
    return promise;
}


Promise.race([requestImg(), timeout()]).then(
    function(results){
        console.log(results);
    }).catch(
        function(reason){
            console.log(reason);
        }
    );
```

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.04.27  
> 更新日期：2019.04.27
