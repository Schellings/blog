---
title: Chrome 80默认SameSite导致跨域登录状态
date: 2020-03-12 22:11:12
tags: 
    - 单点登录
---
<meta name="referrer" content="no-referrer" />

## 背景
```yml
在Chrome 80版本中，Chrome会将没有声明SameSite值的cookie默认设置为SameSite=Lax。
只有采用SameSite=None; Secure设置的cookie可以从外部访问，前提是通过安全连接（即HTTPS）访问。
```

## 什么是SameSite
SameSite是Cookie中的一个属性，它用来标明这个 cookie 是个“同站 cookie”，“同站 cookie” 只能作为第一方cookie，不能作为第三方cookie，因此可以限制第三方Cookie，解决CSRF的问题。不知道CSRF的看着这个。早在Chrome 51中就引入了这一属性，但是不会默认设置，所以相安无事。

第三方Cookie：由当前a.com页面发起的请求的 URL 不一定也是 a.com 上的，可能有 b.com 的，也可能有 c.com 的。我们把发送给 a.com 上的请求叫做第一方请求（first-party request），发送给 b.com 和 c.com 等的请求叫做第三方请求（third-party request），第三方请求和第一方请求一样，都会带上各自域名下的 cookie，所以就有了第一方cookie（first-party cookie）和第三方cookie（third-party cookie）的区别。上面提到的 CSRF 攻击，就是利用了第三方 cookie可以携带发送的特点 。

“同站cookie”不是根据同源策略判断，而是PSL（公共后缀列表），子域名可以访问父域名cookie，但父域名无法访问子域名cookie。

SameSite总共有三个值：Strict、Lax、None。

### Strict
Strict最为严格，完全禁止第三方 Cookie，跨站点时，任何情况下都不会发送 Cookie。换言之，只有当前网页的 URL 与请求目标一致，才会带上 Cookie。
```yaml
Set-Cookie: CookieName=CookieValue; SameSite=Strict;
```
这个规则过于严格，可能造成非常不好的用户体验。比如像本人当前遇到的现象，cookie带不过，等于一直没有登录状态，就会回到登录页。

### Lax

Lax规则稍稍放宽，大多数情况也是不发送第三方 Cookie，但是导航到目标网址的 Get 请求除外。Chrome 80之后默认设置为该值。

```yaml
Set-Cookie: CookieName=CookieValue; SameSite=Lax;
```

导航到目标网址的 GET 请求，只包括三种情况：链接，预加载请求，GET 表单。详见下表。

请求类型|	示例	|正常情况	|Lax
---|---|---|---
链接|	`<a href="…"></a>`|	发送 Cookie	|发送 Cookie
预加载|	`<link rel=“prerender” href="…"/>`|	发送 Cookie|	发送 Cookie
GET 表单|	`<form method=“GET” action="…">`	|发送 Cookie|	发送 Cookie
POST 表单|	`<form method=“POST” action="…">`|	发送 Cookie	|不发送
iframe	|`<iframe src="…"></iframe>`	|发送 Cookie	|不发送
AJAX|	`$.get("…")	`|发送 Cookie	|不发送
Image	|`<img src="…">	`|发送 Cookie	|不发送

设置了Strict或Lax以后，基本就杜绝了CSRF攻击。当然，前提是用户浏览器支持 SameSite 属性。

### None
浏览器会在同站请求、跨站请求下继续发送cookies，不区分大小写。网站可以选择显式关闭 SameSite 属性，将其设为 None ，同时必须设置 Secure 属性（表示Cookie 只能通过 HTTPS 协议发送，HTTP协议不会发送），否则无效。

下面为无效响应头：

```yaml
Set-Cookie: widget_session=abc123; SameSite=None
```

下面为有效响应头：
```yaml
Set-Cookie: widget_session=abc123; SameSite=None; Secure
```

### 解决办法

本人项目中，采用单点登录，在验证登录状态时存在跨域，即采用JSONP的方式获取JWT等相关信息，然后写入本项目域名下的cookie中，满足Lax属性值表单中的AJAX请求，所以不会发送Cookie。
（1）显示关闭SameSite属性，按照上述有效响应头设置登录接口的响应头即可

```java
response.addHeader("Set-Cookie", "_u=xxxx; Path=/Login; SameSite=None; Secure");
```
（2） 浏览器显式关闭该功能。（不推荐，这个功能还是蛮有用的）

地址栏输入：`chrome://flags/`
找到`SameSite by default cookies`和`Cookies without SameSite must be secure`
将上面两项设置为 `Disable`





