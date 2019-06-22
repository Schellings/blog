---
title: 短小精悍的GIF录制软件LICEcap
date: 2019-04-12 22:11:12
tags: 工具使用
---

> 平时学习和开发的过程中，总有一些软件可以提高工作或者开发效率，特此记录一下，并附上简单的入门教程，方便自己查找，但愿也可以帮助到您 [程序员必备软件](http://xiaweizi.cn/article/3cfa/)

## 前言

想必经常写博客的小伙伴经常会需要上传运行的效果图，也就是 `GIF` 动态图，目前有很多方式可供选择。

1. 先使用视频录制软件(比如QQ自带的录制)，然后通过某些转换工具(比如 `GIF Brewery`)将视频文件转换成 `GIF`格式
2. 或者直接使用某些 `GIF` 录制工具 `GifCam` 也很不错

<!-- more -->

那今天我要介绍的就是一款非常轻量级的但功能强大的 `GIF` 录制工具 — **LICEcap**。

> 对此有了解的请绕过。

## 介绍

[LICEcap](https://www.cockos.com/licecap/) 可以捕获桌面区域并将其直接保存为 `.gif` 文件，可直接查看或者在网页上查看。 **操作简单**、 **功能强大**、 **性能卓越**！这就是我一直使用它的原因。

作者还开源了并放在了 `Github` 上。 [源码](https://github.com/justinfrankel/licecap)

除了 `.gif` 格式之外， `LICEcap` 还支持自己原生无损 `.lcf` 文件格式，该格式允许比 `.gif` 更高的压缩比，更高的质量(每帧超过 256 色)，以及更精确的时间戳。

展示一下大致的效果(因为不想用别的录制软件来录制 `LICEcap` 的录制过程，就直接使用官网的效果)：

![官网展示效果](https://www.cockos.com/licecap/how_to_licecap.gif)

## 功能特点

拥有以下的功能特点：

- 可以录制为 `.gif` 或者是 `.lcf` 文件
- 在录制的过程中支持拖拽屏幕捕捉框
- 支持暂停和重新录制功能
- 支持添加标题功能
- 在录制的过程中，支持快捷键操作暂停功能
- 可调节录制帧率到最大来限制 `CPU` 的使用率
- 支持记录鼠标按钮按下
- 录制过程中展示录制时间

## 配置要求

- **Windows**。 Windows XP/Vista/7/8/8.1/10
- **OSX**。 macOS 10.4+
- **Linux**。 大部分
- 给力的 `CPU`
- 1GB+ 的 `RAM`

## 教程

### 操作预览

操作是非常简单的，看一下下面的录制预览图：

![预览](http://xiaweizi.top/2018-03-24-6D389141-DCC9-480A-8E5B-A3E5968E3DBB-1.png)

> 1. 录制的帧率。帧率越大，效果越好，文件大小越大
> 2. 录制区域大小。当然也可以通过拖拽的方式进行区域的选择
> 3. 开始
> 4. 暂停

## 基础配置

点击录制后，需要进行一些基础的配置。

![基础配置](http://xiaweizi.top/2018-03-24-47746539-37D1-469A-A000-7BE5FDAF3D05.png)

> 除了 **文件名称** 和 **保存路径** 这些必须配置外，还有一些可以选择配置的。
>
> 1. 标题的帧率
> 2. 录制的时间
> 3. 鼠标的点击
> 4. 标题内容

### 开始录制

![ 开始录制](http://xiaweizi.top/2018-03-24-DownloadView.gif)

我帧率调到了 **30**，还是蛮高的， **7** 秒钟，**2M** 多，效果还是蛮不错的。

## 获取

[LICEcap v1.28 for Windows](https://www.cockos.com/licecap/licecap128-install.exe) (更新于 12/2/17) (230kb)

[LICEcap v1.28 for OSX](https://www.cockos.com/licecap/licecap128.dmg) (更新于 12/2/17) (730kb)

[官网地址](https://www.cockos.com/licecap/)

[源码](https://github.com/justinfrankel/licecap)
