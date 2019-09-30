---
title: docker批量删除容器与镜像
date: 2018-08-19 16:56:24
tags: 
     - 容器
---
1、停止所有容器
```yaml
docker stop `docker ps -a -q`
```

2、删除所有容器
```yaml
docker rm `docker ps -a -q`
```

3、删除所有镜像
```yaml
docker rmi `docker images -q`
```

4、按条件删除镜像

```yaml
　　没有打标签：
docker rmi `docker images -q | awk '/^<none>/ { print $3 }'`

　　镜像名包含关键字：
docker rmi --force `docker images | grep doss-api | awk '{print $3}'`  //其中doss-api为关键字
```
