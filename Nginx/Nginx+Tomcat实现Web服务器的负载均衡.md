---
title: Nginx+Tomcat实现Web服务器的负载均衡
date: 2018-05-15 19:56:24
tags: 
    - Nginx
    - 负载均衡
---
## 拓扑环境

服务器名称	| 系统版本 |	预装软件 |	IP地址 
---|---|---|---
Nginx服务器	| CentOS 7 最小安装	| Nginx |	192.168.22.227
Web服务器A	| CentOS 7 最小安装	|tomcat+jdk	|192.168.22.229
Web服务器B	| CentOS 7 最小安装 |tomcat+jdk	|192.168.22.230

服务器采用CentOS 7 最小安装模式，完全模拟生成环境，一台Nginx服务器，两台Tomcat服务器，实现一个简化的反向代理和负载均衡服务。 

## 原理图
![](https://img-blog.csdn.net/20160108202202273)

## 编写测试静态页

- 在229服务器编写 Login.html:
```html
<html>
    <head>
        <meta http-equiv="content-type" content="text/html;charset=gb2312" />
    </head>
    <body>
        <h1>您正在访问：192.168.22.229</h1>
    </body>
</html>
```
- 在Tomcat的webapps目录下，新建一个文件夹drp，并将login.html放到drp文件夹里。

- 同样的在230服务器上也新建文件：login.html，并上传到drp目录下。
```html
<html>
    <head>
        <meta http-equiv="content-type" content="text/html;charset=gb2312" />
    </head>
    <body>
        <h1>您正在访问：192.168.22.230</h1>
    </body>
</html>
```
编写完成后，启动229,230服务器上的Tomcat，并在windows上测试是否启动成功

分别输入url：

http://192.168.22.229:8080/drp/login.html

http://192.168.22.230:8080/drp/login.html

测试是否能够访问。

## 修改Nginx核心配置文件nginx.conf

下面配置文件中的几个关键点：

**（1）进程数与每个进程的最大连接数**

```yaml
#工作进程个数，一般跟服务器cpu核数相等，或者核数的两倍
worker_processes 2;

#单个进程最大连接数
events{
    worker_connections 1024; 
}```
```

（1） nginx进程数，建议设置为和服务器cup核数相等，或者是核数的两倍

（2）单个进程最大连接数，该服务器的最大连接数=连接数*进程数； 

服务器支持最大并发数=(连接数*进程数) /2 ，因为反向代理是双向的。

**（2）Nginx的基本配置**
```yaml
server{
    listen 8088;    #端口号
    server_name 192.168.22.227; #服务名
}
```
（1）监听端口一般都为http端口：80;可以修改为其他，这里修改为8088。

（2）server_name ：默认为localhost ，这里修改为服务器ip地址。

**（3）负载均衡列表基本配置**

```yaml
    #服务器集群
    upstream mycluster{
        #这里添加的是上面启动好的两台Tomcat服务器
         server 192.168.22.229:8080 weight=1;
         server 192.168.22.230:8080 weight=1;
    }

location /{
        #将访问请求转向至服务器集群,mycluster和上面upstream mycluster 对应
        proxy_pass http://mycluster;
        # 真实的客户端IP
        proxy_set_header   X-Real-IP        $remote_addr; 
        # 请求头中Host信息
        proxy_set_header   Host             $host; 
        # 代理路由信息，此处取IP有安全隐患
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        # 真实的用户访问协议
        proxy_set_header   X-Forwarded-Proto $scheme;
    }

```
（1） location / {}：负载均衡访问的请求，可以添加筛选，假如我们要对所有的jsp后缀的文件进行负载均衡时，可以这样写：location ~ .*.jsp$ {}

（2） proxy_pass：请求转向自定义的服务器列表，这里我们将请求都转向标识为http://mycluster 的负载均衡服务器列表；

（3）在负载均衡服务器列表的配置中，Server指令：指定服务器的ip地址，weight是权重，可以根据机器配置定义权重。

weigth参数表示权值，权值越高被访问到的几率越大。

（如果某台服务器的硬件配置十分好，可以处理更多的请求，那么可以为其设置一个比较高的weight；而有一台的服务器的硬件配置比较差，那么可以将前一台的weight配置为weight=2，后一台差的配置为weight=1）。

**（四）完整的配置文件示例**
```yaml
user nobody;
#工作进程个数，一般跟服务器cpu核数相等，或者核数的两倍
worker_processes 2;

#单个进程最大连接数
events{
    worker_connections 1024; 
}

http{

    keepalive_timeout 65;
    gzip on;
    #服务器集群
    upstream mycluster{
         #集群有几台服务器即可配置几台，weight表示权重，权重越大被访问到的几率越大
        #这里添加的是上面启动好的两台Tomcat服务器
         server 192.168.22.229:8080 weight=1;
         server 192.168.22.230:8080 weight=1;
    }

    #nginx基本配置
    server{
        listen 8088;    #端口号
        server_name 192.168.22.227; #服务名
        location /{
            #将访问请求转向至服务器集群,mycluster和上面upstream mycluster 对应
            proxy_pass http://mycluster;
            # 真实的客户端IP
            proxy_set_header   X-Real-IP        $remote_addr; 
            # 请求头中Host信息
            proxy_set_header   Host             $host; 
            # 代理路由信息，此处取IP有安全隐患
            proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
            # 真实的用户访问协议
            proxy_set_header   X-Forwarded-Proto $scheme;
        }

        error_page   500 502 503 504  /50x.html;  
        location = /50x.html {  
            root   html;  
        }  

    }
}

```

## 负载均衡测试

在浏览器中输入 : http://192.168.22.227:8088/drp/login.html

![](https://img-blog.csdn.net/20160108205816996)
