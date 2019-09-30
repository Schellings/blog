---
title: Redis安装后的常用配置（for windows）
date: 2018-04-17 19:11:12
tags: Redis
---
## 设置开机自动启动

### 打开cmd窗口并输入：services.msc
![服务](https://img-blog.csdn.net/20180125220953786?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGtoaGti/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
找到Redis选择自动启动即可。

## 设置密码

进入Redis的安装目录打开：redis.windows-conf。

按下Ctrl+F查找：requirepass，找到编辑保存然后重新启动Redis服务即可。

（注意：这里是redis.windows-conf，windows自启动会加载这个配置文件）。

![Redis配置文件](https://img-blog.csdn.net/20180125235022199?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGtoaGti/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 绑定Redis的IP
按下Ctrl+F查找：127.0.0.1，找到编辑保存然后重新启动Redis服务即可。

![Redis配置文件](https://img-blog.csdn.net/20180125235857019?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGtoaGti/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)