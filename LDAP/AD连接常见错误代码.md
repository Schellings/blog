---
title: AD连接常见错误代码
date: 2020-03-20 19:56:24
tags: 
    - LDAP
---
<meta name="referrer" content="no-referrer" />
## 背景
```yaml
"The exception is [ LDAP: error code 49 - 80090308: LdapErr: DSID-0Cxxxxxx, comment: AcceptSecurityContext error, data xxx, vece ]."
```
然而存在这多种情况指示LDAP功能存在异常。下面根据Microsoft Activative Directory给出些一般参考。

`AD指令错误代码是一段在"data"之后并在"vece"或者"v893"这样文字之前的一段字符。`

实际上这些错误代码伴随着绑定过程而返回。

## 常见错误信息

错误码 | 含义
---|---
525|	用户不存在
52e	|密码或凭证无效
530	|此时不允许登录
531	|在此工作站上不允许登录
532	|密码过期
533	|账户禁用
701	|账户过期
773	|用户必须重置密码
775	|用户账户锁定

一般Active Directory LDAP 绑定错误: 

`80090308: LdapErr: DSID-0C09030B, comment: AcceptSecurityContext error, data 525, v893`
十六进制: 0x525 - 用户不存在 
十进制: 1317 - ERROR_NO_SUCH_USER (指定的账户不存在.) 
注释: 当用户名无效时返回

`80090308: LdapErr: DSID-0C09030B, comment: AcceptSecurityContext error, data 52e, v893 `
十六进制: 0x52e - 无效的凭证
十进制: 1326 - ERROR_LOGON_FAILURE (登录失败，未知的用户名或者密码错误.) 
注释: 当用户名有效但是密码或者凭证无效的时候返回。


`80090308: LdapErr: DSID-0C09030B, comment: AcceptSecurityContext error, data 530, v893 `
十六进制: 0x530 - 此时禁止登录
十进制: 1328 - ERROR_INVALID_LOGON_HOURS (登录失败，登录时间违规.) 
注释: 仅当输入了正确的用户名和密码或凭证时才返回此值。说明用户被禁止登录了

`80090308: LdapErr: DSID-0C09030B, comment: AcceptSecurityContext error, data 531, v893 `
十六进制: 0x531 - 此用户禁止在当前工作站登录 
十进制: 1329 - ERROR_INVALID_WORKSTATION (登录失败，在此计算机上该用户不允许登录.) 
LDAP[userWorkstations: <multivalued list of workstation names>] 
注释: 当输入了正确的用户名和密码或凭证时才返回此值。

`80090308: LdapErr: DSID-0C09030B, comment: AcceptSecurityContext error, data 532, v893 `
十六进制: 0x532 - 密码过期
十进制: 1330 - ERROR_PASSWORD_EXPIRED (登录失败，指定的账户密码过期.) 
LDAP[userAccountControl: <bitmask=0x00800000>] - PASSWORDEXPIRED 
注释: 当输入了正确的用户名和密码或凭证是猜返回此值。

`80090308: LdapErr: DSID-0C09030B, comment: AcceptSecurityContext error, data 533, v893 `
十六进制: 0x533 - 账户被禁用
十进制: 1331 - ERROR_ACCOUNT_DISABLED (登录失败，账户当前被禁用了.) 
LDAP[userAccountControl: <bitmask=0x00000002>] - ACCOUNTDISABLE 
注释: 当输入了正确的用户名和密码或凭证是猜返回此值。


`80090308: LdapErr: DSID-0C09030B, comment: AcceptSecurityContext error, data 701, v893 `
十六进制: 0x701 - 账户已过期
十进制: 1793 - ERROR_ACCOUNT_EXPIRED (用户账户已过期.) 
LDAP[accountExpires: <value of -1, 0, or extemely large value indicates account will not expire>] - ACCOUNTEXPIRED 
注释: 当输入了正确的用户名和密码或凭证是猜返回此值。


`80090308: LdapErr: DSID-0C09030B, comment: AcceptSecurityContext error, data 773, v893 `
十六进制: 0x773 - 账户密码必须被重置
十进制: 1907 - ERROR_PASSWORD_MUST_CHANGE (用户密码在第一次登录之前必须修改.) 
LDAP[pwdLastSet: <value of 0 indicates admin-required password change>] - MUST_CHANGE_PASSWD 
注释: 当输入了正确的用户名和密码或凭证是猜返回此值。


`80090308: LdapErr: DSID-0C09030B, comment: AcceptSecurityContext error, data 775, v893 `
十六进制: 0x775 - 账户被锁定
十进制: 1909 - ERROR_ACCOUNT_LOCKED_OUT (账户当前已被锁定，不允许登录The referenced account is currently locked out and may not be logged on to.) 
LDAP[userAccountControl: <bitmask=0x00000010>] - LOCKOUT 
注释: 即便是输入了错误的密码也可能返回此值
