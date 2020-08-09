---
title: LDAP与AD
date: 2018-06-28 19:56:24
tags: 
    - LDAP
---
<meta name="referrer" content="no-referrer" />

## AD和LDAP的关系

### LDAP
LDAP是轻量目录访问协议(Lightweight Directory Access Protocol)的缩写。

LDAP标准实际上是在X.500标准基础上产生的一个简化版本

### AD
AD是Active  Directory的缩写，AD应该是LDAP的一个应用实例，而不应该是LDAP本身。比如：windows域控的用户、权限管理应该是微软公司使用LDAP存储了一些数据来解决域控这个具体问题，

只是AD顺便还提供了用户接口，也可以利用Active Directory当做LDAP服务器存放一些自己的东西而已。比如LDAP是关系型数据库，微软自己在库中建立了几个表，每个表都定义好了字段。显然这些表和字段都是根据微软自己的需求定制的，而不是LDAP协议的规定。然后微软将LDAP做了一些封装接口，用户可以利用这些接口写程序操作LDAP，使得Active Directory也成了一个LDAP服务器。

### 总结
总之：Active Directory = LDAP服务器＋LDAP应用（Windows域控）。Active Directory先实现一个LDAP服务器，然后自己先用这个LDAP服务器实现了自己的一个具体应用（域控）