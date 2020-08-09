---
title: WeIdentity 规范（三）
date: 2020-02-04 19:56:24
tags: 
    - 区块链
    - DID规范
---
<meta name="referrer" content="no-referrer" />

## Credential操作

- 创建Credential
任何实体都可以issue一个Credential。

- 验证Credential
通过这个接口，一个Entity可以对一个Credential进行验证。

- 存储/提取Credential
Credential的holder可以转移这个Credential，或者提取完整的Credential私下存储。

- 撤销 Credential
Credential的Issuer可以撤销这个Credential。

## 选择性披露

### 如何为Credential生成签名

Issuer生成Credential签名的过程：

- Claim中的每个字段计算生成一个对应的hash值。
- 将Claim中的每个字段的hash值以某种形式拼接起来形成一个字符串Claim_Hash，然后跟Credential原有的其他字段组成一个新的用于计算hash的Credential结构。
- 对这个包含Claim_Hash的Credential结构计算hash，得到Credential Hash。
- 使用Private Key对这个Credential Hash进行签名，得到签名的值Signature。

![](https://weidentity.readthedocs.io/zh_CN/latest/_images/sign-credential.png)

### 如何验证选择性披露的Credential

用户选择需要披露的字段集合（可以是一个或者几个字段，这个例子中是Field_1），需要披露的字段提供原文，其他字段提供hash值。将包含这个Claim结构的Credential披露给Verifier。下面是Verifier验证的过程：

- Verifier从Credential提取用户披露的Claim字段。
- Verifier对用户披露的字段分别计算hash（这个例子中是Field_1,计算出Field_1_hash）,然后得到一个包含所有字段hash值的Claim结构。
- 对这个Claim结构中的每个字段的hash值以某种形式拼接起来形成一个字符串Claim_Hash，然后跟Credential原有的其他字段组成一个新的用于计算hash的Credential结构。
- 对这个包含Claim_Hash的Credential结构计算hash，得到Credential Hash。
- 使用Credential的Signature和Issuer的public key进行decrypt，得到一个签名的计算值。
- 比较Credential的Signature与签名的计算值，看是否相等，确认这个Credential的合法性。

![](https://weidentity.readthedocs.io/zh_CN/latest/_images/verify-credential.png)

### Credential撤销

#### 撤销如何工作

Credential撤销机制利用了下面两点：

- 任意大于1的整数 a，如果 a 不是素数，则 a 可以表示为一系列素数的乘积，且这个表示是唯一的（不考虑顺序）。参见：算数基本定理
- 目前没有一个有效率的算法，对两个足够大的素数乘积得到的半素数（semiprime）进行整数分解。参见：整数分解

WeIdentity 会公开一个大素数的文件，每个素数会有一个 index，供所有 Issuer 使用。

Issuer 发行一个 Credential 的时候，就随机从这个大素数文件中选择一个素数，这个素数的 index 会作为这个 Credential 的属性之一。

并把以往所有发行的未撤销的 Credential 的素数相乘，得到一个大数 Accumulator（每个 Issuer 会维护自己的 Accumulator），并将这个 Accumulator 公开供所有接入方查询。

#### Issuer 如何撤销一个 Credential
用这个 Credential 对应的大素数去除 Issuer 自己的 Accumulator，将结果更新为新的 Accumulator。

#### Verifier 如何验证一个 Credential 有效（未被撤销）
用这个 Credential 对应的大素数去除 Credential 的 Issuer 公开的 Accumulator，如果能整除，则表明是 Credential 是有效的（未被撤销）。

![](https://weidentity.readthedocs.io/zh_CN/latest/_images/before-revocation.png)

![](https://weidentity.readthedocs.io/zh_CN/latest/_images/after-revocation.png)
