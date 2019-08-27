---
title: maven中scope标签作用
date: 2019-07-25 16:56:24
tags: Maven
---

scope 是用来限制 dependency 的作用范围的，影响 maven 项目在各个生命周期时导入的 package 的状态，主要管理依赖的部署。

scope 的作用范围：

（1）compile：默认值，适用于所有阶段（表明该 jar 包在编译、运行以及测试中路径均可见），并且会随着项目一起发布。

（2）test：只在测试时使用，用于编译和运行测试代码，不会随项目发布。

（3）runtime：无需参与项目的编译，不过后期的测试和运行周期需要其参与，与 compile 相比，跳过了编译。如 JDBC 驱动，适用运行和测试阶段。

（4）provided：编译和测试时有效，但是该依赖在运行时由服务器提供，并且打包时也不会被包含进去。如 servlet-api。

（5）system：类似 provided，需要显式提供包含依赖的jar，不会从 maven 仓库下载，而是从本地文件系统获取，需要添加 systemPath 的属性来定义路径。