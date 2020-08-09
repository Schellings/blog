---
title: docker安装OpenLDAP
date: 2019-05-21 16:56:24
tags: 
    - 容器
    - LDAP 
---
<meta name="referrer" content="no-referrer" />
## 说明

```yaml
安装LDAP分Server端和Client端两部分的安装部署
Server端参考地址： https://github.com/osixia/docker-openldap
Client端参考地址： https://github.com/osixia/docker-phpLDAPadmin
server端安装openldap服务，client端安装phpLdapAdmin（linux）或ldapAdmin（windows）
```

## Server端

### 拉取镜像

```yaml
docker pull osixia/openldap:1.2.4
```
### 运行镜像
```yaml
docker run -p 389:389 -p 636:636 --name my-openldap-container --detach osixia/openldap:1.2.4
```
本命令是ldap会默认创建一个admin用户，默认密码也是admin.

```yaml
dn: cn=admin,dc=example,dc=org
pd: admin
```

## Client端

### 拉取镜像
```yaml
docker pull osixia/phpldapadmin:0.8.0
```

### 运行镜像

```yaml
docker run -p 8888:443 \
        --env PHPLDAPADMIN_LDAP_HOSTS=192.168.2.138 \
        --detach osixia/phpldapadmin:0.8.0
```

### 登录
安装成功后输入https://ip:8888 ,然后使用默认的dn+pd登录即可

> windows端可安装ldapAdmin，或者Apache Directory Studio

## 问题

可能会出现连接LDAP服务端失败或者用户名密码错误。

## 解决方案

由于LDAP Server访问是389端口号（具体看映射使用的外部端口），需要将389端口加入到防火墙白名单里面。


