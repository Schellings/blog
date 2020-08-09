---
title: AD获取员工照
date: 2020-04-05 19:56:24
tags: 
    - LDAP
---
<meta name="referrer" content="no-referrer" />

## 获取员工照片
用户头像其实对应的是AD中一个叫thumbnailphoto的属性，根据微软给的参数解释，这个属性可以存放100K大小的位图，也就是差不多96像素见方的图片，.
那从AD中导出这个属性也就是一堆100K以下的小图片

