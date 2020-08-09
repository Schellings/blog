---
title: Decentralized Identifiers (DIDs)简介
date: 2019-12-25 19:56:24
tags: 
    - 区块链
    - DID规范
---
<meta name="referrer" content="no-referrer" />

##  背景与现状

### 数字身份认证背景

```xml
中心化身份 => 联盟身份 => 中心化身份（DID）
```

#### 中心化身份

一开始的数字认证始是中心化的，比如ICANN管理的域名与IP地址分配，以及PKI（Public Key Infrastructure）系统中的CA（Certificate Authority）证书机构管理的数字证书。

中心化身份系统的本质就是，中央集权化的权威机构掌握着身份数据，因为围绕数据进行的认证、授权等也都由中心化的机构来决定。身份不是由用户自己控制的。

而且不同的中心化网站（比如淘宝、知乎、豆瓣等等）上有一套自己的身份系统，所以都需要你重新注册一个账户。而不同网站自己用的身份系统（及账户对应的数据）之间是**不互通**的。

#### 联盟身份

为了解决这个问题，不同的网站自己联合起来推出了联盟身份（这个概念是首先由微软在1999年提出的）。在联盟身份体系下，用户的在线身份有了一定的可移植性。

如今的不少网站注册都可以支持第三方登录，比如微信、QQ、新浪微博等。

#### 中心化身份（DID）

