---
title: Git常用命令
date: 2019-03-23 16:56:24
tags: Git
---
# Git常用命令总结
# git init

在本地新建一个repo,进入一个项目目录,执行git init,会初始化一个repo,并在当前文件夹下创建一个.git文件夹.
 
## git clone
获取一个url对应的远程Git repo, 创建一个local copy.

一般的格式是git clone [git url地址].

clone下来的repo会以url最后一个斜线后面的名称命名,创建一个文件夹,如果想要指定特定的名称,可以git clone [url] newname指定.

## git fetch
可以git fetch [alias]取某一个远程repo,也可以git fetch --all取到全部repo fetch将会取到所有你本地没有的数据,所有取下来的分支可以被叫做remote branches,它们和本地分支一样(可以看diff,log等,也可以merge到其他分支),但是Git不允许你checkout到它们. 
 
## git pull

git pull会首先执行git fetch,然后执行git merge,把取来的分支的head merge到当前分支.这个merge操作会产生一个新的commit.    

  

 
## git push
将会把当前分支merge到alias上的[branch]分支.如果分支已经存在,将会更新,如果不存在,将会添加这个分支.

## git status

查询repo的状态.
   
```
git status -s: -s表示short,
```
-s的输出标记会有两列,第一列是对staging区域而言,第二列是对working目录而言.
 
## git log
查看Git的提交日志


```
git log --oneline --number: 每条log只显示一行,显示number条.
git log --oneline --graph:可以图形化地表示出分支合并历史.
git log branchname       ：可以显示特定分支的log.
git log --author=[author name] 可以指定作者的提交历史.
git log --since --before --until --after 根据提交时间筛选log.
git log --grep=keywords  根据commit信息过滤keywords log: 
```

## git add
在提交之前,Git有一个暂存区(staging area),可以放入新添加的文件或者加入新的改动. commit时提交的改动是上一次加入到staging area中的改动,而不是我们disk上的改动.

命令：**git add .**

会递归地添加当前工作目录中的所有文件.
 
## git diff
此命令比较的是工作目录中当前文件和暂存区域快照之间的差异,也就是修改之后还没有暂存起来的变化内容.

若要看已经暂存起来的文件和上次提交时的快照之间的差异,可以用:

```
git diff --cached
```
 
 
## git commit
提交已经被add进来的改动.

```
git commit -m “the commit message"
```
-m参数天剑提交的提示信息




```
git commit -a
```
上述会先把所有已经track的文件的改动add进来,然后提交。


```
git commit --amend 
```
 增补提交，上述命令会使用与当前提交节点相同的父节点进行一次新的提交,旧的提交将会被取消.
 
## git reset
下面的的HEAD关键字指的是当前分支最末梢最新的一个提交.也就是版本库中该分支上的最新版本.

```
git reset HEAD
```
这个命令用来把不小心add进去的文件从staged状态取出来,可以单独针对某一个文件操作: git reset HEAD  --filename, 这个--filename参数也可以不加.

## git revert
反转撤销提交.只要把出错的提交(commit)的名字(reference)作为参数传给命令就可以了.

```
git revert HEAD
```
撤销最近的一个提交.

    
## git rm
git rm file: 从staging区移除文件,同时也移除出工作目录.

git rm --cached: 从staging区移除文件,但留在工作目录中.

git rm --cached从功能上等同于git reset HEAD,清除了缓存区,但不动工作目录树.
 
## git clean
git clean是从工作目录中移除没有track的文件.
通常的参数是git clean -df

-d表示同时移除目录,-f表示force,因为在git的配置文件中,clean.requireForce=true,如果不加-f,clean将会拒绝执行.
 

## git stash
把当前的改动压入一个栈.
     
git stash将会把当前目录和index中的所有改动(但不包括未track的文件)压入一个栈,然后留给你一个clean的工作状态,即处于上一次最新提交处.
git stash list 会显示这个栈的list.
git stash apply 取出stash中的上一个项目(stash@{0}),并且应用于当前的工作目录.

常用语解决在git分支上做错了修改，比如，原本想在dev分支上做修改，确没注意在master分支上做了修改，此时，可使用git stash系列命令来将修改应用到dev分支

 
## git branch
git branch可以用来列出分支,创建分支和删除分支.

git branch -v可以看见每一个分支的最后一次提交.

git branch: 列出本地所有分支,当前分支会被星号标示出.
git branch (branchname): 创建一个新的分支(当你用这种方式创建分支的时候,分支是基于你的上一次提交建立的). 

git branch -a：查看所有分支

git branch -d (branchname): 删除一个分支.

 
## git checkout
切换到一个分支.
git checkout -b (branchname): 创建并切换到新的分支.

这个命令是将git branch newbranch和git checkout newbranch合在一起的结果.

checkout还有另一个作用:替换本地改动。

git checkout --<filename>
此命令会使用HEAD中的最新内容替换掉你的工作目录中的文件.已添加到暂存区的改动以及新文件都不会受到影响.

**注意:git checkout filename会删除该文件中所有没有暂存和提交的改动,这个操作是不可逆的.**
 
## git merge

把一个分支merge进当前的分支.

git merge [alias]/[branch]

把远程分支merge到当前分支.

 
## git tag
会在一个提交上建立永久性的书签,通常是发布一个release版本或者ship了什么东西之后加tag.

比如: git tag v1.0
git tag -a v1.0, -a参数会允许你添加一些信息,即make an annotated tag.

当你运行git tag -a命令的时候,Git会打开一个编辑器让你输入tag信息.
     
 
## git remote
管理remote repo的url

因为不需要每次都用完整的url,所以Git为每一个remote repo的url都建立一个别名,然后用git remote来管理这个list.

```
     git remote: 列出remote aliases.
     如果你clone一个project,Git会自动将原来的url添加进来,别名就叫做:origin.
     git remote -v:可以看见每一个别名对应的实际url.
     git remote add [alias] [url]: 添加一个新的remote repo.
     git remote rm [alias]: 删除一个存在的remote alias.
     git remote rename [old-alias] [new-alias]: 重命名.
     git remote set-url [alias] [url]:更新url. 可以加上—push和fetch参数,为同一个别名set不同的存取地址.
```

 
## 特殊符号:
^代表父提交,当一个提交有多个父提交时,可以通过在^后面跟上一个数字,表示第几个父提交: \^相当于^1.

