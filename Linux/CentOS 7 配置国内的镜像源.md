
---
title: CentOS 7 配置国内的镜像源
date: 2018-02-30 16:56:24
tags: Linux
---
<meta name="referrer" content="no-referrer" />

## 问题描述

```yaml
centos 7 安装完后采用的是国外的镜像仓库，在下载或者更新软件相对慢，所以，在安装完centos 7 系统后，要把镜像源更换成国内的，相对比较好的。
```

## 操作说明

国内比较好的镜像源有：

- 阿里云（http://mirrors.aliyun.com/repo/Centos-7.repo）
- 网易 (http://mirrors.163.com/.help/CentOS6-Base-163.repo)
- 华为（https://repo.huaweicloud.com/repository/conf/CentOS-7-reg.repo ）

我这里选择华为镜像源，作为例子，其他类型的。

使用说明：
1、备份配置文件：

```yaml
cp -a /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
```

2、下载新的CentOS-Base.repo文件到/etc/yum.repos.d/目录下

```yaml
wget -O /etc/yum.repos.d/CentOS-Base.repo https://repo.huaweicloud.com/repository/conf/CentOS-7-reg.repo
```

3、执行yum clean all清除原有yum缓存

```yaml
yum clean all
```

4、执行yum makecache 刷新缓存

```yaml
yum makecache // yum repolist all
```

