---
title: Docker常用命令总结
date: 2018-04-05 16:56:24
tags: 容器
---
<meta name="referrer" content="no-referrer" />
# 1、docker的启动、停止、重启
- 启动

```
sudo systemctl start docker
```

- 停止

```
sudo systemctl stop docker
```

- 重启

```
sudo systemctl restart docker
```
# 2、镜像相关
-  查看镜像       

```
 docker images
```
![image](https://img-blog.csdnimg.cn/20190309192326167.png)
- 搜索镜像       

```
docker search 镜像名称
```
![image](https://img-blog.csdnimg.cn/20190309192902566.png)
-  拉取镜像          

```
docker pull 镜像名称
```
执行后会进行下载（默认下载最新版，即latest版）
![image](https://img-blog.csdnimg.cn/20190309192338778.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RlbmdmZW5nYW4=,size_16,color_FFFFFF,t_70)

- 按镜像ID删除镜像     

```
docker rmi 镜像ID
```
其中镜像ID可通过命令docker images查看IMAGE ID栏
![image](https://img-blog.csdnimg.cn/20190309192427240.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RlbmdmZW5nYW4=,size_16,color_FFFFFF,t_70)

- 批量删除镜像      

1、删除所有容器

```
docker rm `docker ps -a -q`
```

2、删除所有镜像

```
docker rmi `docker images -q`
```

3、按条件删除镜像（没有打标签）

```
docker rmi `docker images -q | awk '/^<none>/ { print $3 }'`
```

4、镜像名包含关键字


```
docker rmi --force `docker images | grep doss-api | awk '{print $3}'`
//其中doss-api为关键字
```
   
# 3、容器相关
- 查看正在运行的容器          

```
docker ps
```
![image](https://img-blog.csdnimg.cn/20190309192513871.png)
- 查看所有容器

```
docker ps –a
```
![image](https://img-blog.csdnimg.cn/20190309192539368.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RlbmdmZW5nYW4=,size_16,color_FFFFFF,t_70)
- 查看最后一次运行的容器  

```
docker ps –l
```
![image](https://img-blog.csdnimg.cn/20190309192539370.png)

- 查看停止的容器  

```
docker ps -f status=exited
```
![image](https://img-blog.csdnimg.cn/20190309192539388.png)

- 创建容器命令

```
docker run
```
**参数说明**：

-i：表示运行容器

-t：表示容器启动后会进入其命令行。加入这两个参数后，容器创建就能登录进去。即分配一个伪终端。

--name :为创建的容器命名。

-v：表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使用多个－v做多个目录或文件映射。注意：最好做目录映射，在宿主机上做修改，然后共享到容器上。

-d：在run后面加上-d参数,则会创建一个守护式容器在后台运行（这样创建容器后不会自动登录容器，如果只加-i -t两个参数，创建后就会自动进去容器）。

-p：表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个-p做多个端口映射

**示例**：

**1. 交互式方式创建容器**


```
docker run -it --name=容器名称 镜像名称:标签 /bin/bash
```
![image](https://img-blog.csdnimg.cn/2019030919261256.png)

**退出当前容器**

```
exit
```
![image](https://img-blog.csdnimg.cn/20190309192628854.png)

**2、守护式方式创建容器**

```
docker run -di --name=容器名称 镜像名称:标签
```
![image](https://img-blog.csdnimg.cn/20190309192628863.png)

![image](https://img-blog.csdnimg.cn/20190309192628877.png)

**登录守护式容器**

```
docker exec -it 容器名称 (或者容器ID) /bin/bash
```

![image](https://img-blog.csdnimg.cn/20190309192628883.png)

- 停止容器

```
docker stop 容器名称（或者容器ID）
```
![image](https://img-blog.csdnimg.cn/20190309192641782.png)

- 启动容器

```
docker start 容器名称（或者容器ID）
```
![image](https://img-blog.csdnimg.cn/20190309192641788.png)

- 文件拷贝

```
docker cp 需要拷贝的文件或目录 容器名称:容器目录
```

![image](https://img-blog.csdnimg.cn/20190309192658626.png)


- 查看容器运行的各种数据           

```
docker inspect 容器名称（容器ID）
```

![image](https://img-blog.csdnimg.cn/20190309192658811.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RlbmdmZW5nYW4=,size_16,color_FFFFFF,t_70)

- 查看容器IP地址

```
docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名称（容器ID）
```
![image](https://img-blog.csdnimg.cn/20190309192714459.png)

- 查看指定容器的日志记录


```
docker logs -f 容器名称（容器ID）
```

# 4、案例
## MySQL容器部署

```
docker run -di --name=test_mysql -p 33308:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
```
-p 代表端口映射，格式为 宿主机映射端口:容器运行端口

-e 代表添加环境变量 MYSQL_ROOT_PASSWORD 是root用户的登陆密码

mysql 代表镜像名称

![image](https://img-blog.csdnimg.cn/20190309192738638.png)