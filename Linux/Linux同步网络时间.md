---
title: Linux同步网络时间
date: 2018-11-02 16:56:24
tags: Linux
---
<meta name="referrer" content="no-referrer" />

## 前言

在一些对时间比较敏感的业务系统中，服务器时间误差出现在3-5分钟就有可能导致系统出现一些无法预知的故障，下面来介绍一下不同网络时间


## 同步

```yaml
查看Linux服务器时间
date

同步网络时间

ntpdate -u ntp.api.bz

如果ntpdate命令不存在
yum -y install ntp

```