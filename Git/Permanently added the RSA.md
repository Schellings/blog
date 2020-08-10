---
title: Permanently added the RSA host key for IP address
date: 2020-04-12 16:56:24
tags: Git
---
<meta name="referrer" content="no-referrer" />

在使用git push origin master推送内容到github上时报如下错误：

Warning: Permanently added the RSA host key for IP address 'xx.xxx.xxx.xxx' to the list of known hosts.

解决办法：

linux：

在vim /etc/hosts文件下添加如下内容：

xx.xxx.xxx.xxx github.com   根据自己的Ip修改

windows：

在C:\Windows\System32\drivers\etc\hosts文件添加

xx.xxx.xxx.xxx github.com   根据自己的ip修改