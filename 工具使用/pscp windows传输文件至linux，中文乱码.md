---
title: pscp windows传输文件至linux，中文乱码
date: 2020-03-01 19:56:24
tags: 
    - 工具使用
---
<meta name="referrer" content="no-referrer" />
## 问题描述

pscp命令跨系统传输文件至linux后，文件名出现中文乱码。

## 解决方法
1.安装convmv
```yaml
sudo apt-get install convmv
或：yum -y install convmv
```

2.在要修改文件的路径下打开终端
```yaml
convmv -rf GBK -t utf-8 --notest *
```
