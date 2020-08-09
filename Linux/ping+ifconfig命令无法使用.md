
---
title: ping/ifconfig命令无法使用
date: 2018-02-27 16:56:24
tags: Linux
---
<meta name="referrer" content="no-referrer" />

Centos7镜像安装完了发现基础的ping/ifconfig命令都是无法使用
- 修改网络配置

将ONBOOT=no改为yes

```
vi /etc/sysconfig/network-scripts/ifcfg-ens33 

TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=23b1ac4c-3d20-4d25-9c44-438c9c03845a
DEVICE=ens33
ONBOOT=yes					//此处默认为NO改为yes

```
- 修改完毕后重启网卡

```
service network restart 
```

- ping一下网络

```
[root@localhost ~]# ping www.baidu.com
PING www.a.shifen.com (183.232.231.172) 56(84) bytes of data.
64 bytes from 183.232.231.172 (183.232.231.172): icmp_seq=1 ttl=54 time=31.0 ms
64 bytes from 183.232.231.172 (183.232.231.172): icmp_seq=2 ttl=54 time=31.0 ms
64 bytes from 183.232.231.172 (183.232.231.172): icmp_seq=3 ttl=54 time=31.3 ms
```
- 安装基础命令依赖包

```
yum install net-tools.x86_64

```
- 安装完测试命令

```
[root@localhost ~]# ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.12.150  netmask 255.255.255.0  broadcast 10.0.12.255
        inet6 fe80::b62e:5bb1:b13d:7ac3  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:05:e8:85  txqueuelen 1000  (Ethernet)
        RX packets 8919  bytes 11085873 (10.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3773  bytes 273515 (267.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 332  bytes 28808 (28.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 332  bytes 28808 (28.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```
