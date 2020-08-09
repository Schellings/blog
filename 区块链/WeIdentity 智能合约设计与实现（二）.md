---
title: WeIdentity 智能合约设计与实现（二）
date: 2020-02-05 19:56:24
tags: 
    - 区块链
    - 智能合约
    - DID规范
---
<meta name="referrer" content="no-referrer" />

## WeIdentity Authority智能合约
### 概述
Authority智能合约的主要任务是联盟链的权限管理。在WeIdentity的业务场景中，存在以下挑战：

不同的DID实体拥有不同的权限。

例如，存在Authority Issuer这一角色用来描述现实世界中的「权威凭证发行者」，它们能够发行低段位授权CPT，权限高于一般的DID；  
更进一步地，在Authority Issuer之上存在着委员会（Committee），它们的权限更高，包括了对Authority Issuer的治理等内容。因此，WeIdentity需要设计合理的「角色—操作」二元权限控制。

权限管理的业务逻辑会随着业务迭代而不断更新。在真实业务场景中，随着业务变化，权限管理逻辑也可能随之改变；同时，不同的业务方可能会有定制化权限管理的需求。  
因此，WeIdentity需要进行合理的分层设计，将**数据和行为逻辑分离**，在升级的情况下就只需对行为逻辑部分进行升级，数据存储保持不变，尽可能地降低更新成本。

当前，业内已经有了一些对权限进行操作和维护的开源解决方案，如ds-auth和OpenZepplin的Role智能合约；但它们的权限管理逻辑可扩展性较差且不支持合约分层更新。下文将介绍WeIdentity的Authority智能合约实现。

### 架构

#### 角色与权限
当前的WeIdentity角色设计了四种角色：

- 一般DID。一般的实体（人或物），由WeIdentity的分布式多中心的ID注册机制生成，没有特定权限。
- Authority Issuer。授权机构，具有发行低段位授权CPT的权限。
- Committee Member。机构委员会成员。具有管理Authority Issuer成员资格的权限。
- Administrator。系统管理员。具有管理Committee Member及Authority Issuer成员资格的权限，未来还包括修改合约地址的权限。

每个角色具体的权限表如下：

操作|	一般DID|	Authority Issuer|	Committee Member|	Administrator
---|---|---|---|---
增删改Administrator|	N|	N|	N	|Y
增删改Committee Member|	N	|N|	N|	Y
增删改Authority Issuer|	N|	N	|Y|	Y
发行授权CPT|	N	|Y	|Y	|Y

#### 合约分层

WeIdentity采用分层设计模式，即将合约分为逻辑合约、数据合约、及权限合约。

- 逻辑合约：它专注于数据的逻辑处理和对外提供接口，通过访问数据合约获得数据，对数据做逻辑处理，写回数据合约。一般情况下，控制器合约不需要存储任何数据，它完全依赖外部的输入来决定对数据合约的访问。
- 数据合约：它专注于数据结构的定义、数据内容的存储和数据读写的直接接口。
- 权限合约：它专注于判断访问者的角色，并基于判断结果确定不同操作的权限。

上述架构图如下：

![](https://weidentity.readthedocs.io/zh_CN/latest/_images/authority-contract-arch.png)


#### 权限与安全管理

当前的WeIdentity权限管理的挑战是：

- 合约在链上部署之后，攻击者可能会绕过SDK直接以DApp的形式访问合约。因此合约层面必须要有自完善的权限处理逻辑，不能依赖SDK。
- 数据合约是公开的，因此数据合约的操作也需要进行权限管理。

WeIdentity的权限管理依赖于一个独立的RoleManager权限管理器合约，它承担了合约所有的权限检查逻辑。WeIdentity的权限粒度是基于角色和操作的二元组，这也是当前大多数智能合约权限控制的通用做法。它的设计要点包括：

- 将角色和操作权限分别存储。
- 设计一个权限检查函数checkPermission()供外部调用，输入参数为「地址，操作」的二元组。
- 对角色和权限分别设计增删改函数供外部调用。
- 所有WeIdentity的数据合约里需要进行权限检查的操作，都通过外部合约函数调用的方式，调用checkPermission()。
- 所有WeIdentity依赖权限管理器的合约，需要有更新权限管理器地址的能力。

WeIdentity的权限管理有以下特性：

- 优秀的可扩展性。WeIdentity的权限控制合约使用外部调用而非继承（如ds-auth和OpenZepplin的Role智能合约实现角色管理方式）方式实现。
```json
在权限控制合约升级的场景中，外部调用方案只需简单地将权限管理器合约地址更新即可，极大地提升了灵活度。
```
- 使用tx.origin而非msg.sender进行调用源追踪。这是因为用户的权限和自己的DID地址唯一绑定。因此所有权限的验证必须要以最原始用户地址作为判断标准，不能单纯地依赖msg.sender。


