---
layout: post
title:  "Linux 常用指令"
date:   2019-09-17 20:20:00 +0200
categories: linux
excerpt: 
tagg: Linux
---
# Linux 常用命令
**命令格式：**

命令 [-选项] [参数]

## 索引

**目录处理命令：**  
[ls](#1)  
[pwd](#2)  
[mkdir](#3)  
[rmdir](#4)  
[cp](#5)  
[mv](#6)
[rm](#7)  

**文件处理命令:**  
[touch](#8)  
[cat](#9)  
[tac](#10)  
[more](#11)  
[less](#12)  
[tail](#13)  
[dd](#13.1)  

**查看文件磁盘使用**  
[du](#13.5)

**链接命令：**  
[ln](#14)  

**权限管理命令**  
[chmod](#15)  
[chown](#16)  
[chgrp](#17)  
[umask](#18)  

**文件搜索命令**  
[find](#19)  
[locate](#20)  
[which](#21)  
[whereis](#22)  
[grep](#23)  

**帮助命令**  
[man](#24)  
[whatis](#25)  
[apropos](#26)  
[--help](#27)(还是这个比较有用)  
[info](#28)(和man类似，看更喜欢哪种)  
[help](#29)  

**用户管理命令**  
[useradd](#30)  
[passwd](#31)  
[who](#32)  
[w](#33)  

**压缩解压命令**  
[gzip](#34)  
[gunzip](#35)  
[tar](#36)  
[zip](#37)  
[unzip](#38)  
[bzip2](#39)  
[bunzip](#40)  

**网络命令**  
[write](#41)  
[wall](#42)  
[ping](#43)  
[ifconfig](#44)  
[mail](#45)  
[last](#46)  
[lastlog](#47)  
[traceroute](#48)  
[netstat](#49)  
[setup](#50)(centos7 使用nmtui命令)  
[mount](#51)  

**关机重启命令**  
[shutdown](#52)  
[reboot](#53)  
[logout](#54)  

---

### <span id="1">ls</span>
**ls [-aldi] [文件或目录]**
- `-a` 显示所有文件，包括隐藏文件 all
- `-l` 详细信息显示 long,**可以使用 `ll` 指令代替 `l -l`**

```
-rw-------. 1 root root 1451 9月  12 00:42 anaconda-ks.cfg
```

- `-lh` 人性化显示详情 human

```
总用量 4.0K
-rw-------. 1 root root 1.5K 9月  12 00:42 anaconda-ks.cfg
```

- `-d` 查看目录属性，后面接目录可以看目录本身一般配合使用： `ls -ld /etc`
- `-i` 查看文件id

以上为例解释：

-rw-------. | 1 | root | root | 1451 | 9月  12 00:42
-- | -- | -- | -- | -- | --
文件类型 | 一种引入技术 | 所有者 | 所属组 | 大小(默认字节) | 最后一次修改时间

#### 1.文件类型

**第一个字符表示文件类型**：

``` "-" ```:表示是一个二进制文件

``` "d" ```:表示是一个目录

``` "l" ```:表示是一个软链接

**后面是三个字符为一个意思，例如：`-rw-r--r--` :**

`rw-` | `r--` | `r--`
-- | -- | --
u所有者 | g所数组 | o其它人

代表字符 | 权限 | 对文件的含义 | 对目录的含义
-- | -- | -- | --
r | 读 | 可以查看文件 | 可以列出目录中的内容
w | 写 | 可以修改文件内容 | 可以再目录中创建、修改、删除文件（即使对文件本身没有权限，也可以删除、修改）
x | 执行 | 可以执行文件 | 可以进入目录

#### 2.所有者
是文件拥有者，一般谁创建的谁就是所有者。可以转让  

#### 3.所属组
所属组是一个组，组里的用户对该文件有特殊权力

---

### <span id="2">pwd</span>
print working directory
显示当前目录
---

### <span id="3">mkdir</span>

**mkdir -p [目录名]**

- `-p` 多层创建文件夹

---

### <span id="4">rmdir</span>

**rmdir [目录名]**

删除空目录

---

### <span id="5">cp</span>

**cp -rp [源文件或目录] [目标目录]**

- `-r` 复制目录
- `-p` 保留文件属性，比如最后修改时间

可同时复制多个文件，使用空格隔开

重命名只需要在目标目录中添加一个名字即可

---

### <span id="6">mv</span>
剪切文件、改名

**mv [源文件或目录] [目标目录]**

---

### <span id="7">rm</span>
删除文件或目录

**rm -rf [文件或目录]**

- `-r` 删除目录
- `-f` 强制执行

---

### <span id="8">touch</span>
创建空文件

**touch [文件名]**

可以同时创建多个文件  用空格隔开

如果想创建有空格的文件名，需要将文件名用双引号引起来:

```
touch "hiki work"
```

---

### <span id="9">cat</span>
显示文件内容

**cat -n [文件名]**

- `-n`显示行号

---

### <span id="10">tac</span>
显示文件内容(方向列示)

**tac [文件名]**

---

### <span id="11">more</span>
分页显示文件内容

- 空格/f 翻页  
- 回车 下一行  
- q 退出
- b 回翻

**more [文件名]**

---

### <span id="12">less</span>
分页显示文件内容

- 空格/f 翻页  
- 回车 下一行  
- q 退出
- page up 回翻
- 输入 `/` + 文件名可以搜索关键词，按 `n` 显示关键词下一个位置

**more [文件名]**

---

### <span id="13">tail</span>
显示文件后面几行

**tail -nf  [文件名]**

- `-n` 指定行数
- `-f` 动态显示文件末尾内容

```
tail -n 18 /etc/services
```

---

### <span id="13.1">dd</span>
磁盘复制命令

**dd if=输入文件 of=输出文件 bs=字节数 count=个数**

将/dev/zero中文件，复制到/root/testfile中，每次传1k，传100000次：

```
dd if=/dev/zero of=/root/testfile bs=1k count=100000
```



---

### <span id="13.5">du</span>

**du [选项] [文件/文件夹]**

查看 文件/文件夹 占用大小

```
du -sh [文件/文件夹]
```

---

### <span id="14">ln</span>
生成链接文件

**ln -s [原文件] [目标文件]**

- `-s` 生成软链接，不加这个选项则生成硬连接

软链接相当于快捷方式，可以跨分区，i节点与源文件不同
硬链接相当于拥有同步更新的复制，不可跨分区，不可硬链接目录，i节点与源文件一样

---

## 权限管理命令

### <span id="15">chmod</span>
改变文件或目录权限

**ch**ange the permissions **mod**e of a file

**chmod [{ugoa}{+-=}{rwx}] [文件或目录]**  
**chmod [421] [文件或目录]**  
**chmod -R 递归修改**：也就是如果修改目录，目录下的所有文件也被修改

- u: 所有者
- g: 所属组
- o: 其它人
- a: 所有人（也就是前面三个都）

```
chmod g=rwx /tmp/myworld
```

可以通过逗号进行多次授权：
```
chmod g+w,o-r /tmp/myworld
```

使用数字来分配权限：

- r: 4
- w: 2
- x: 1

rwx = 4+2+1=7(其实就是二进制)

```
chmod 777 /tmp/myworld
```

---

### <span id="16">chown</span>
改变文件或目录的所有者

只有root有权利改变权限

**chown [用户] [文件或目录]**  

---

### <span id="17">chgrp</span>
改变文件或目录的所属组

只有root有权利改变权限

**chgrp [用户组] [文件或目录]**  

---

### <span id="18">umask</span>
显示、设置文件的缺省权限

默认目录缺省权限： rwx r-x r-x  
默认文件缺省权限： rw- r-- r--

**umask [-S]**  

```
umask -S
```

直接执行umask返回四个数字
```
umask
0022
```
- 第一个0是特殊权限
- 后面三位是指缺省权限（022 -> --- -w- -w- 。但是这里是与777亦或的关系，所以缺省的权限为 rwx r-x r-x）

修改缺省权限（一般真的不用）：
```
umask 077
```

---

## 文件搜索命令

### <span id="19">find</span>
搜索文件，服务器高峰建议不要用

**支持正则表达式**：  
???：匹配三个字符  
*:匹配任意多个字符

**find [搜索范围] [匹配条件]**

- `-name` :根据文件名搜索 `find /etc -name hiki`
- `-iname` :不区分大小写文件名搜索
- `-size` :根据文件大小搜索 `find / -size +204800` `+n`: 大于, `-n`:小于,`n`:等于 - `-user` 查询指定所有者的文件
- `-group` :查询指定所属组的文件
- `-amin` :访问时间 ，`find /etc -amin -5`:查询5分钟内被访问过的文件
- `-cmin` :change 文件属性被修改
- `-mmin` :modify 文件内容被修改
- `-type` :根据文件类型查找 `f` 文件， `d` 目录, `l`软链接文件 `find /etc -type f`
- `-a` :and 用于多个条件一起查找
- `-o` :or 用于多个调节人以满足查找
- `-inum` :根据节点查找
- `-exec` :对查找到的文件进行操作 ： `find /etc -name inittab -exec ls -l {}\;` 其中 `{}` 表示查到的结果，`\;` 是转义 `;` 表示结束。
- `-ok` :跟exec一样，但是多了个确认环节

---

### <span id="20">locate</span>
文件资料库中查找文件，比find快很多，它是定期更新资料库，查找时不是遍历硬盘而是再资料库中进行查找

**locate [-i] [文件名]**

**放在`/tmp/`目录中的文件不在资料库的范围中**

资料库在`/var/lib/mlocate/mlocate.db`

手动更新资料库： 
```
updatedb
```

使用:
```
locate inittab
```

- `-i` 不区分大小写查找

---

### <span id="21">which</span>
搜索命令所在目录及别名信息

**which [命令名]**

---

### <span id="22">whereis</span>

搜索命令所在目录及帮助文档路径

**whereis [命令名]**

---

### <span id="23">grep</span>
在文件中搜索字串匹配的行并输出

**grep -iv [指定字串][文件]**

- `-i` 不区分大小写
- `-v` 排除指定字串
- `--colopr=auto` 颜色显示(Centos7自带)

排除注释行( `^` 表示行首 )：

```
grep -v ^# /etc/inittab
```

---

## 帮助命令

### <span id="24">man</span>
manual（帮助手册）
查看 命令/配置文件 的帮助信息

**man [15][命令名/配置名]**

查看的时候可以 按 `/` 然后输入 `-a`之类的选项进行快速定位

查看配置的帮助信息不需要绝对路径

```
man 5 passwd
```

- `1` :表示命令的帮助
- `5` :表示配置文件的帮助
- 
---

### <span id="25">whatis</span>
查看命令的基本介绍

**whatis [命令名]**

---

### <span id="26">apropos</span>
查看配置的基本介绍

**apropos [配置名]**

---

### <span id="27">--help</span>
查看命令的介绍、选项

**[命令名] --help**

---

### <span id="28">info</span>
查看命令的介绍、选项

**info [命令名/配置名]**

---

### <span id="29">help</span>
查看Shell内置命令帮助信息 如 cd、umask

**help [命令名]**

---

## 用户管理命令

### <span id="30">useradd</span>
添加新用户

**useradd 用户名**

---

### <span id="31">passwd</span>
设置用户密码

**passwd [用户名]**

未输入用户名是更改正在使用的用户密码

---

### <span id="32">who</span>
查看登录用户信息

**who**

```
[root@localhost]# who 
root     tty1         2019-09-16 19:47
root     pts/0        2019-09-16 19:48 (10.0.1.14)
```

tty：本地终端登录  
pts：远程终端登录


### <span id="33">w</span>

**w**

查看登录用户信息

```
[root@localhost]# w
 03:58:00 up  8:10,  2 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     tty1                      19:47    8:09m  0.07s  0.07s -bash
root     pts/0    10.0.1.14        19:48    0.00s  0.26s  0.00s w
```

字段 | 含义
-- | --
TTY | 登录终端： `tty`：本地终端登录，`pts`：远程终端登录
IDLE | 用户空闲时间，从用户上一次任务结束后，开会记时。
JCPU | 表示在摸段时间内，所有与该终端相关的进程任务所耗费的CPU时间。
PCPU | 指WHAT域的任务执行后耗费的CPU时间。
WHAT | 表示当前执行的任务。


其中 `up` 后面写的时间为linux运行多久的时间（没有关机），也可以用指令查看：

```
[root@localhost tmp]# uptime
 04:00:56 up  8:13,  2 users,  load average: 0.00, 0.01, 0.05
```

---

## 压缩解压命令
### <span id="34">gzip</span>

压缩文件,只能压缩文件

**gzip [文件]**

压缩后文件格式： `.gz`

---

### <span id="35">gunzip</span>

解压缩文件

**gunzip [文件]**

---

### <span id="36">tar</span>

打包目录

**tar [-zcf][压缩后的文件名][目录]**

- `-z` :打包同时压缩
- `-c` :打包
- `-v` :显示详细信息
- `-f` :指定文件名(f要放在最后面)
- `-x` :解压

打包文件名一般后缀： `.tar`
打包顺便压缩后缀： `.tar.gz`

**`.tar.gz`也是常见的源代码安装包格式**

```
tar -zcvf [文件名] [目录]
```

```
tar -zxvf [文件名]
```

---

### <span id="37">zip</span>

压缩文件或目录，zip是Linux和Windows都支持的格式

**zip [-r][压缩后的文件名][文件或目录]**

- `-r` :压缩目录

---

### <span id="38">unzip</span>

解压缩文件或目录

**unzip [压缩文件]**

---

### <span id="39">bzip2</span>
gz的升级版，有保留原文件的选项，而且压缩比更高，也可以使用 `tar` 指令压缩成bzip2格式

压缩文件或目录

**bzip2 [-k][文件]**

- `-k` : 产生压缩文件后保留原文件

```
bzip2 -k [文件名]
tar -cjf [文件名.tar.bz2] [文件名]
```

---

### <span id="40">bunzip2</span>


解压缩文件

**bunzip2 [-k][压缩文件]**

- `-k` : 产生压缩文件后保留原文件

```
bzip2 -k [压缩文件]
tar -xjf [压缩文件] 
```

---

### <span id="41">write</span>

给用户发信息，以Ctrl+D保存结束

**write <用户名>**

---

### <span id="42">wall</span>
write all

发广播信息

**wall [message]**

---

### <span id="43">ping</span>
测试网络连通性

**ping [-c] [ip]**

- `-c` :指定发送次数

---

### <span id="44">ifconfig</span>
查看和设置网卡信息

```
ifconfig ens33 10.0.1.15
```
如果虚拟机是桥接模式，需要配置的ip地址需与windows连接网络在相同网段

---

### <span id="45">mail</span>
查看发送电子邮件

**mail [用户名]**

查看时输入邮件序号即可

q退出

---

### <span id="46">last</span>
列出目前与过去登入系统的用户信息

---

### <span id="47">lastlog</span>
列出用户最后一次登录信息

**lastlog [-u] [用户名]**

添加-u可以查看指定用户最后一次登录信息

---

### <span id="48">traceroute</span>
跟踪数据包到主机间的路径

**traceroute [网址]**

---

### <span id="49">netstat</span>
显示网络相关信息

**netstat [选项]**

选项 | 作用
-- | --
-t | TCP协议
-u | UDP协议
-l | 监听
-r | 路由
-n | 显示IP地址和端口号

```
netstat -tlun 查看本机监听的端口
netstat -an 查看本机所有的网络连接
netstat -rn 查看本机路由表
```

---

### <span id="50">setup(centos7 使用nmtui命令)</span>
配置网络

---

### <span id="51">mount</span>
挂载命令

把设备连接到挂载点

**mount [-t文件系统] 设备文件名 挂载点**

```
mount -t iso9660 /dev/sr0 /mnt/cdrom

mount /dev/sr0 /mnt/cdrom

```

**umount 设备文件名**

卸载

---

## 关机重启命令

### <span id="52">shutdown</span>
**shutdown [选项] 时间**
- `-c` :取消前一个关机命令
- `-h` :关机
- `-r` :重启

```
shutdown -h now
shutdown -h 20:30
```

其它关机命令:
```
halt
poweroff
init 0
```

系统运行级别：
级别 | 含义
-- | --
0 | 关机
1 | 单用户
2 | 不完全多用户，不含NFS服务（文件共享）
3 | 完全多用户
4 | 未分配
5 | 图形界面
6 | 重启

使用下面指令查看当前运行级别：
```
runlevel
```

---

### <span id="53">reboot</span>
重启命令

其他重启命令：
```
init 6
```

---

### <span id="54">logout</span>
退出登录

---

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.09.17  
> 更新日期：2019.09.19