---
title: 使用GitBook编写文档书籍
date: 2018-09-28 19:56:24
tags: 
    - 工具使用
---

## 前言

GitBook 是一个基于 Git+Node.js 的命令行工具，可使用 Github/Git 和 Markdown 来制作精美的电子书。GitBook支持输出以下几种文档格式：

- 静态站点：GitBook默认输出该种格式
- PDF：需要安装gitbook-pdf依赖
- eBook：需要安装ebook-convert
- GitBook可以用来写书、API文档、公共文档，企业手册，论文，研究报告等。


## GitBook安装

- [nodejs安装](https://blog.xielin.top/2018/04/15/Node.js/CentOS7%E5%88%A9%E7%94%A8yum%E5%BF%AB%E9%80%9F%E5%AE%89%E8%A3%85node/)
- [git安装](https://blog.xielin.top/2018/03/20/Git/Git%E7%8E%AF%E5%A2%83%E5%AE%89%E8%A3%85/)
- gitbook安装

在安装之前，我们还需要配置一下 nodejs 插件安装的下载镜像地址。因为默认的镜像地址在国外，需要翻墙才可以访问，因此我们需要设置国内的镜像地址。这里使用阿里巴巴的镜像地址 http://registry.npm.taobao.org 。

使用下面的命令，进行配置。

```shell
npm config set registry http://registry.npm.taobao.org
```

安装命令：

```yaml
npm install gitbook -g
//安装命令
npm install -g gitbook-cli

//卸载命令
npm uninstall -g gitbook
```

检验下是否安装成功

```yaml
//显示gitbook以及gitbook-cli版本号
gitbook -V
```

## 基本使用

Gitbook需要2个基本文件：

README.md

SUMMARY.md

README.md是关于你的书的介绍，而SUMMARY.md中则包含了书的目录。我们还可以添加“book.json（电子书配置文件）”、“GLOSSARY.md（书尾词汇表）”以及“封面图片”等，默认文件树中没有这些，需要自行添加。

### 生成图书目录结构
创建一个目录test， 使用gitbook init命令就可以在目录下生产这两个文件。一个SUMMARY.md文件的格式大致如下，每一行对应一个相应的文件。

```yaml
# Summary

* [Introduction](README.md)
* [第一章](chapter1/README.md)
    * [第一节](chapter1/seciont1.md)
    * [第二节](chapter1/section2.md)
* [第二章](chapter2/README.md)
    * [第一节](chapter2/seciont1.md)
    * [第二节](chapter2/section2.md)
* [结束](end/README.md)

```
执行 gitbook init 会根据 SUMMARY.md 目录生成对应的文件夹和 md 文件，每一个 md 文件对应每一章节，每一章节的内容在对应的 md 文件里编辑。

你可以使用本地支持Markdown语法的编辑器编辑，例如Markdown Pad。

如果想要新增章节，可以在 SUMMARY.md 里面新增，然后执行 gitbook init 就会新增对应的 md 文件，原有文件不会变化；如果想要删除章节，在 SUMMARY.md 里面删除，然后执行 gitbook init 想要删除的 md 文件并不会删除，需要手动删除。

### 生成图书
#### 输出为静态网站
执行下面的命令

gitbook serve
> 该命令在windows系统上执行时经常会发生找不到相关静态文件的问题，无需理会，多执行几遍即可

执行之后访问 http://localhost:4000 即可访问

这里你会发现，你在你的图书项目的目录中多了一个名为_book的文件目录，而这个目录中的文件，即是生成的静态网站内容。

使用build参数生成到指定目录

与直接预览生成的静态网站文件不一样的是，使用这个命令，你可以将内容输入到你所想要的目录中去：

```yaml
$ mkdir /tmp/gitbook
$ gitbook build --output=/tmp/gitbook
```

#### 输出为PDF
输出为PDF文件，需要先使用NPM安装上gitbook pdf：
```yaml
$ sudo npm install gitbook-pdf -g
```
### Gitbook的插件支持

新建book.json，可以做一些配置，比如标题，作者，指定readme文件，关闭分享链接等。

```json
{
  "title": "gitbook tutorial",
  "description": "Webpack 是当下最热门的前端资源模块化管理和打包工具，本书大部分内容翻译自 Webpack 官网。",
  "structure": {
    "readme": "README.md"
  },
  "links": {
    "gitbook": false,
    "sharing": {
      "google": false,
      "facebook": false,
      "twitter": false,
      "all": false
    }
  },

  "plugins": ["disqus","advanced-emoji"],
  "pluginsConfig": {
    "disqus": {
        "shortName": "gitbookuse"
    }
  }
}
```

安装插件:
```yaml
$ gitbook install .
```

### Gitbook常用命令

- gitbook init //初始化目录文件
- gitbook help //列出gitbook所有的命令
- gitbook --help //输出gitbook-cli的帮助信息
- gitbook build //生成静态网页
- gitbook serve //生成静态网页并运行服务器
- gitbook build --gitbook=2.0.1 //生成时指定gitbook的版本, 本地没有会先下载
- gitbook ls //列出本地所有的gitbook版本
- gitbook ls-remote //列出远程可用的gitbook版本
- gitbook fetch 标签/版本号 //安装对应的gitbook版本
- gitbook update //更新到gitbook的最新版本
- gitbook uninstall 2.0.1 //卸载对应的gitbook版本
- gitbook build --log=debug //指定log的级别
- gitbook builid --debug //输出错误信息







