---
title: WeIdentity 规范（二）
date: 2020-02-04 19:56:24
tags: 
    - 区块链
    - DID规范
---
<meta name="referrer" content="no-referrer" />

## 可验证数字凭证 (WeIdentity Credential)
现实世界中存在着各种各样用于描述实体身份、实体间关系的数据，如身份证、行驶证、存款证明、处方、毕业证、房产证、信用报告等。WeIdentity Credential提供了一整套基于W3C VC规范的解决方案，旨在对这一类数据进行标准化、电子化，生成可验证、可交换的「凭证」（Credential），支持对凭证的属性进行选择性披露，及生成链上存证（Evidence）。  
WeIdentity支持认证机构自行注册标准化凭证模板，共同丰富公众联盟链的生态。 

## Credential结构

属性	|说明
---|---
@context	|用于描述Credential的字段信息等
id	|本Credential的ID，按UUID生成
issuer	|Issuer的DID，遵从WeIdentity规范
issued	|issue日期
claim	|Claim的具体细节，数据结构由CPT定义，详见CPT介绍
claim.primeNumberIdx	|素数在素数表中的index（重要，用于权限校验）
claim.type	|Claim Protocol Type的ID，申请按序递增，例如中国内地驾照设置为CPT100
revocation	|撤销相关实现，待补充
signature	|Issuer的签名列表，是一个数组，可由holder和issuer分别打上签名
signature.type	|签名类型
signature.created	|签名的创建时间
signature.creator	|签名机构的WeIdentity DID
signature.domain	|domain
signature.nonce	|随机数
signature.signatureValue	|签名的具体value，对整个Credential结构中除去signature字段的其他字段做签名

```json
{
  "@context": "https://weidentity.webank.com/vc/v1",
  "id": "dsfewr23sdcsdfeqeddadfd",
  "type": ["Credential", "cpt100"],
  "issuer": "did:weid:1:2323e3e3dweweewew2",
  "issued": "2010-01-01T21:19:10Z",
  "claim": {
    "primeNumberIdx":"1234"
    //the other properties in this structure varied according to different CPT
  },
  "revocation": {
    "id": "did:weid:1:2323e3e3dweweewew2",
    "type": "SimpleRevocationList2017"
  },
  "signature": [{
    "type": "LinkedDataSignature2015",
    "created": "2016-06-18T21:19:10Z",
    "creator": "did:weid:1:2323e3e3dweweewew2",
    "domain": "www.diriving_card.com",
    "nonce": "598c63d6",
    "signatureValue": "BavEll0/I1zpYw8XNi1bgVg/sCneO4Jugez8RwDg/+MCRVpjOboDoe4SxxKjkC
  OvKiCHGDvc4krqi6Z1n0UfqzxGfmatCuFibcC1wpsPRdW+gGsutPTLzvueMWmFhwYmfIFpbBu95t501+r
    SLHIEuujM/+PXr9Cky6Ed+W3JT24="
  }]
}
```
## 区块链上的Credential结构

属性	|说明
---|---
id	|同上
type	|同上
issued|	同上
claimHash	|Claim结构内容的hash
revocation|	同上
signature	|同上

## Claim Protocol Type（CPT）注册机制

不同的Issuer按业务场景需要，各自定义不同类型数据结构的Claim，所有的Claim结构都需要到CPT合约注册，以保证全网唯一。所有的CPT定义文件（JSON-LD格式）可以从CPT合约下载。

属性	|说明
---|---
@context	|用于描述CPT等信息
id	|CPT的编号，要保证唯一
version	|CPT的版本号
publisher	|CPT的发布者的WeIdentity DID
signature	|CPT的发布者的签名
claim	|Claim的格式定义
created	|创建时间
updated	|更新时间
description	|CPT的描述信息

```json
"CPT": {
  "@context" : "https://weidentity.webank.com/cpt100/v1",
  "version" : "v1",
  "id" : "CPT100",
  "publisher" : "did:weid:1:2323e3e3dweweewew2",
  "signature" : "BavEll0/I1zpYw8XNi1bgVg/sCneO4Jugez8RwDg/+MCRVpjOboDoe4SxxKjkC
  OvKiCHGDvc4krqi6Z1n0UfqzxGfmatCuFibcC1wpsPRdW+gGsutPTLzvueMWmFhwYmfIFpbBu95t501+r
    SLHIEuujM/+PXr9Cky6Ed+W3JT24=",
  "claim" : "",
  "address" : "重庆",
  "class" : "C1",
  "created" : "2010-06-20T21:19:10Z",
  "updated" : "2016-06-20T21:19:10Z",
  "description" : "中国内地驾照"
}
```

### CPT和Credential的关系

![](https://weidentity.readthedocs.io/zh_CN/latest/_images/cpt-er.png)

其中CPT为模板类，定义了Claim包含的数据字段及各字段属性要求。Claim为CPT的实例。Issuer将Claim进行签名，即可生成Credential。


下面介绍Credential的使用场景。
