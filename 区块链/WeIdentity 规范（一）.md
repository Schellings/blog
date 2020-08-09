---
title: WeIdentity 规范（一）
date: 2020-02-03 19:56:24
tags: 
    - 区块链
    - DID规范
---
<meta name="referrer" content="no-referrer" />

## 什么是 WeIdentity？
WeIdentity是一套分布式多中心的技术解决方案，可承载实体对象（人或者物）的现实身份与链上身份的可信映射、以及实现实体对象之间安全的访问授权与数据交换。WeIdentity由微众银行自主研发并完全[开源]()，秉承公众联盟链整合资源、交换价值、服务公众的理念，致力于成为链接多个垂直行业领域的分布式商业基础设施，促进泛行业、跨机构、跨地域间的身份认证和数据合作。

## 模块介绍
WeIdentity目前主要包含两大模块：WeIdentity DID以及WeIdentity Credential。

### 分布式身份标识 (WeIdentity DID)
传统方式中，用户的注册和身份管理完全依赖于单一中心的注册机构；随着分布式账本技术（例如区块链）的出现，分布式多中心的身份注册、标识和管理成为可能。  
WeIdentity DID模块在FISCO-BCOS区块链底层平台上实现了一套符合W3C DID规范的分布式多中心的身份标识协议，使实体（人或物）的现实身份实现了链上的身份标识；同时，WeIdentity DID给与Entity（人或者物）直接拥有和控制自己身份ID的能力。

### 可验证数字凭证 (WeIdentity Credential)
现实世界中存在着各种各样用于描述实体身份、实体间关系的数据，如身份证、行驶证、存款证明、处方、毕业证、房产证、信用报告等。WeIdentity Credential提供了一整套基于W3C VC规范的解决方案，旨在对这一类数据进行标准化、电子化，生成可验证、可交换的「凭证」（Credential），支持对凭证的属性进行选择性披露，及生成链上存证（Evidence）。  
WeIdentity支持认证机构自行注册标准化凭证模板，共同丰富公众联盟链的生态。 
 
 
 ## WeIdentity参考场景
 
 ![](https://weidentity.readthedocs.io/zh_CN/latest/_images/roles-relation.png)
 
 在WeIdentity生态中，存在着上图所示的几类角色，不同角色的权责和角色之间的关系如下表所示：
 
 角色|	说明
 ---|---
 User (Entity)|	用户（实体）。会在链上注册属于自己的WeIdentity DID，从Issuer处申请Credential，并授权转发或直接出示给Verifier来使用之。
 Issuer	|Credential的发行者。会验证实体对WeIdentity DID的所有权，其次发行实体相关的Credential。
 Verifier	|Credential的使用者。会验证实体对WeIdentity DID的所有权，其次在链上验证Credential的真实性，以便处理相关业务。
 User Agent / Credential Repository	|用户（实体）在此生成WeIdentity DID。为了便于使用，实体也可将自己的私钥、持有的Credential托管于此。
 
 在实际业务里，WeIdentity可以被广泛运用在「实体身份标识」及「可信数据交换」场景中。首先，通过User Agent为不同的实体生成独立唯一的DID；其次，Issuer验证实体身份及DID所有权，为实体发行各种各样的电子化Credential。
 
 当实体需要办理业务的时候，可以直接将Credential出示给Verifier，也可以通过在链上进行主动授权 + 授权存证上链的方式，由之前授权的凭证存储机构转发给Verifier。
 
 ## WeIdentity DID与WeIdentity Credential的关系
 
 ![](https://weidentity.readthedocs.io/zh_CN/latest/_images/weidentity-er.png)
 
 从图中可见，WeIdentity DID与WeIdentity Credential的关系并非单纯的一对多：从设计目标上看，WeIdentity DID用来描述实体（人或物），WeIdentity Credential用来描述实体的身份、属性和实体间关系。  
 因此，一个WeIdentity DID可以持有多个WeIdentity Credential；而一个WeIdentity Credential则会包含至少一个所描述的WeIdentity DID，可能会有多个。  
 最后，每个WeIdentity DID都有一个WeIdentity Document，用来存储此DID的认证方式（如公钥、私钥套件）等信息，与WeIdentity Credential无关。
 
 ## 总体流程
 
 ![](https://weidentity.readthedocs.io/zh_CN/latest/_images/overall-flow@2x.png)
 
 一般来说，WeIdentity解决方案的基本流程如下：
 
 - 用户根据业务需求，选择是否需要进行[KYC认证](https://www.jianshu.com/p/2d52239ada45)。
 - 用户生成WeIdentity DID。
 - 用户向相关业务方申请Credential。
 - 相关业务方扮演Issuer的角色，发行Credential交给用户。
 - 用户成为了Credential的Holder。
 - 用户出示Credential，以完成业务需求。
 - 相关业务方扮演Verifier的角色，验证Credential有效性。
 
 ## 使用场景
 
 ![](https://weidentity.readthedocs.io/zh_CN/latest/_images/scenario.png)
 
 上图展示了五个WeIdentity生态下Credential在不同角色间流转的场景：
 
 身份证明机构作为Issuer向用户发行「实名认证Credential」，政府机构作为Verifier在办理公共事务时对其进行验证。  
 学校作为Issuer向用户发行「学历证明Credential」，公司作为Verifier在对候选人进行背景调查时对其进行验证。    
 出入境机构作为Issuer向用户发行「签证Credential」，海关作为Verifier在出入境时对其进行验证。  
 公司作为Issuer向用户发行「收入证明Credential」，银行作为Verifier在发放贷款时对其进行验证。  
 商户作为Issuer向用户发行「优惠券Credential」，商户自己作为Verifier在对优惠券核销时对其进行验证。  
 
 接下来介绍WeIdentity在VC标准上的实现：WeIdentity Credential。
 