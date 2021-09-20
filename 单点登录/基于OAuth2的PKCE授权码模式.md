---
title: 基于OAuth2的PKCE授权码模式
date: 2020-12-03 19:56:24
tags:
- 单点登录
---
<meta name="referrer" content="no-referrer" />

## 背景

假设某一个原生客户端（即没有服务端的纯桌面应用程序、安卓、IOS、浏览器插件程序等）有需要接入第三方OAuth2.0的需求（code 授权码模式）。

首先按照OAuth2.0授权码模式的标准，需要按如下顺序工作：

- 这个客户端首先需要请求认证服务器的获取code的URL
- 认证服务器弹出登录页面
- 用户登录或确认授权
- 原生客户端截获认证服务器 redirect_uri 地址并获取code（一次性授权码）
- 原生客户端使用 code 调用认证服务器的/token?grant_type=authorize_code接口换取 token



其中第五步为请求示例为：

```shell
curl --location --request POST 'https://xxx/sso/oauth/2/token?code=YmluZ286YmluZ29fbWVtYmVyOnlJR3pwalRXNA&grant_type=authorization_code' \
--header 'Authorization: Basic MTExOjIyMg=='
```

其中code为一次性授权码，header中的Authorization请求头为客户端基本身份认证，规范为Basic Base64("clientid"+":"+"clientsecret")

客户端的基本信息只经过基础的base64编码即在原生客户端发起换取 token 的网络请求中携带的时候，这两个值很容易被网络不法分子截获而泄漏（常规来说有服务端的应用这个请求是由服务器和服务提供商进行交互的，服务器发起的网络请求不会在互联网上被截获），所以对于纯客户端来说，Oauth2的这样使用已经不能满足实际需求了。


所以衍生了 [PKCE 授权码模式](https://tools.ietf.org/html/rfc7636)

## PKCE (Proof Key for Code Exchange)

PKCE，全称 Proof Key for Code Exchange。这其实是通过一种密码学手段确保恶意第三方即使截获Authorization Code 或者其他密钥，也无法向认证服务器交换Access Token。(我们一般称为挑战模式)

PKCE的流程大概如下：

- 随机生成一串字符并作URL-Safe的Base64编码处理，结果用作 `code_verifier`（这个值记录下来后续会用到）
- 将这串字符通过SHA256哈希，并用URL-Safe的Base64编码处理，结果用作 `code_challenge`
- 把 code_challenge 带上，跳转认证服务器，获取 Authorization Code
- 把 code_verifier带上，换取Access Token

由于网络不法分子这样的中间人虽然可以截获 code_challenge，但是他并不能由 code_challenge 逆推 code_verifier，只有客户端自己才知道这两个值。因此即使中间人截获了 code_challenge，Authorization Code 等，也无法换取Access Token，避免了安全问题。

别忘了我们一开始提到的关于 Client ID 和 Client Secret 的问题，所以使用 PKCE 方式我们可以在不提供这两个值的情况下依然达到安全获取 token 的目的。

具体参考 https://fusionauth.io/docs/v1/tech/oauth/endpoints/#authorize



