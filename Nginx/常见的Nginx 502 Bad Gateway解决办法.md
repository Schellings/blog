---
title: 常见的Nginx 502 Bad Gateway解决办法
date: 2018-09-10 16:56:24
tags: 
    - Nginx
---

## Nginx 502错误情况1：
网站的访问量大，而php-cgi的进程数偏少。

针对这种情况的502错误，只需增加php-cgi的进程数。具体就是修改/usr/local/php/etc/php-fpm.conf 文件，将其中的max_children值适当增加。这个数据要依据你的VPS或独立服务器的配置进行设置。一般一个php-cgi进程占20M内存，你可以自己计算下，适量增多。

/usr/local/php/sbin/php-fpm restart 然后重启一下.

## Nginx 502错误情况2

CPU占用率、内存占用率非常高，遭到CC攻击.

解决方法请参考：LinuxVPS简单解决CC攻击

## Nginx 502错误情况3

CPU占用率不高，内存溢出。

检查一下网站程序有没有问题？一般小偷站点常常会出现内存溢出。

检查一下/var/log/目录下的日志，看看是不是有人爆破SSH和FTP端口？

SSH、FTP遭到穷举也会占用大量内存。是的话改掉SSH端口和FTP端口即可

将网上找到的一些和502 Bad Gateway错误有关的问题和排查方法列一下，先从FastCGI配置入手。

### 1.查看FastCGI进程是否已经启动

NGINX 502错误的含义是sock、端口没被监听造成的。我们先检查fastcgi是否在运行

### 2.检查系统Fastcgi进程运行情况

除了第一种情况，fastcgi进程数不够用、php执行时间长、或者是php-cgi进程死掉也可能造成nginx的502错误

运行以下命令判断是否接近FastCGI进程，如果fastcgi进程数接近配置文件中设置的数值，表明worker进程数设置太少

netstat -anpo | grep "php-cgi" | wc -l

### 3.FastCGI执行时间过长

根据实际情况调高以下参数值

fastcgi_connect_timeout 300; 
 
fastcgi_send_timeout 300; 
 
fastcgi_read_timeout 300;

### 4.头部太大

nginx和apache一样，有前端缓冲限制，可以调整缓冲参数

fastcgi_buffer_size 32k; 
fastcgi_buffers 8 32k;

如果你使用的是nginx的负载均衡Proxying，调整

proxy_buffer_size 16k; 
proxy_buffers 4 16k;

### 5.https转发配置错误

正确的配置方法
```yaml
server_name www.jb51.net; location /myproj/repos 
{ 
 set $fixed_destination $http_destination; 
 if ( $http_destination ~* ^https(.*)$ )
     { 
  set $fixed_destination http$1;
  } 
 proxy_set_header Host $host; 
 proxy_set_header X-Real-IP $remote_addr; 
 proxy_set_header Destination $fixed_destination; 
 proxy_pass http://subversion_hosts; 
}
```






