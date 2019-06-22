---
title: Git环境安装
date: 2019-03-20 16:56:24
tags: Git
---
# 1、Git初识
Git 是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。

Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。

Git 与常用的版本控制工具 CVS, Subversion 等不同，它采用了分布式版本库的方式，不必服务器端软件支持。

## Git 与 SVN 区别
Git 不仅仅是个版本控制系统，它也是个内容管理系统(CMS)，工作管理系统等。

如果你是一个具有使用 SVN背景的人，你需要做一定的思想转换，来适应 Git 提供的一些概念和特征。

Git 与 SVN 区别点：

1、Git 是分布式的，SVN 不是：这是 Git 和其它非分布式的版本控制系统，例如 SVN，CVS 等，最核心的区别。

2、Git 把内容按元数据方式存储，而 SVN 是按文件：所有的资源控制系统都是把文件的元信息隐藏在一个类似 .svn、.cvs 等的文件夹里。

3、Git 分支和 SVN 的分支不同：分支在 SVN 中一点都不特别，其实它就是版本库中的另外一个目录。

4、Git 没有一个全局的版本号，而 SVN 有：目前为止这是跟 SVN 相比 Git 缺少的最大的一个特征。

5、Git 的内容完整性要优于 SVN：Git 的内容存储使用的是 SHA-1 哈希算法。这能确保代码内容的完整性，确保在遇到磁盘故障和网络问题时降低对版本库的破坏。

![image](https://www.runoob.com/wp-content/uploads/2015/02/0D32F290-80B0-4EA4-9836-CA58E22569B3.jpg)

# 2、Git下载与安装
## 2.1、安装
百度搜索Git官网，下载链接:https://git-scm.com/download ，根据自己电脑系统下载相应的安装包。

Git的安装并不需要多注意什么，选择好安装路径一路next直到finish即可。

Git安装结束后，可开启cmd，输入命令**git --version** 如果返回Git版本信息则安装成功
## 2.2、Git环境配置
Git安装好去GitHub或者其他Git代码托管平台上注册一个账号，注册好后，点击桌面上的Git Bash快捷图标，我们要用账号进行环境配置。

打开Git Bash窗口，输入以下两个命令：


```
# 配置用户名
git config --global user.name "username"    //（ "username"是自己的账户名，）
# 配置邮箱
git config --global user.email "username@email.com"     //("username@email.com"注册账号时用的邮箱)

```
配置结束后，可通过命令

```
git config --global --list
```
来查看Git配置信息
## 2.3、添加Git SSH Key
使用命令：

```
ssh-keygen -t rsa
```
一路回车，会在C:\Users\当前用户\\.ssh文件夹下生成SSH秘钥和私钥。

其中id_rsa是私钥文件，id_rsa_pub是公钥文件。
我们需要把id_rsa_pub的内容添加到GitHub的SSH Key中。进入你的GitHub首页，登录后点击头像——>settings——>ssh and gpg keys——>在SSH Keys处add一个key,内容填写id_rsa_pub文件中的内容。

至此添加完SSH Keys之后，可通过Git命令来操作远程Git仓库，下一节，我们来学习==Git中的常用命令==。