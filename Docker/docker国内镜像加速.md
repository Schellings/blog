---
title: Docker国内镜像加速
date: 2018-03-31 16:56:24
tags: 容器
---
# 镜像加速器
请在 **/etc/docker/daemon.json** 中写入如下内容（如果文件不存在请新建该文件）

```
{
 "registry-mirrors": [
    "http://hub-mirror.c.163.com"
  ]
}
```

注意，一定要保证该文件符合 json 规范，否则 Docker 将不能启动。
之后重新启动服务。

重启docker

```
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

