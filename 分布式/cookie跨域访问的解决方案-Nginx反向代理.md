---
title: cookie跨域访问的解决方案-Nginx反向代理
date: 2019-05-17 19:56:24
tags: 
    - Cookie跨域
    - Nginx
    - 反向代理
---

## 什么是跨域

随着项目模块越来越多 ，很多模块现在都是独立部署，模块之间的交流有时可能会通过cookie完成 。
 
 比如说门户和应用部署在不同的机器或者web容器中 ，假如用户登录之后会在浏览器客户端写入cookie （记录着用户上下文信息） ，应用想要回去门户下的cookie ，这就产生了cookie跨域的问题 。

## cookie介绍

### （1）cookie路径 ：

cookie 一般是由与用户访问页面而被创建的，可是并不是只有在创建 cookie的页面才可以访问这个cookie。

在默认情况下，出于安全方面的考虑，**只有与创建 cookie 的页面处于同一个目录或在创建cookie页面的子目录下的网页才可以访问。** 

那么此时如果希望其父级或者整个网页都能够使用cookie，就需要进行路径的设置。

path表示cookie所在的目录，asp.net默认为/，就是根目录。

在同一个服务器上有目录如下：/test/,/test/cd/,/test/dd/。

现设一个cookie1的path为/test/，cookie2的path为/test/cd/，

那么test下的所有页面都可以访问到cookie1，而/test/和/test/dd/的子页面不能访问cookie2。

这是因为cookie能让其path路径下的页面访问。


### 域：

domain表示的是cookie所在的域，默认为请求的地址，如网址为www.jb51.net/test/test.aspx，那么domain默认为www.jb51.net。

而跨域访问，如域A为t1.test.com，域B为t2.test.com，那么在域A生产一个令域A和域B都能访问的cookie就要将该cookie的domain设置为.test.com。

如果要在域A生产一个令域A不能访问而域B能访问的cookie就要将该cookie的domain设置为t2.test.com。


## Nginx反向代理

**反向代理概念：**

可参照我的另一篇博客：[传送门](https://blog.xielin.top/2019/01/25/Nginx/Nginx%E4%BB%A3%E7%90%86%E4%B8%8E%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1/)

**场景模拟：**
- 两个工程web1, web2, 部署在同一台机器上的不同tomcat上，请求web1工程的index.html,如下：

![](https://images2015.cnblogs.com/blog/640632/201608/640632-20160806191910934-73557688.png)

- 然后点击链接请求web2工程的index.jsp, 内容如下：

![](https://images2015.cnblogs.com/blog/640632/201608/640632-20160806192012075-901400950.png)

- nginx配置如下：

```yaml
worker_processes  2; 
events {
    worker_connections  65535;
}
http {
    include       mime.types;
    default_type  application/octet-stream;

   log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    server_names_hash_bucket_size 128;
    client_header_buffer_size 32k;
    large_client_header_buffers 4 32k;
    client_body_buffer_size    8m;
    server_tokens off;
    ignore_invalid_headers   on;
    recursive_error_pages    on;
    server_name_in_redirect off;
    sendfile        on;
    tcp_nopush     on;
    tcp_nodelay    on;
    #keepalive_timeout  0;
    keepalive_timeout  65;
    upstream web1{
         server  127.0.0.1:8089  max_fails=0 weight=1;
    }
    upstream web2 {
         server 127.0.0.1:8080    max_fails=0 weight=1;
    }

    server {
        listen       80;
        server_name  127.0.0.1;
        charset utf-8;
        index index.html;

        location /web/web1 {
            proxy_pass http://web1/web1;
            proxy_set_header Host  127.0.0.1;
            proxy_set_header   X-Real-IP        $remote_addr;
            proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;

            proxy_set_header Cookie $http_cookie;
            log_subrequest on;
        }

        location /web/web2 {
            proxy_pass http://web2/web2;
            proxy_set_header Host  127.0.0.1;
            proxy_set_header   X-Real-IP        $remote_addr;
            proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
            proxy_set_header Cookie $http_cookie;
            log_subrequest on;
        }

        location /nginxstatus {
            stub_status on;
            access_log on;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}

```
- 这样就保证了cookie 在同一域下， web2工程中的index.jsp中的输出内容如下：

![](https://images2015.cnblogs.com/blog/640632/201608/640632-20160806192307512-1773158144.png)

## 技术总结

利用nginx的方向代理来解决cookie跨域问题，其实是通过“欺骗”浏览器来实现的。

通过nginx，我们可以将不同工程的cookie放到nginx域下，通过nginx反向代理就可以取到不同工程写入的cookie。





