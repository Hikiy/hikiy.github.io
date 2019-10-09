---
layout: post
title:  "Java知识补充：InputStream写入byte[]"
date:   2019-07-31 23:58:00 +0200
categories: java
excerpt: 
tagg: Java
---

# InputStream写入byte[]

## 使用场景
最近在尝试从阿里云对象存储OSS中获取图片流，然后再前端直接显示出来

发现要从前端直接显示图片，需要返回的是byte[]。

需要解决问题： InpuStream 转换/写 到byte[]中。
## 解决方案
直接贴代码，需要理解的话继续往下看
```
    InputStream fileStream = ossObject.getObjectContent();
    //将流处理为byte数组
    ByteArrayOutputStream output = new ByteArrayOutputStream();
        try{
            byte[] buf = new byte[1024];
            int numBytesRead = 0;
            while ((numBytesRead = fileStream.read(buf)) != -1) {
                output.write(buf, 0, numBytesRead);
            }
            bytes = output.toByteArray();
        }catch (Exception e){
            System.out.println(e.toString());
        }finally {
            output.close();
            fileStream.close();
            ossObject.close();
        }
    return bytes;
```
## 关于InputStream的读
该补课了兄弟

其实就是不太懂InputStream是怎么读的。他这个读不是一次性读完的。他是流 源源不断的流

它有几种读法
- read()
- read(byte [])
- read(byte [], off , length)

### read()
这个方法是对这个流一个一个字节的读，返回的int就是这个字节的int表示方式,如果读不到了就返回-1。

> 在探索解决方案的时候尝试过使用read()方法读取单个byte然后将byte一个个拼接为byte[]。最后得到的byte[]大小确实正确，但是在显示图片的时候无报错失败，显示的是黑屏。原因没有查明。。

### read(byte [])
这个方法是先规定一个数组长度，将这个流中的字节缓冲到数组b中，返回的这个数组中的字节个数，这个缓冲区没有满的话，则返回真实的字节个数，到未尾时都返回-1

通过查看源码发现，这个read的实现其实就是调用了下面的read(byte [], off , length)
```
read(byte[] b, 0, b.length)
```

### read(byte [], off , length)

读从off开始，到length结束

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.07.31  
> 更新日期：2019.07.31
