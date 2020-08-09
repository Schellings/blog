---
title: DIDs规范详解
date: 2020-01-15 19:56:24
tags: 
    - 区块链
    - DID规范
---
<meta name="referrer" content="no-referrer" />

## 一个简单的例子

一个DID是一串简单的文本字符，主要包含如下三部分：

- URL方案标识符（did）
- DID方法的标识符
- DID方法特定的标识符

```xml
# 一个简单的DID例子：
did:example:123456789abcdefghi
```

上面示例的DID解析为DID文档，DID文档包含与DID相关的信息，例如在DID的控制下以密码方式对实体进行身份验证的方式，以及可用于与实体进行交互的服务。

```json
# 一个自我管理的最小DID文档
{
  "@context": "https://www.w3.org/ns/did/v1",
  "id": "did:example:123456789abcdefghi",
  "authentication": [{
    "id": "did:example:123456789abcdefghi#keys-1",
    "type": "RsaVerificationKey2018",
    "controller": "did:example:123456789abcdefghi",
    "publicKeyPem": "-----BEGIN PUBLIC KEY...END PUBLIC KEY-----\r\n"
  }],
  "service": [{
    "id":"did:example:123456789abcdefghi#vcs",
    "type": "VerifiableCredentialService",
    "serviceEndpoint": "https://example.com/vc/"
  }]
}
```

去中心化标识是大型系统的组成部分，例如可验证凭证生态系统（该系统推动了此规范的设计目标）。本节总结了此规范的主要设计目标：


目标	|描述
---|---
去中心化 |	消除了对标识符管理中集中管理机构或单点故障的要求，包括全局唯一标识符，公共验证密钥，服务端点和其他元数据的注册。
控制性	|赋予实体（无论人类还是非人类）以直接控制其数字标识符的能力，而无需依赖外部权限。
隐私性	|使实体能够控制其信息的隐私，包括对属性或其他数据的最小范围控制、选择性控制的和逐步公开的数据控制。
安全性	|为依赖方提供足够的安全性，以使其依赖DID文档获得所需的保证级别。
证明性	|与其他实体进行交互时，使DID主体能够提供加密证明。
可发现性	|使实体可以发现其他实体的DID，以了解更多有关这些实体或与这些实体进行交互的信息。
互操作性	|使用可互操作的标准，以便DID基础结构可以利用为互操作性而设计的现有工具和软件库。
可移植性	|不受系统和网络的影响，并使实体能够在支持DID和DID方法的任何系统上使用其数字标识符。
简单性	|具备简单的功能和架构，以使该技术更易于理解，实施和部署。
可扩展性	|在可能的情况下，启用可扩展性，前提是它不会极大地妨碍互操作性，可移植性或简单性。

DID和DID文档互操作性的实现将基于评估实现创建和解析符合规范的DID和DID文档的能力来进行测试，DID方法的互操作性将通过评估每个DID方法规范来确定，以至少确定：

- DID方法的名称是唯一的，并且没有被已有的不兼容的DID方法所使用；
- 支持所需的操作；
- 描述了需要描述的操作；
- 规范足够具体，详细和完整，可以独立实施；
- 规范包含描述安全和隐私注意事项的部分；

为DID和DID文档的生产者和消费者提供互操作性是为了确保DID和DID文档是一致的。

DID方法规范的细节都提供了DID方法中规范的互操作性，但是需要注意的是因为一个浏览器不需要执行所有已知的URI方案，同样的与DID兼容的软件也不需要执行所有已知的DID方法。

然而，针对特定DID方法实现的操作必须与使用该方法实现的其他方可以进行互操作。

## 专业术语

术语	|描述
---|---
Decentralized Identifier (DID)	|全局唯一标识符，不需要中央注册机构，因为它是通过分布式分类帐技术（DLT）或其他形式的分散网络进行注册和管理的
Decentralized Public Key Infrastructure (DPKI)	|基于去中心化标识符和包含可验证的公共密钥描述的身份记录（例如DID文档）的公共密钥基础结构
DID Controller	|控制DID或DID文档的一个或一组实体（DID控制器可能包括DID主题）
DID Document	|一组描述DID对象的数据，包括DID对象可以用来认证自己并证明其与DID关联的机制，例如公钥和假名生物测定；DID文档可能还包含描述对象的其他属性或声明， 这些文档是基于图形的数据结构，通常使用JSON-LD表示，但是可以使用其他兼容的基于图形的数据格式表示
DID Fragment	|DID URL中位于第一个哈希符号字符（＃）之后的部分， DID片段语法与URI片段语法相同
DID Method	|有关如何在特定的分布式分类帐或网络上实施特定DID方案的定义，包括解析和停用DID以及编写和更新DID文档的精确方法
DID Path	|DID URL的开头并包括第一个正斜杠字符（/）的部分，DID路径语法与URI路径语法相同
DID Query	|DID URL中第一个问号字符（？）之后的部分，DID查询语法与URI查询语法相同
DID Registry	|系统执行的角色是调解分散标识符的创建、验证、更新和停用， DID注册表是一种可验证的数据注册表
DID Resolver	|能够检索给定DID的DID文档的系统
DID Scheme	|去中心化标识符的形式语法
DID Subject	|由DID标识并由DID文档描述的实体
DID URL|	DID加上可选的DID路径，可选的?字符后跟DID查询，以及可选的＃字符后跟DID片段
Distributed Ledger (DLT)	|分布式账本
Extensible Data Interchange (XDI)	|语义图格式和语义数据交换协议
JSON Pointer	|定义字符串语法以标识JavaScript对象表示法（JSON）文档中的特定值
Public Key Description	|DID文档中包含的JSON对象，其中包含使用公共密钥或验证密钥所需的所有元数据
Service Endpoint	|DID对象运行服务的网络地址