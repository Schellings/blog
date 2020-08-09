---
title: BCrypt对密码进行加密及密码验证
date: 2020-02-10 19:56:24
tags: Java
---
<meta name="referrer" content="no-referrer" />

## BCrypt初识

BCrypt这种加密方式是单向的哈希加密，每次加密都会有不同的密文产生，再加上不能反向解密，所以只能做匹配。

- 既然每次hash都不一样，那么如何判断加密是否正确呢？
  是这样的：在加密的时候，先随机获取salt，然后跟password进行hash。在下一次校验的时候，从hash中取出salt，salt跟password进行hash，得到的结果跟保存在在数据库的hash进行比较。
- 生成的密码中，$是分割符，无意义；2a是bcrypt加密版本号；10是cost的值（默认值）；而后的前22位是salt值；再然后的字符串就是密码的密文了
- BCrypt是单向的，而且经过salt和cost的处理，使其受rainbow攻击破解的概率大大降低，同时破解的难度也提升不少
- 加盐：如果两个人或多个人的密码相同，加密后保存会得到相同的结果。破一个就可以破一片的密码。如果名为A的用户可以查看数据库，那么他可以观察到自己的密码和别人的密码加密后的结果都是一样，那么，别人用的和自己就是同一个密码，这样，就可以利用别人的身份登录了。其实只要稍微混淆一下就能防范住了，这在加密术语中称为“加盐”。具体来说就是在原有材料（用户自定义密码）中加入其它成分（一般是用户自有且不变的因素），以此来增加系统复杂度。当这种盐和用户密码相结合后，再通过摘要处理，就能得到隐蔽性更强的摘要值。（引用：https://blog.csdn.net/Tracyhuixingfu/article/details/46653659）
- 目前，MD5和BCrypt比较流行。相对来说，BCrypt比MD5更安全，但加密更慢。


## 工具包

```yaml
<dependency>
    <groupId>org.mindrot</groupId>
    <artifactId>jbcrypt</artifactId>
    <version>0.4</version>
</dependency>
```
## demo

```java
import org.mindrot.jbcrypt.BCrypt;

public class BCryptTest {
    public static void main(String[] args) {
        String pwd = "123456";
        String encodePwd = BCrypt.hashpw(pwd, BCrypt.gensalt()); // 加密，核心代码
        System.out.println(encodePwd);
        boolean flag = BCrypt.checkpw(pwd, encodePwd); // 验证加密是否正确
        System.out.println(flag);
    }
}
```

.net中可能也会使用到该加密机制，可参照此处：[传送门](https://www.helplib.com/GitHub/article_145586)