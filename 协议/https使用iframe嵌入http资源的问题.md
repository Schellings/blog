---
title: https使用iframe嵌入http资源的问题
date: 2019-01-07 19:56:24
tags: 
    - 协议
---
<meta name="referrer" content="no-referrer" />

## 目前现象
https 网站使用iframe嵌入http资源网站的内容，会弹出“是否加载不安全的内容”的提示，点击“加载”按钮后显示正常。对用户来说显示不友好。

## 问题原因

https中使用http的资源时，浏览器会认为可能会不安全， 会自动弹出“是否加载不安全的内容”的提示。

该提示由浏览器自动弹出，不能通过修改代码的方式解决。

https中使用https资源时，如果https资源不安全，同样会报错。         

## 尝试解决方案
1） 使用自定义ssl证书，将http资源模拟成为https，使用nginx或者apache服务器，将http协议的资源包装为https协议的资源使用，前提条件是nginx或者apache服务器可以同时访问到http的资源和https的资源，将http链接地址从原网站改为nginx或apache包装的https的地址即可。
   否则会报错“ 因为没有使用有效的安全证书进行签名，该内容已被屏蔽”的新错误提示。

2）使用第三方签名的ssl证书，将http资源转换成为https，和方案1相同，但是使用第三方签名CA证书，网站可以正常访问。但是第三方的CA签名的ssl证书是需要按年付费的， 价格 从每年几百到每年几千元均有。

3) 要求http网站资源提供https的格式内容，且使用的ssl证书为第三方CA签名证书。      
