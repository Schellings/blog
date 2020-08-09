---
title: maven install提示@return异常
date: 2018-02-24 16:56:24
tags: Maven
---
<meta name="referrer" content="no-referrer" />
## @Return异常

该问题的引发是maven在打包时会尝试生成javadoc文件，一旦在方法的注释@return上没有做信息说明，即引发该问题

## 问题的解决

打包时跳过Javadoc的生成

```yaml
 mvn clean install -Dmaven.test.skip=true -Dmaven.javadoc.skip=true
```
