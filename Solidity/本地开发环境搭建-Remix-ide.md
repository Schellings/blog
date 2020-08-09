---
title: 本地开发环境搭建-Remix-ide
date: 2019-08-01 19:56:24
tags: 
    - 智能合约
    - Solidity
---
<meta name="referrer" content="no-referrer" />

## 写在前面

[在线的Remix-ide环境](https://remix.ethereum.org/#optimize=true&version=soljson-v0.4.26+commit.4563c3fc.js&evmVersion=null)受限于b/s架构和网络原因，导致用起来十分不方便，本教程在win10本地搭建Remix-ide环境。

## 安装步骤

- 没有安装过git的话就下载安装windows版本的git软件：“Git-2.21.0-64-bit.exe”
- 没有安装node软件的话就要下载安装。就用我实验成功的“node-v10.15.3-x64.msi”吧。
- remix软件需要C++和python2.7，所以先安装C++工具包。下载vs2017社区版本

运行它，开始安装过程，注意一定要选中“C++桌面开发”、“通用Windows平台开发”。

![](https://img-blog.csdnimg.cn/20190502232856351.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTIwODQ4Mjc=,size_16,color_FFFFFF,t_70)


- 管理员运行打开cmd
- 执行以下命令，安装windows工具包
```js
npm install --global --production windows-build-tools
```

- 安装remix-ide
```js
npm install remix-ide –g
```
该命令会在当前文件夹创建remix-ide文件夹

- 启动remix-ide

![](https://img-blog.csdnimg.cn/20190502232856177.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTIwODQ4Mjc=,size_16,color_FFFFFF,t_70)


- 访问

打开浏览器，输入地址：http://localhost:8080/

- 插件激活

点击右侧的插头图标，显示出插件列表，激活必要的两项：Run Transations”，“Solidity Compiler”