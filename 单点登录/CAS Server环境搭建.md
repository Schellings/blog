---
title: CAS Server环境搭建
date: 2018-08-25 19:56:24
tags: 
    - 单点登录
---
<meta name="referrer" content="no-referrer" />

##  前言

本次环境搭建使用的CAS Server为3.5.2

## 下载

[CAS Server Webapp](https://repo1.maven.org/maven2/org/jasig/cas/cas-server-webapp/3.5.2/cas-server-webapp-3.5.2.war)

## 部署

（1）下载完毕后，将cas-server-webapp-3.5.2.war重命名为cas.war放入tomcat/webapps下

（2）修改cas server端口

如果tomcat的端口不是用的8080的话，还需要在cas工程中修改一下配置文件cas.properties。
```yaml
server.name=http://localhost:8080
```

（3）启动tomcat，访问 http://localhost:8080/cas 即可看到登录页，默认使用相同的用户名与密码即可登录，如：admin/admin

（4）关闭https认证
- deployerConfigContext.xml 增加参数p:requireSecure="false"

文件路径：cas/WEB-INF/deployerConfigContext.xml

```yaml
<bean class="org.jasig.cas.authentication.handler.support.HttpBasedServiceCredentialsAuthenticationHandler"
					p:httpClient-ref="httpClient"
					p:requireSecure="false"/>
```
- ticketGrantingTicketCookieGenerator.xml修改p:cookieSecure="false"

文件路径： cas/WEB-INF/spring-configuration/ticketGrantingTicketCookieGenerator.xml

```yaml
<bean id="ticketGrantingTicketCookieGenerator" class="org.jasig.cas.web.support.CookieRetrievingCookieGenerator"
		p:cookieSecure="false"
		p:cookieMaxAge="3600"
		p:cookieName="CASTGC"
		p:cookiePath="/cas" />
```
- warnCookieGenerator.xml修改p:cookieSecure="false"

文件路径： cas/WEB-INF/spring-configuration/warnCookieGenerator.xml

```yaml
<bean id="warnCookieGenerator" class="org.jasig.cas.web.support.CookieRetrievingCookieGenerator"
		p:cookieSecure="false"
		p:cookieMaxAge="3600"
		p:cookieName="CASPRIVACY"
		p:cookiePath="/cas" />
```

>注：如果不处理步骤4，cas登录成功后直接刷新又会回到登录页。

（5）重启tomcat