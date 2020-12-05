
---
title: service network start失败 - Failed to start LSB
date: 2018-02-28 16:56:24
tags: Linux
---
<meta name="referrer" content="no-referrer" />

## 问题描述

`systemctl restart network`命令报错：

```yaml
Job for network.service failed. See 'systemctl status network.service' and 'journalctl -xn' for details.[FAILED][root@localhost network-scripts]# systemctl status network.servicenetwork.service - LSB: Bring up/down networking  
...
```

## 问题解决

解决方式：禁用NetworkManager
```yaml
systemctl stop NetworkManager
systemctl disable NetworkManager
systemctl restart network
```