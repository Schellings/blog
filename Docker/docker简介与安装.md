---
title: Docker简介与安装
date: 2019-03-28 16:56:24
tags: 容器
---

环境：CentOS 7.6

# docker-ce 安装
1、卸载老版本，较老版本的Docker被称为docker或docker-engine。如果这些已安装，请卸载它们以及关联的依赖关系。

```
sudo yum remove docker \
                  docker-common \
                  docker-selinux \
                  docker-engine
```


2、安装所需的软件包 yum-utils提供了yum-config-manager 效用，并device-mapper-persistent-data和lvm2由需要devicemapper存储驱动程序。


```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```


3、添加镜像源

```
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```


4、将软件包添加至本地缓存

```
sudo yum makecache fast
```


5、安装docker-ce

```
sudo yum install docker-ce
```


6、启动docker

```
sudo systemctl start docker
```

7、检查是否安装成功


```
sudo docker info
```
如此时docker版本信息会在终端打印出来，则docker安装成功。