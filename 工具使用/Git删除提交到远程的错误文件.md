---
title: Git删除提交到远程的错误文件
date: 2018-08-28 19:56:24
tags: 
    - 工具使用
    - Git
---

## 前言
用 git rm 来删除文件，同时还会将这个删除操作记录下来；
用 rm 来删除文件，仅仅是删除了物理文件，没有将其从 git 的记录中剔除。

直观的来讲，git rm 删除文件，执行git commit -m "msg"提交时，会自动将删除该文件的操作提交上去。

rm命令直接删除的文件，单纯执行git commit -m  "msg"提交时，则不会将删除该文件的操作提交上去，需要在执行commit的时候，多加一个参数a,

>例如 rm删除文件后，需要使用git commit -am "msg" 提交才会将删除文件的操作提交上去。

## 方法一

```yaml
git rm test.html
git commit -m '删除文件'
git push origin master
```

## 方法二

```yaml
rm test.html
git commit -am '删除文件'
git push origin master
```

## 方法三（删除目录）

```yaml
git rm -rf work
git commit -m '删除目录'
git push origin master
```




