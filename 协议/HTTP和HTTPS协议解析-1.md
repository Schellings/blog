---
title: HTTP和HTTPS协议解析（一）
date: 2018-01-07 19:56:24
tags: 
    - 协议
top: 10
---
<meta name="referrer" content="no-referrer" />
# 大纲
![image](http://on-img.com/chart_image/5b5039cbe4b067df59e110dd.png)

# 一、前言：
http:

![image](https://img-blog.csdn.net/20180719095113808?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9taW5nMTAwMDAx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

https:

![image](https://img-blog.csdn.net/20180719133425838?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9taW5nMTAwMDAx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

先来观察这两张图，第一张访问域名http://www.12306.cn，谷歌浏览器提示不安全链接，第二张是https://kyfw.12306.cn/otn/regist/init，浏览器显示安全，为什么会这样子呢？

2017年1月发布的Chrome浏览器开始把收集密码或信用卡数据的HTTP页面标记为“不安全”，若用户使用2017年10月推出的Chrome ，带有输入数据的HTTP页面和所有以无痕模式浏览的HTTP页面都会被标记为“不安全”。

此外，苹果公司强制所有iOS App在2017年1月1日前使用HTTPS加密。

# 二、HTTP和HTTPS发展历史
## 什么是HTTP?


```
超文本传输协议，是一个基于请求与响应，无状态的，应用层的协议，常基于TCP/IP协议传输数据，互联网上应用最为广泛的一种网络协议,所有的WWW文件都必须遵守这个标准。

设计HTTP的初衷是为了提供一种发布和接收HTML页面的方法。
```

## 发展历史


版本  |	产生时间	| 内容	|发展现状
--- | ---|  ---| ---
HTTP/0.9 |	1991年 |	不涉及数据包传输，规定客户端和服务器之间通信格式，只能GET请求|	没有作为正式的标准
HTTP/1.0  |	1996年|	传输内容格式不限制，增加PUT、PATCH、HEAD、 OPTIONS、DELETE命令	| 正式作为标准
HTTP/1.1 |	1997年 |	持久连接(长连接)、节约带宽、HOST域、管道机制、分块传输编码 |	2015年前使用最广泛
HTTP/2 |	2015年 |	多路复用、服务器推送、头信息压缩、二进制协议等 |逐渐覆盖市场

![image](https://img-blog.csdn.net/20180723103857872?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9taW5nMTAwMDAx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

这个Akamai公司建立的一个官方的演示，使用HTTP/1.1和HTTP/2同时请求379张图片，观察请求的时间，**明显看出HTTP/2性能占优势**。

![image](https://img-blog.csdn.net/20180723105652242?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9taW5nMTAwMDAx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**多路复用：**
通过单一的HTTP/2连接请求发起多重的请求-响应消息，多个请求stream共享一个TCP连接，实现多留并行而不是依赖建立多个TCP连接。

## 什么是HTTPS？

```
《图解HTTP》这本书中曾提过HTTPS是身披SSL外壳的HTTP。HTTPS是一种通过计算机网络进行安全通信的传输协议，经由HTTP进行通信，利用SSL/TLS建立全信道，加密数据包。

HTTPS使用的主要目的是提供对网站服务器的身份认证，同时保护交换数据的隐私与完整性。

PS:TLS是传输层加密协议，前身是SSL协议，由网景公司1995年发布，有时候两者不区分。

```

# 三、HTTP VS HTTPS

## HTTP特点：
- 1、无状态：协议对客户端没有状态存储，对事物处理没有“记忆”能力，比如访问一个网站需要反复进行登录操作

- 2、无连接：HTTP/1.1之前，由于无状态特点，每次请求需要通过TCP三次握手四次挥手，和服务器重新建立连接。比如某个客户机在短时间多次请求同一个资源，服务器并不能区别是否已经响应过用户的请求，所以每次需要重新响应请求，需要耗费不必要的时间和流量。

- 3、基于请求和响应：基本的特性，由客户端发起请求，服务端响应

- 4、简单快速、灵活

- 5、通信使用明文、请求和响应不会对通信方进行确认、无法保护数据的完整性

## 解析案例

下面通过一个简单的抓包实验观察使用HTTP请求传输的数据： 

![image](https://img-blog.csdn.net/20180723103319469?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9taW5nMTAwMDAx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![image](https://img-blog.csdn.net/20180719135617449?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9taW5nMTAwMDAx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

结果分析：**HTTP协议传输数据以明文形式显示**

**针对无状态的一些解决策略**：

场景：逛电商商场用户需要使用的时间比较长，需要对用户一段时间的HTTP通信状态进行保存，比如执行一次登陆操作，在30分钟内所有的请求都不需要再次登陆。

1、通过Cookie/Session技术

2、HTTP/1.1持久连接（HTTP keep-alive）方法，只要任意一端没有明确提出断开连接，则保持TCP连接状态，在请求首部字段中的Connection: keep-alive即为表明使用了持久连接

## HTTPS特点

基于HTTP协议，通过SSL或TLS提供加密处理数据、验证对方身份以及数据完整性保护。

![image](https://img-blog.csdn.net/20180719135629906?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9taW5nMTAwMDAx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

通过抓包可以看到**HTTPS数据不是明文传输**。

而且HTTPS有如下特点：

- 1、内容加密：采用混合加密技术，中间者无法直接查看明文内容
- 2、验证身份：通过证书认证客户端访问的是自己的服务器
- 3、保护数据完整性：防止传输的内容被中间人冒充或者篡改


```
1、混合加密：结合非对称加密和对称加密技术。客户端使用对称加密生成密钥对传输数据进行加密，然后使用非对称加密的公钥再对秘钥进行加密，所以网络上传输的数据是被秘钥加密的密文和用公钥加密后的秘密秘钥，因此即使被黑客截取，由于没有私钥，无法获取到加密明文的秘钥，便无法获取到明文数据。 

2、数字摘要：通过单向hash函数对原文进行哈希，将需加密的明文“摘要”成一串固定长度(如128bit)的密文，不同的明文摘要成的密文其结果总是不相同，同样的明文其摘要必定一致，并且即使知道了摘要也不能反推出明文。 

3、数字签名技术：数字签名建立在公钥加密体制基础上，是公钥加密技术的另一类应用。它把公钥加密技术和数字摘要结合起来，形成了实用的数字签名技术。
 （1）收方能够证实发送方的真实身份；
 （2）发送方事后不能否认所发送过的报文；
 （3）收方或非法者不能伪造、篡改报文。
```
加密过程如下所示：

![image](https://img-blog.csdn.net/20180719103559793?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9taW5nMTAwMDAx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

非对称加密过程需要用到公钥进行加密，那么公钥从何而来？

其实公钥就被包含在数字证书中，数字证书通常来说是由受信任的数字证书颁发机构CA，在验证服务器身份后颁发，证书中包含了一个密钥对（公钥和私钥）和所有者识别信息。

数字证书被放到服务端，具有服务器身份验证和数据传输加密功能。
