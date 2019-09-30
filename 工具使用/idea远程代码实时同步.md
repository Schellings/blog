---
title: idea远程代码实时同步
tags:
  - 工具使用
date: '2018-05-27 15:10'
---
## 前言

开发时一般的平台都是windows，但windows对开发极其不友好，一般都会在本地开启虚拟机，安装上linux环境进行项目的部署测试。下面介绍一种windows主机与linux虚拟机代码同步的方法。

## 前置条件

在windows与linux中都有着同一份代码。

## 配置步骤

1、打开windows主机上的idea，选择tools->deployment->brower remote host

2、点击右上方三个点，新建一个连接
    
```
连接类型：sftp
依次填写虚拟机信息
root path项选中linux虚拟机中的项目位置
打开mapping面板
选择deployment path为根路径/
确定保存
```
3、选择tools->deployment—>auto upload使其处于开启状态

此时，在window主机对项目所做的任何修改一旦被保存都会被实时推送到虚拟机中。一旦在linux虚拟机中项目功能验证通过，开始推送到git远程仓库中。

## 注

该方法可以避免由于windows系统所带来的一些麻烦，同时能利用windows图形化优势。当然，有钱的话直接买mac更合适。

该方法同样适用于远程服务器，并且所有的jetbrains家族的开发工具都支持该操作。

本博客就是通过webstorm设置远程代码同步推送来加快部署。
