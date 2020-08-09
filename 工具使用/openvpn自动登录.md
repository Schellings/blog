---
title: openvpn自动登录
date: 2020-02-03 19:56:24
tags: 
    - 工具使用
---
<meta name="referrer" content="no-referrer" />

## 背景

目前我司实行远程办公，开通openvpn账号之后，每次连接都需要多次输入账号密码信息。

## 方法

- 管理员运行openvpn
- 在openvpn程序上右击，选择[edit config]
- 在文件的最后一行添加：
```xml
auth-user-pass pass.txt
```
- 打开openvpn安装目录
- 找到{openvpn install path}/config，在该目录下创建文件pass.txt
- 编辑pass.txt，分别写入账号密码信息，文件格式：一行账号一行密码即可。
```xml
账号信息
密码信息
```
- 双击openvpn程序即可完成自动连接


