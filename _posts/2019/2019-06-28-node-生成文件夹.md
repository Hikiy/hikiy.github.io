---
layout: post
title:  "node.js学习记录：生成文件夹"
date:   2019-06-28 16:19:00 +0200
categories: nodejs
tagg: node.js
excerpt: 
---

# 生成文件夹

### 生成文件夹

```
const fs = require('fs');

    /**
     * 生成目录
     * @author Hiki
     * 
     * @param dir 目录
     * 
     * @return Boolean
     */
    async mkdir(dir){
        if( !fs.existsSync(dir) )
            {
                try{
                    fs.mkdirSync(dir);
                    return true;
                }catch(e){
                    console.log(e);
                    return false;
                }
        }
        return true;
    }
```

### 生成多层文件夹
在node.js中，上面的方法不能一次性生成多层文件夹，必须一层一层生成

这里给出思路：

```
        let dir = `app/public/upload/${number}/pics`
        //将地址分割成迭代的数组
        let dirArr = dir.split("/");
        let length = dirArr.length;
        for( let i = 1; i < length; i++){
            dirArr[i] = dirArr[i-1] + '/' + dirArr[i];
        }
        
        for( let n in dirArr){
            await this.mkdir(dirArr[n]);
        }

        
```

**PS:这里用到了个${}是语法糖，可以拿参数，但是注意不用双引号框起来，而是用" ` "框起来的**

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.06.28  
> 更新日期：2019.06.28
