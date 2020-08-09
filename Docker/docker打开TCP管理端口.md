---
title: docker打开TCP管理端口
date: 2019-05-25 16:56:24
tags: 容器
---
<meta name="referrer" content="no-referrer" />

## 开启TCP管理端口

### 创建目录

```shell script
mkdir -p /etc/systemd/system/docker.service.d
```

### 追加内容

```shell script
cat > /etc/systemd/system/docker.service.d/tcp.conf <<EOF
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
EOF
```

### 重启服务
```shell script
systemctl daemon-reload
systemctl restart docker
```
### 查看服务状态

```shell script
netstat -an | grep 2375
```
