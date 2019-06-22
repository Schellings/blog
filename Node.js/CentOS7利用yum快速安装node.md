---
title: CentOS7利用yum快速安装node.js
date: 2019-04-15 22:11:12
tags: Node.js
---

## 下载node.js
V8.x: 
```yaml
#curl --silent --location https://rpm.nodesource.com/setup_8.x | bash -
```

V7.x:
```yaml
#curl --silent --location https://rpm.nodesource.com/setup_7.x | bash -
```
V6.x:
```yaml
#curl --silent --location https://rpm.nodesource.com/setup_6.x | bash -
```
V5.x:
```yaml
#curl --silent --location https://rpm.nodesource.com/setup_5.x | bash -
```
## yum安装node.js
```yaml
yum install -y nodejs
```
## 检查
```yaml
node -v
```
