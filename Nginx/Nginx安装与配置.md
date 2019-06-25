---
title: Nginx代理与负载均衡
date: 2019-01-24 16:56:24
tags: 
    - Nginx
---


## 添加源
```yaml
sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```
## 安装
```yaml
sudo yum install -y nginx
```
## 启动Nginx并设置开机自动运行
```yaml
sudo systemctl start nginx.service
sudo systemctl enable nginx.service
```
## 配置文件 nginx.conf

文件所在位置：/etc/nginx/nginx.conf
常用配置：静态代理+端口转发
### 端口转发 proxy_pass
```yaml
    server {
        server_name blog.xielin.top;
        listen 80;
    	charset utf-8;
    	access_log off;
        location / {
            proxy_pass   http://127.0.0.1:4000/;
                        proxy_read_timeout 3000;
            }
    }
```

### 静态代理
```yaml
server {
	server_name chatbot.xielin.top;
	listen 80;
    charset utf-8;
  		location / {
  		root /root/WorkSpace/html/chatbot; 
  			index index.html;
  		}	
    }
```
