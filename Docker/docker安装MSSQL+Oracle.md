---
title: docker安装MSSQL+Oracle
date: 2019-08-14 16:56:24
tags: 
     - 容器
     - 数据库
---

## docker安装mssql

### 安装
```yaml
docker search mssql
docker pull microsoft/mssql-server-linux

-- 等待下载

docker run --name mssql -m 2048m -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=pass@word1' -p 8663:1433 -d microsoft/mssql-server-linux

-- 执行镜像的时候需要指定内存，建议给2g

```
### 连接

MSSQL:
   ip:docker宿主机的ip地址
   port:8663
   username:sa
   password:pass@word1

## docker安装oracle

### 安装
```yaml
-- 等待下载并启动
docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
docker run --name oracle -p 8661:1521 -d registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g

-- 登录
docker exec -it oracle /bin/bash
source /home/oracle/.bash_profile
sqlplus /nolog
conn as sysdba
username:root
password:helowin

-- 启用scott用户

alter user scott account unlock;
alter user scott identified by tiger;

```

### 连接

Oracle:
  ip:docker宿主机的ip地址
  port:8661
  sid: HELOWIN
  username:scott
  password:tiger