---
title: Nignx SSL 设置，开启HTTPS
date: 2018-03-10 16:56:24
tags: 
    - Nginx
---
<meta name="referrer" content="no-referrer" />


## HTTPS 配置

配置nginx配置文件`nginx.conf`

```json
server {
        ssl_certificate 4327634_blog.xielin.top.pem;  # 指定证书的位置，绝对路径
        ssl_certificate_key 4327634_blog.xielin.top.key;  # 绝对路径，同上
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #按照这个协议配置
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;#按照这个套件配置
        ssl_prefer_server_ciphers on;
        server_name blog.xielin.top;
        listen 443 default ssl;
        charset utf-8;
        access_log off;
        location / {
            proxy_pass   http://127.0.0.1:4000/;
                        proxy_read_timeout 3000;
            }
    }
```


## HTTP自动转HTTPS

```json
server {
        listen       80;
        server_name  blog.xielin.top;
        rewrite ^(.*)$ https://$host$1 permanent;
        }
```

重启nginx

```shell script
nginx -s reload
```



