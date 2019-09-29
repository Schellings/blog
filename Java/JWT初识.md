---
title: JWT初识
date: 2019-07-05 19:56:24
tags: 
    - 加密
---

## JWT(JSON Web Token)

JWT是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准。

### JWT是什么

一般是一个经过签名的字符串，通过 **.** 分割为三个部分，如：

```yaml
eyJhbGciOiJSUzI1NiIsImtpZCI6IjFlOWdkazcifQ.ewogImlzcyI6ICJodHRw
Oi8vc2VydmVyLmV4YW1wbGUuY29tIiwKICJzdWIiOiAiMjQ4Mjg5NzYxMDAxIiw
KICJhdWQiOiAiczZCaGRSa3F0MyIsCiAibm9uY2UiOiAibi0wUzZfV3pBMk1qIi
wKICJleHAiOiAxMzExMjgxOTcwLAogImlhdCI6IDEzMTEyODA5NzAKfQ.ggW8hZ
1EuVLuxNuuIJKX_V8a_OMXzR0EHR9R6jgdqrOOF4daGU96Sr_P6qJp6IcmD3HP9
9Obi1PRs-cwh3LO-p146waJ8IhehcwL7F09JdijmBqkvPeB2T9CJNqeGpe-gccM
g4vfKjkM8FcGvnzZUN4_KSP0aAp1tOJ1zZwgjxqGByKHiOtX7TpdQyHE5lcMiKP
XfEIQILVq0pc_E2DzL7emopWoaoZTF_m0_N0YzFC6g6EJbOEoRoSK5hoDalrcvR
YLSrQAZZKflyuVCyixEoV9GfNQC3_osjzw2PAithfubEEBLuVVk4XUVrWOLrLl0
nx7RkKU8NXNHq-rvKMzqg
```

第一部分称为**header**，第二部分称为**playload**，第三部分称为**signature**。

### JWT解析
header和playload是两个base64编码的json字符串，直接用base64解码即可，以上面的jwt为例，解码的结果如下：

```yaml

// header
{
  "alg":"RS256",
  "kid":"1e9gdk7"
}

// playload
{
 "iss": "http://server.example.com",
 "sub": "248289761001",
 "aud": "s6BhdRkqt3",
 "nonce": "n-0S6_WzA2Mj",
 "exp": 1311281970,
 "iat": 1311280970
}

```

signature是签名信息，**JWT是否有效取决于对签名信息的验证是否通过**。