在联盟身份提出后，身份系统就开始走向去中心化了。期间也有很多去中心化的标准、方案出现，比如[OpenID](https://openid.net/)、[oauth2](https://tools.ietf.org/html/rfc6749)。其实就算是一些网站支持的微信、QQ第三方登录，其用户体验也不是很好，而且往往还是需要你用手机号 + 验证码进行注册的。

综上所述，中心化身份主要的问题就是两个:

- 一是个人并不是真正意义上拥有自己的身份
- 二是身份无法互通。

### Decentralized IDentity（DID）现状

DID可以说是区块链领域一个偏冷门的方向。目前只有很少的团队在研究DID，开发的项目也不多，屈指可数。DID的热度和[扩容](https://www.jianshu.com/p/e02ddc0d6599)、[跨链](https://blog.csdn.net/niyuelin1990/article/details/86607309)这些热门概念是无法相比的。但是其实它看上去有不小的价值的，微软布局DID或许就是从侧面说明了这点。

#### 标准

目前（2019年）已经提出的标准主要有：

- W3C的DID标准：[W3C Decentralized Identifiers](https://w3c-ccg.github.io/did-spec/)
- DIF（Decentralized Identity Foundation）的DID Auth：[DIF官网](https://links.jianshu.com/go?to=https%3A%2F%2Fidentity.foundation%2F%23about)

#### 项目

目前已经有的比较知名的DID项目有：MicrosoftDID、Sovrin、uPort、Evernym、Civic、ShoCard。

项目名称	|大致内容
---|---
MicrosoftDID |	微软的DID
Sovrin	|位于HyperLedger
uPort	|位于ETH
Evernym	|用于交易
Civic	|使生物识别的多因素身份认证、移动身份平台
ShoCard	|移动身份平台、保护隐私
[WeIdentity](https://weidentity.readthedocs.io/zh_CN/latest/README.html)| 微众银行开放的分布式多中心的技术解决方案



## W3C DID 标准

```xml
去中心化身份标识(Decentralized Identifier，DID)是一种新类型的标识符，具有全局唯一性、高可用性可解析性和加密可验证性。DIDs通常与加密材料(如公钥)和服务端点相关联，以建立安全的通信信道。
DIDs对于任何受益于自管理、加密可验证的标识符(如个人标识符、组织标识符和物联网场景标识符)的应用程序都很有用。例如，当前W3C可验证凭据的商业部署大量使用DIDs来标识人员、组织和事物，
并实现许多安全和隐私保护保证。
——W3C 文档
```

基础层：DID规范

- DID标识符（Identifier）
- DID文档（Document）

应用层：可验证声明
- 可验证声明（Verifiable Claims 或 Verifiable Credentials，本文接下去都简称VC）

### DID 规范

DID标识符，是一个全局唯一的表示你身份的东西，就像你的身份证号码一样。其形式大致如下：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93M2MtY2NnLmdpdGh1Yi5pby9kaWQtcHJpbWVyL2RpZC1wcmltZXItZGlhZ3JhbXMvZGlkLWZvcm1hdC5wbmc)

DID示例：did:eth:123456789abcdefg

DID标识符不容易记忆。根据[Zooko三角形理论](https://en.wikipedia.org/wiki/Zooko%27s_triangle)，没有任何标识符能够同时实现易记忆、安全、去中心化。在这里，W3C的DID取了后两者。


DID文档是一个[JSON-LD Object](https://www.w3.org/TR/2014/REC-json-ld-20140116/)，包括6个部分（都是optional的）：

- DID标识符。
- 一个加密材料的集合。比如公钥。
- 一个加密协议的集合。
- 一个服务端点的集合。
- 时间戳。
- 一个可选的JSON-LD签名。用来证明这个DID文档是合法的。

文档内容示例：

```json
{
  "@context": "https://w3id.org/did/v1",
  "id": "did:example:123456789abcdefghi",
  "authentication": [{
    // used to authenticate as did:...fghi
    "id": "did:example:123456789abcdefghi#keys-1",
    "type": "RsaVerificationKey2018",
    "controller": "did:example:123456789abcdefghi",
    "publicKeyPem": "-----BEGIN PUBLIC KEY...END PUBLIC KEY-----\r\n"
  }],
  "service": [{
    // used to retrieve Verifiable Credentials associated with the DID
    "id":"did:example:123456789abcdefghi#vcs",
    "type": "VerifiableCredentialService",
    "serviceEndpoint": "https://example.com/vc/"
  }]
}
```

这里需要注意的是，DID文档中没有任何和你个人真实信息相关的内容，比如你的真实姓名、地址、手机号等。

因此光靠DID规范是无法验证一个人的身份的，必须要靠DID应用层中的VC（可验证声明 Verifiable Claims 或 Verifiable Credentials）。


```xml
注意：
区块链分布式账本中本质上就是一个分布式数据库，在这个数据库中，DID标识符是键，而DID文档是值。
```

### 可验证声明

W3C认为前面的DID规范是基础，而把可验证声明视作是next higher layer，并认为这一层才是**建立DID整个体系的价值所在**。

因为在这个应用层中，DID既可以用来标识个体的身份、也可以用来标识组织的身份，甚至标识物品的身份（言外之意是不仅可以改变当前的互联网，还可以改变物联网？）。

接下去我将可验证声明简称之VC。VC有点类似于数字签名，要是实现数字签名，需要有PKI体系。这里要实现VC也是一样，需要用一套系统来支持它。

在VC的这套系统中，有以下几种参与者（列出了其功能）：

- 发行者（Issuer）：拥有用户数据并能开具VC的实体，如政府、银行、大学等机构和组织。
- 验证者（Inspector-Verifier，IV）：接受VC并进行验证，由此可以提供给出示VC者某种类型的服务。
- 持有者（Holder）：向Issuer请求、收到、持有VC的实体。向IV出示VC。开具的VC可以放在VC钱包里，方便以后再次使用。
- 标识符注册机构（Identifier Registry）：维护DIDs的数据库，如某条区块链、分布式账本（差不多就是前面提到的DID里的example字段）。

之所以需有Identifier Registry，是因为IV要验证VC，也要验证用户。验证VC用VC和发VC的Issuer，验证用户用DID和存DID的数据库。

因为DID对应的DID文档里没有用户的真实信息，所以当用户进行某个操作时，网站需要用户出示证明。

比如，要求你证明“我XXX年龄已经大于18周岁”。这个时候你就需要Issuer帮你发出（并签名）这样一个VC给网站，网站做作为Inspector就可以进行验证。验证之后你可以进行操作了。

这里有一点要注意，那就是Issuer只需要给出你是超过18岁的VC，而不需要给出你的生日是多少的的VC，前者泄露你更少的信息。最理想的VC应该是一个回答是否的回复，而不是回答多少和什么的回复。这样能泄露最少的信息给IV。

VC的格式也是JSON的。示例如下：

```json
{
  // set the context, which establishes the special terms we will be using
  // such as 'issuer' and 'alumniOf'.
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://www.w3.org/2018/credentials/examples/v1"
  ],
  // specify the identifier for the credential
  "id": "http://example.edu/credentials/1872",
  // the credential types, which declare what data to expect in the credential
  "type": ["VerifiableCredential", "AlumniCredential"],
  // the entity that issued the credential
  "issuer": "https://example.edu/issuers/565049",
  // when the credential was issued
  "issuanceDate": "2010-01-01T19:73:24Z",
  // claims about the subjects of the credential
  "credentialSubject": {
    // identifier for the only subject of the credential
    "id": "did:example:ebfeb1f712ebc6f1c276e12ec21",
    // assertion about the only subject of the credential
    "alumniOf": {
      "id": "did:example:c276e12ec21ebfeb1f712ebc6f1",
      "name": [{
        "value": "Example University",
        "lang": "en"
      }, {
        "value": "Exemple d'Université",
        "lang": "fr"
      }]
    }
  },
  // digital proof that makes the credential tamper-evident
  // see the NOTE at end of this section for more detail
  "proof": {
    // the cryptographic signature suite that was used to generate the signature
    "type": "RsaSignature2018",
    // the date the signature was created
    "created": "2017-06-18T21:19:10Z",
    // purpose of this proof
    "proofPurpose": "assertionMethod",
    // the identifier of the public key that can verify the signature
    "verificationMethod": "https://example.edu/issuers/keys/1",
    // the digital signature value
    "jws": "eyJhbGciOiJSUzI1NiIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..TCYt5X
      sITJX1CxPCT8yAV-TVkIEq_PbChOMqsLfRoPsnsgw5WEuts01mq-pQy7UJiN5mgRxD-WUc
      X16dUEMGlv50aqzpqh4Qktb3rk-BuQy72IFLOqV0G_zS245-kronKb78cPN25DGlcTwLtj
      PAYuNzVBAh4vGHSrQyHUdBBPM"
  }
}
```

因为VC中是没有Issuer的公钥的（也不应该有，因为就算有了，IV还是得亲自验证公钥是否是真的）。这里VC的id是一个URI，而VC中的Issuer字段也是一个URI。而Issuer也可能是使用DID来作为其身份的。

因此通过VC中的Issuer字段——URI地址得到其DID，然后从DID对应的DID文档里就可以得到其公钥了。用公钥验证对VC的签名就能验证VC是否Issuer发的。

当然IV验证用户的方法也是如此：用Holder（即用户）的DID对应的DID文档里的公钥来验证其数字签名的合法性。


