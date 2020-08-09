---
title: W3C 发布分散式标识符（DID）规范
date: 2019-12-20 19:56:24
tags: 
    - 区块链
    - DID规范
---
<meta name="referrer" content="no-referrer" />
## 背景
2019年11月7日，W3C 分散式标识符工作组（[Decentralized Identifier Working Group](https://www.w3.org/2019/did-wg/)）发布分散式标识符：核心数据模型和语法（[Decentralized Identifiers (DIDs) v1.0](https://www.w3.org/TR/2019/WD-did-core-20191107/)）规范的首个公开工作草案（First Public Working Draft）。

## 简介
分散式标识符（[DIDs](https://www.w3.org/TR/2019/WD-did-core-20191107/#dfn-did-controller-s)）是用于可验证的去中心化数字身份的一种新型标识符。 这些新的标识符旨在使 DID 的控制器能够证明对它的控制，并且可以独立于任何集中注册表、身份提供者或证书认证机构而实施。  
DIDs 是通过 DID 文档的方式将 DID 主题（[DID subject](https://www.w3.org/TR/2019/WD-did-core-20191107/#dfn-did-subject)）和与之进行可信交互的方法联系起来的 URLs。DID 文档（DID document）是描述如何使用特定 DID 的简单文档。 每个 DID 文档都可以表示加密材料、验证方法和/或服务端点（[service endpoint](https://www.w3.org/TR/2019/WD-did-core-20191107/#dfn-service-endpoint)）。   
这些提供了一组使 DID 控制器（[DID controller](https://www.w3.org/TR/2019/WD-did-core-20191107/#dfn-did-controller-s)）能够证明对 DID 控制的机制。 服务端点实现了与 DID 主题的可信交互。

这份规范为 DIDs、DID 文档以及 DID 方法指定了一个通用数据模型、一个 URL 格式以及一组操作。

更多内容，请参阅[英文原文](https://www.w3.org/blog/news/archives/8032)。