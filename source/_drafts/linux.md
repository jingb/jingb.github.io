---
title: linux相关
tags: 
- shell
---

# bash

查看当前使用的shell   ps -p $$
系统有的shell      cat /etc/shells

查看shell解释  www.explainshell.com

## 变量相关
```
name="Shell"
url="http://c.biancheng.net/shell/"

str1=$name$url  #中间不能有空格
str2="$name $url"  #如果被双引号包围，那么中间可以有空格
```
<!-- more -->

引用变量
```
cd /lib/modules/`uname -r`/
ls -l `locate crontab`

```

export 解决在父shell把变量值带给子shell
source 是解决不用重新登录变量就能生效问题 
set 显示用户的局部变量和用户环境变量


变量后面接 #, ##, %, %%, /, //   可以做一些操作

### 重要变量
优先级 ```LC_ALL > LC_* >LANG```

### 变量读取顺序和文件夹

#### 顺序
1. 先读取 /etc/profile，再根据 /etc/profile 的内容去读取其它额外的设定档， 例如 /etc/profile.d 与 /etc/inputrc 等等设定档； 
2. 根据不同的使用者，到使用者家目录去读取 ~/.bash_profile 或 ~/.bash_login 或 ~/.profile 等设定档；  
3. 根据不同使用者，到他家目录去读取 ~/.bashr (**在最后面，怕被覆盖的变量可以放这里**)

#### ogin shell 与 non-login shel
**non-login shell ，读取的就是仅有 ~/.bashr**

# 守护进程
分 超级守护进程 和 stand_alone，系统启动时由一个统一的守护进程xinet来负责管理一些进程，当相应请求到来时需要通过xinet的转接才可以唤醒被xinet管理的进程。这种进程的优点时最初只有xinet这一守护进程占有系统资源，其他的内部服务并不一直占有系统资源。stand_alone是速度快，比方http服务需要
xinetd(eXtended InterNET services Daemon) 扩展因特网服务守护进程，通过xinetd服务来管理一些功能简单小服务。如：telnet、tftp、rsync 服务等。并为这些服务提供安全访问控制功能

# 磁盘
一块磁盘  有个MPR  里面有个partition table
由于这个 MBR 区块的容量有限，所以，当初设计的时候，就 只有设计成 4 个分割纪录，这些分割记录就被称为 Primary ( 主分割 ) 及 Extended ( 延伸分割 ) ，也就是说，一颗硬盘最多可以有 4 个 ( Primary + Extended ) 的扇区，其中， Extended 只能有一个， 3P+E    4P

每一个 partition 就是一个 Filesystem

## 分区划分
/etc  /sbin  /bin /dev /lib  要和 根目录在一个partition   因为系统启动的时候只挂一个partition，前面这几个很重要
/home  /usr /var 等可以单独一个partition        比方格式化系统的时候 /home下面的个人数据可以不受影响

# 指令
du -sm /*  检查根目录底下每个目录所占用的容量

# 文件
每个文件用一个 inode(记录属性，属主，操作时间等) 和 一些block(存储内容)

# 正则
[练习文件](http://linux.vbird.org/linux_basic/0330regularex/regular_express.txt)
```
grep -n 't[ae]st' regular_express.txt  // 搜tast或test
grep -n '[^a-z]oo' regular_express.txt // oo前面没有小写字母
grep -n '^[a-z]oo' regular_express.txt // **行首的单词** 以小写字母开头然后接oo 的行  
grep -n '[a-z]oo' regular_express.txt  // 和上面比少了行首  ^在[]内表示没有，[]外表示行首
grep -n '\.$' regular_express.txt  // 捞出行尾是句点的  要加\斜杠是为了防止句点匹配任何字符
grep -nE 'o{2,4}g' regular_express.txt  // o有2-4个然后以g结尾
grep -nE 'o{2,}g' regular_express.txt  // o有2个以上
grep -n 'g*g' regular_express.txt  // g*g  里面的 g* 代表『空字符或一个以上的 g』在加上后面的 g，-> g, gg, ggg, ggg


在万用字符当中，* 代表的是 0~无限 多个字符的意思，但是在正规表示法当中, 
*  则是重复 0 到多个的前一个  RE  字符的意思～使用的意义并不相同
ls a* 是捞a开头的文件  正则用法是 ls | grep  '^a.*'
```

