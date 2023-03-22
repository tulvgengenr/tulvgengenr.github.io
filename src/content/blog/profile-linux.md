---
title: "/usr/bin /etc/profile等文件的区别"
description: "如何优雅地在Linux中安装软件，并管理系统变量和用户变量"
pubDate: "March 22 2023"
heroImage: "/linux-bash.jpg"
---
在项目中遇到了一个问题：在docker中的Alpine-linux的容器中，安装了自主编写的软件，通过`export`将软件路径加入到`/etc/profile`中，从而预期实现命令的全局性，但是却没有识别到。问题在于，Alpine/Docker并不会自动加载/etc/profile：https://stackoverflow.com/a/43743532

因此通过本文章，想梳理下如何优雅地在Linux中安装软件，并管理系统变量和用户变量

## 简介linux文件系统

图by@[罗罗的游戏](https://www.zhihu.com/people/luozhiwen)
![file-system](/linux-file.jpg)

主目录：/root、/home/username

挂载点：/mnt

分享目录：/usr/share、/usr/local/share

内核：/boot

服务器数据：/var、/srv

系统信息：/proc、/sys

## 如何安装软件

### 管理员进行安装

+ 所有用户均可执行的可执行文件放在`/bin`(`/usr/bin`)

+ 管理员才能执行的可执行文件放在`/sbin`(`/usr/sbin`)

+ 共享库放在`/lib`(`/usr/lib`)、`lib64`(`/usr/lib64`)

+ 源文件放在`/usr/src`

+ 配置文件放在`/etc`

+ 头文件放在`/usr/include`
  
### 用户进行安装

+ 所有用户均可执行的可执行文件放在`/usr/local/bin`

+ 管理员才能执行的可执行文件放在`/usr/local/sbin`

+ 共享库放在`/usr/local/lib`、`/usr/local/lib64`

+ 源文件放在`/usr/local/src`

+ 配置文件放在`/usr/local/etc`

+ 头文件放在`/usr/local/include`

即，只需要把父目录改为**usr/local**

## 系统变量和用户变量

+ /etc/profile: 用来设置系统环境参数，比如$PATH. 这里的环境变量是对系统内的所有用户生效的。

+ /etc/bashrc: 用来设置bash shell的环境变量，对系统内的所有用户生效，只要用户使用bash shell登录系统，该文件就会被读取。

+ ~/.profile: 功能和/etc/profile 类似，但是这个是针对用户来设定的，比如在/home/user1/.profile 中设定了环境变量，那么这个环境变量只针对 user1 这个用户生效.

+ ~/.bashrc: 作用类似于/etc/bashrc, 只是针对用户自己而言，不对其他用户生效。

### Shell

#### 交互式Shell

交互式Shell分为登录和非登录。

使用ssh登录到系统的时候，得到的是一个交互式的、登录的shell。

但是当在一个已经登录的shell上调用新的shell时，得到的是一个交互式的、非登录的shell，此shell只执行~/.bashrc文件。

#### 非交互式Shell

非交互式shell不需要人类干预即可执行命令，比如当脚本生成子shell来执行命令时，子shell是非交互式shell，**非交互式shell不会执行任何启动文件**（如果我们想要通过python脚本调用某个自己安装的软件命令，就不要放在配置文件中，就应该放在/bin下），它从创建它的shell中继承了环境变量。 

#### 启动配置文件执行顺序

交互式的登录的bash shell：

```shell
/etc/profile --> /etc/profile.d/*.sh --> (~/.bash_profile | ~/.bash_login | ~/.profile) --> ~/.bashrc --> /etc/bashrc
```
注：`~/.bash_profile`、`~/.bash_login`、`~/.profile`以先找到哪个为准
<br>
交互式的非登录的bash shell：
        
```shell   
~/.bashrc --> /etc/bashrc --> /etc/profile.d/*.sh
```

因此，无论是登录还是非登录，~/.bashrc、/etc/bashrc中的配置都会生效。

#### 如何添加shell配置

注意到除了bash shell外还有其他shell，比如从docker中直接进入alpine-linux容器，启动的就是ash（一种轻量级的shell）.

**因此我们首选配置在`/etc/profile`和`~/.profile`。**

（不过bash是Linux系统默认使用的shell，所以大部分人都会默认以bash shell登录，所以都习惯配置在`~/.bashrc`中）


