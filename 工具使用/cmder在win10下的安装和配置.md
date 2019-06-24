---
title: CentOS 7.6 Hexo+Yelee博客搭建
date: 2018-12-21 19:56:24
tags: 
    - 工具使用
---
## 工具介绍

相信很多windows上做开发的朋友都对cmd与powershell两个工具是又爱又恨，今天介绍一个新的命令行工具——**cmder**.

cmder支持你在windows上使用一些linux的常用命令，例如cp ,pwd,ls等。

## 下载和解压
（1）到[cmder官网](http://cmder.net/)下载 cmder_mini.zip（当然也可以选择到[cmder的github](https://github.com/cmderdev/cmder)上下载）

（2）直接解压 cmder_mini.zip，如解压到：D:\\cmder_mini

（3）将 D:\\cmder_mini 添加到系统环境变量中，至此已可用cmder

（4）crtl+R，输入cmder，确定，即可运行cmder

##  注册右键菜单
作用：在鼠标右击时，菜单栏会出现“cmder here”可快速进入项目目录

配置操作：对开始菜单按钮右击，选择打开windows powershell的管理员模式，执行以下命令即可：

命令：
```yaml
Cmder.exe /REGISTER ALL
```
## 更改lambda符号

在cmder中执行以下命令：
```yaml
cmderr
cd vendor/
vi clink.lua
```
将第41行的代码修改如下：
```yaml
原来代码：

color codes: "\x1b[1;37;40m"
local cmder_prompt = "\x1b[1;32;40m{cwd} {git}{hg}{svn} \n\x1b[1;30;40m{lamb} \x1b[0m"

-- 修改后：
-- color codes: "\x1b[1;37;40m"
local cmder_prompt = "\x1b[1;32;40m{cwd} {git}{hg}{svn} \n\x1b[1;37;40m$ \x1b[0m"
```

将{lamb}替换为$

### 快捷指令设置
cmder默认不支持ll命令，修改如下：
```yaml
cmderr
cd config/
vi user-aliases.cmd

:set nu
insert(键)

  1 ;= @echo off
  2 ;= rem Call DOSKEY and use this file as the macrofile
  3 ;= %SystemRoot%\system32\doskey /listsize=1000 /macrofile=%0%
  4 ;= rem In batch mode, jump to the end of the file
  5 ;= goto:eof
  6 ;= Add aliases below here
  7 e.=explorer .
  8 gl=git log --oneline --all --graph --decorate  $*
  9 ls=ls --show-control-chars -F --color $*
 10 pwd=cd
 11 clear=cls
 12 history=cat "%CMDER_ROOT%\config\.history"
 13 unalias=alias /d $1
 14 vi=vim $*
 15 cmderr=cd /d "%CMDER_ROOT%"
 16 
 17 ll=ls -l --color=auto $*

:wq
```

第17行就是添加的 ll 指令



