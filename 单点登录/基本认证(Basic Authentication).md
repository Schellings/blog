---
title: 基本认证(Basic Authentication)
date: 2018-12-03 19:56:24
tags: 
    - 单点登录
---

## 基本认证(Basic Authentication)

基本认证(Basic Authentication)是指在一次HTTP请求中传递参数用于验证本次请求的客户端身份的验证方式。

包含基本认证的HTTP请求需要添加如下请求头：

```yaml
Authorization: Basic ${Base64.encode(clientId+":"+clientSecret)}
```

## 发送方法

### postman

在postman发送请求时，如果需要进行基本认证，则在请求时添加Authorization头，类型选择为Basic Auth，依次输入username,password(对应客户端的账号，密码)

### curl

```yaml
curl -kX POST --user iamapp:iamapp http://114.67.23.106:8888/sso/login
```
其中 --user参数即客户端，客户端密钥的值
