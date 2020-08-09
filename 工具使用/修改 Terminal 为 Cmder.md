---
title: 修改 IDEA Terminal 为 Cmder
date: 2018-01-06 18:56:24
tags: 工具使用
---
<meta name="referrer" content="no-referrer" />
# 修改 IDEA Terminal 为 Cmder

默认IDEA Terminal使用的是系统的cmd.exe，然而Cmder对于我们的使用更方便。因此用cmder替换cmd。

# 增加系统变量:::wq








变量名为：CMDER_ROOT

变量值为：C:\Application\cmder(主机上 Cmder 安装主目录)


![image](https://img-blog.csdnimg.cn/20190215173035153.png)

# IDEA设置

1、打开设置（快捷键：Ctrl + Alt + S），进入 Plugins,搜索栏搜索 Terminal，查看 Terminal 插件是否打勾选中，如果没有，请打勾。


2、进入设置（快捷键：Ctrl + Alt + S），进入 Tools，再进入 Terminal，在 Shell path 那一栏中，输入以下内容


```
"cmd.exe" /k ""%CMDER_ROOT%\vendor\init.bat""
```

![image](https://img-blog.csdnimg.cn/20190215173407238.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTIyOTQ1MTU=,size_16,color_FFFFFF,t_70)


3、重新启动 Idea IDE，然后打开 Terminal查看是否配置正确，下图已使用cmder。

![image](https://img-blog.csdnimg.cn/20190215173605164.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTIyOTQ1MTU=,size_16,color_FFFFFF,t_70)