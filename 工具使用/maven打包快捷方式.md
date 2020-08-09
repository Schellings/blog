---
title: idea中maven打包快捷方式
tags:
  - 工具使用
date: '2019-02-21 15:10'
---
<meta name="referrer" content="no-referrer" />

## 方法

（1）配置idea使用终端为cmder

（2）自定义cmder命令别名

文件：{cmder安装路径}/config/user_aliases.cmd

追加别名定义:

```yaml
mc= mvn clean
mt=mvn test

mi= mvn installl
mit= mcn install -Dmaven.test.skip=true

mct=mvn clean test
mci=mvn clean installl
mcit=mvn clean install -Dmaven.test.skip=true
mcp=mvn clean package
mcpt=mvn clean package -Dmaven.test.skip=true
```

（3）重启idea