---
title: 一、Linux基本命令
date: 2019-03-03 16:56:24
tags: Linux
---

# 一、Linux基本命令
## 1.1 关机和重启
### 关机

```
// 立刻关机:
shutdown -h now
```


```
// 5分钟后关机
shutdown -h 5
```


```
// 立刻关机
powerof 
```

### 重启


```
// 立刻重启
shutdown -r now
```

```
// 5分钟后重启
shutdown -r 5
```

```
// 立刻重启
reboot
```

## 1.2 帮助命令

### --help命令

命令格式：Linux命令 --help

```
// 举例:
shutdown --help
ifconfig --help
```

### man 命令（命令说明书）

命令格式：man Linux命令

```
// 举例：
man shutdown
man ifconfig
// 注意：使用man指令打开命令说明书之后，使用按键q退出
```

