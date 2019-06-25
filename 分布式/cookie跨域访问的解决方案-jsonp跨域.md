---
title: cookie跨域访问的解决方案-Nginx反向代理
date: 2019-05-21 19:56:24
tags: 
    - Cookie跨域
    - jsonp
---
## 同源策略
要了解跨域，必须先了解浏览器同源策略，接下来搬运了一些大神的总结:

### 什么是同源策略？

同源策略/SOP（Same origin policy）是一种约定，由Netscape公司1995年引入浏览器，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，浏览器很容易受到XSS、CSFR等攻击。

所谓同源是指"协议+域名+端口"三者相同，即便两个不同的域名指向同一个ip地址，也非同源。

### 同源策略限制行为

```yaml
1.Cookie、LocalStorage 和 IndexDB 无法读取
2.DOM 和 Js对象无法获得
3.AJAX 请求不能发送
```
假设没有同源策略，那么我在A网站下的cookie就可以被任何一个网站拿到。

那么这个网站的所有者，就可以使用我的cookie(也就是我的身份)在A网站下进行操作。

同源策略可以算是 web 前端安全的基石，如果缺少同源策略，浏览器也就没有了安全性可言。

同源策略做了很严格的限制，但是在实际的场景中，又确实有很多地方需要突破同源策略的限制，也就是我们常说的跨域。

跨域的方法有很多，如接下来要接触的jsonp跨域。
（关于Nginx反向代理解决跨域问题参考—>[传送门](https://blog.xielin.top/2019/05/17/%E5%88%86%E5%B8%83%E5%BC%8F/cookie%E8%B7%A8%E5%9F%9F%E8%AE%BF%E9%97%AE%E7%9A%84%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88-Nginx%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86/)。

## 使用jsonp跨域
由于同源策略，一般来说位于 server1.example.com 的网页无法与不是 server1.example.com的服务器沟通，而HTML的<script> 元素是一个例外。

利用<script>元素的这个开放策略，网页可以得到从其他来源动态产生的 JSON资料，而这种使用模式就是所谓的 JSONP。

用 JSONP 抓到的资料并不是 JSON，而是任意的JavaScript，用 JavaScript 直译器执行而不是用 JSON 解析器解析。

## 示例
```javascript
function handleResponse(response) {
   alert(`You get the data : ${response}`)
}
const script = document.createElement('script')
script.src = 'http://somesite.com/json/?callback=handleResponse'
document.body.insertBefore(script, document.body.firstChild)
```
这里的callback回调函数很重要，动态添加在body中的script标签可以使用被加载的文件与HTML文件下的其他JS文件共享一个全局作用域。

也就是说，<scritp>标签加载到的资源是可以被全局作用域下的函数所使用的！



