---
title: make 失败 【cc command not found】
date: 2018-02-30 16:56:24
tags: Linux
---
<meta name="referrer" content="no-referrer" />


## 问题描述

在构建一些以第三方源码安装的工具包时，经常需要执行以下命令

```yaml
make && make install
```

在默认安装的CentOS 7中通常会构建失败，抛出错误

```yaml
/bin/sh cc command not found
```

原因是因为源码包使用c/c++来编写构建，此时依赖于gcc环境

## 问题解决

安装 gcc ，安装命令如下

```yaml
sudo yum -y install gcc gcc-c++ libstdc++-devel
```

安装完后，重新执行

```yaml
make && make install
```