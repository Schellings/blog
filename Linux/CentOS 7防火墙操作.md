---
title: CentOS 7防火墙操作
date: 2018-11-05 16:56:24
tags: Linux
---
<meta name="referrer" content="no-referrer" />

## 查看防火墙状态
 - 查看防火墙状态 systemctl status firewalld
 - 开启防火墙 systemctl start firewalld  
 - 关闭防火墙 systemctl stop firewalld
 - 开启防火墙 service firewalld start 
 
 若遇到无法开启
 - 先用：systemctl unmask firewalld.service 
 - 然后：systemctl start firewalld.service

## 查看对外开放的端口状态
- 查询已开放的端口 netstat  -ntulp | grep 端口号：可以具体查看某一个端口号  
- 查询指定端口是否已开 firewall-cmd --query-port=666/tcp
  

## 对外开放端口

- 查看想开的端口是否已开：firewall-cmd --query-port=6379/tcp
- 添加指定需要开放的端口：firewall-cmd --add-port=123/tcp --permanent
- 重载入添加的端口：firewall-cmd --reload
- 查询指定端口是否开启成功：firewall-cmd --query-port=123/tcp
- 移除指定端口：firewall-cmd --permanent --remove-port=123/tcp