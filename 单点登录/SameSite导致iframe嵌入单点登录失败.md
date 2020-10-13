---
title: SameSite导致iframe嵌入单点登录失败.md
date: 2020-07-16 22:11:12
tags: 
    - 单点登录
---
<meta name="referrer" content="no-referrer" />

## 背景
```yml
在多个接入单点登录业务系统，通过iframe嵌入其他系统页面时，单点登录请求无法携带cookie，导致单点登录失败，打开登录页
```

## cookie携带场景


请求类型|	示例	|正常情况	|Lax
---|---|---|---
链接|	`<a href="…"></a>`|	发送 Cookie	|发送 Cookie
预加载|	`<link rel=“prerender” href="…"/>`|	发送 Cookie|	发送 Cookie
GET 表单|	`<form method=“GET” action="…">`	|发送 Cookie|	发送 Cookie
POST 表单|	`<form method=“POST” action="…">`|	发送 Cookie	|不发送
iframe	|`<iframe src="…"></iframe>`	|发送 Cookie	|不发送
AJAX|	`$.get("…")	`|发送 Cookie	|不发送
Image	|`<img src="…">	`|发送 Cookie	|不发送


## 解决方案

### 浏览器设置

Firefox69 版本以上出现该问题
- 地址栏输入：about:config
搜索
- **network.cookie.sameSite.laxByDefault**  false
- **network.cookie.sameSite.noneRequiresSecure** false

调整对应配置，重启浏览器

chrome80 版本以上出现该问题
- 地址栏输入：chrome://flags/

搜索
- **SameSite by default cookies**  Disable
- **Cookies without SameSite must be secure** Disable

调整对应配置，重启浏览器

### 服务端调整

```yaml
response.addHeader("Set-Cookie", "_u=xxxx; Path=/Login; SameSite=None; Secure");
```
> 注意：SameSite=None和Secure必须同时指定，即该策略仅在https下可正常工作，因此需要将统一认证中心访问升级为https，对相关cookie添加SameSite=None; Secure属性






