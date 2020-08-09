---
title: 一台电脑上的git同时使用多个git仓库服务器配置
date: 2020-04-22 16:56:24
tags: Git
---

## 创建两个ssh证书

```yaml
1、根据每个repo用到的email生成ssh证书 ,填入自己在代码仓库中的邮箱帐号
 ssh-keygen -t rsa -b 4096 -C "tester@test.com"
2、根据不同的git仓库进行命名，后面需要给每个仓库配置证书，这里我采用默认路径
Enter a file in which to save the key (/Users/you/.ssh/id_rsa): 
3、输入密码，一般都是直接回车，每次都是免密 
Enter passphrase (empty for no passphrase):
Enter same passphrase again: 
4、这样就在/c/Users/win10/.ssh/id_rsa_test目录下生产了两个文件id_rsa和id_rsa.pub
5、执行上面同样的语句，再次生成一个证书 github_id_rsa
```

![](g1.png)

## ssh公钥注册

根据git服务器不同，将各自的ssh公钥注册到相应的git服务器，如github和gitlab

## 编辑配置文件

在 .ssh 目录下新建config文件 编辑如下内容

```yaml
#test
Host gitlab.test.com
User test
HostName gitlab.test.com
IdentityFile ~/.ssh/id_rsa

#test2
Host github.com
User test2
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_id_rsa

# Host和HostName 可以填写你的gitlab的服务器的主域名 gitlab.test.com
# User 填写你在代码仓库里的用户名（建议，不强制）
# IdentityFile 这个配置一定要配置正确。那个密码对应那个服务器，不能弄错。
```

完成之后，就可以愉快的在不同的服务器拉取代码。