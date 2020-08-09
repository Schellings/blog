---
title: 使用ssh连接虚拟机
date: 2018-02-27 16:56:24
tags: Linux
---
<meta name="referrer" content="no-referrer" />
1.查看防火墙状态

```
sudo systemctl status firewalld
```

根据防火墙的状态，关闭时执行开启防火墙命令，否则执行查看端口命令

2.关闭防火墙

```
sudo systemctl stop firewalld
```

3.开启防火墙

```
sudo systemctl start firewalld
```

4.查看开放端口

```
[root@localhost ~]# firewall-cmd --list-ports

```

5.如果没有开放22端口需开放22端口

```
firewall-cmd --zone=public --add-port=22/tcp --permanent
```

6.重启防火墙
```
firewall-cmd --reload
```

7.修改虚拟机为桥接模式

```
注：修改桥接模式将虚拟机IP通过本机桥接外网则虚拟机和本机IP网段保持一致，可以通过ssh连接
```

8.在命令行工具中使用ssh命令连接虚拟机（推荐使用cmder）

```
ssh 登录用户@虚拟机ip地址
（输入密码）
```
