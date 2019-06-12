---
layout: post
title:  "test"
date:   2019-06-12 14:19 +0200
categories: test 
---
# Win10上安装Docker  
- 下载Docker，[官网下载](https://www.docker.com/get-started)
- 安装
- 开启win10专业版系统的Hyper-V
>右键 ‘开始’ -> 应用和功能 ->启用或关闭Windows功能 -> 将Hyper-V的都勾上
>![image](https://note.youdao.com/yws/public/resource/2c26474e768cbdac36ea44fafdbd1946/xmlnote/366430551A374AD5AFEA2745B5E0087F/13445)
>![image](https://note.youdao.com/yws/public/resource/2c26474e768cbdac36ea44fafdbd1946/xmlnote/E1470E37ECFF4E9195B2BB8A247488F7/13447)
>![image](https://note.youdao.com/yws/public/resource/2c26474e768cbdac36ea44fafdbd1946/xmlnote/4C5862E5752E48628B2568D89F01E576/13449)

- 启动桌面安装好的DockerDesktop
- 然后会发现如下问题（当然如果没有遇到，那恭喜你）
![image](https://note.youdao.com/yws/public/resource/2c26474e768cbdac36ea44fafdbd1946/xmlnote/DE54EFB238204166AB79E5C5D4CA4B93/13470)
- 要在bios界面中打开虚拟化
- 网上很多说法是在configuration之类的设置中，然而在公司遇到的ASRock主板的命名不一样，找了很久，要一个个看，最后找到的是因为右边详情里写着vitual字样，最后才打开成功
- 在使用指令pull 的时候回报错：
```
Error response from daemon: Get https://registry-1.docker.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```
- 解决方案：
![image](https://note.youdao.com/yws/public/resource/2c26474e768cbdac36ea44fafdbd1946/xmlnote/7373C170EF96495ABC8D77BC5E29A624/13520)
- 到此安装成功
# 镜像加速器

### Windows
&emsp; 对于使用 Windows 10 的系统，在系统右下角托盘 Docker 图标内右键菜单选择 Settings，打开配置窗口后左侧导航菜单选择 Daemon。在 Registry mirrors 一栏中填写加速器地址 https://registry.docker-cn.com，之后点击 Apply 保存后 Docker 就会重启并应用配置的镜像地址了。   
&emsp;可用国内镜像加速：
- Docker 官方提供的中国 registry mirror https://registry.docker-cn.com
- [阿里云加速器(需登录账号获取)](https://cr.console.aliyun.com/cn-hangzhou/mirrors)
- 七牛云加速器 https://reg-mirror.qiniu.com/

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.04.17  
> 更新日期：2019.04.17

<center>(<font color=red size=2>转载文章请注明作者和出处 </font><a href="https://github.com/Hikiy">Hiki)</a></center>  