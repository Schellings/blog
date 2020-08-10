---
title: Linux命令小技巧（长期更新）
date: 2020-05-05 16:56:24
tags: Linux
---
<meta name="referrer" content="no-referrer" />

## 查找文件内容

在一些错误日志中，会明显提示某一个配置校验错误，但是由于配置文件众多，不清楚该环境具体是将该配置放置在哪个配置文件，可使用该命令来批量递归查找

```shell script
grep -rn "keyword" .
```

在当前目录递归查找关键字，并输出相关行号

## vim行数跳转

接上文，找到相关行之后
```shell script
vim application-dev.yml
:line number
```
