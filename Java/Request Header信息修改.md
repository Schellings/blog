---
title: Request Header信息修改
date: 2018-12-21 19:56:24
tags: 
    - Java
---
<meta name="referrer" content="no-referrer" />

## 前言

在Java中服务端Request的Header信息在请求时进行构建，在服务端获取到的HttpServletRequest request对象只能获取header信息，而无法修改，下面根据反射机制，修改其Header信息，并封装相关工具类。

## 工具类

```java
import org.apache.tomcat.util.buf.MessageBytes;
import org.apache.tomcat.util.http.MimeHeaders;

import javax.servlet.http.HttpServletRequest;
import java.lang.reflect.Field;
import java.util.Objects;
/**
 * @Description 修改header信息，key-value键值对加入到header中
 * @auther Schelling
 * @create 2019-10-17 12:26
 */
public class ReflectRequestHeaderHelper {
    public static void setHeader(HttpServletRequest request,String key,String value){
        Class<? extends HttpServletRequest> requestClass = request.getClass();
        try {
            // 注意断点查看内部实现中是否含义该字段，根据实际进行修改
            Field request1 = requestClass.getDeclaredField("request");
            request1.setAccessible(true);
            Object o1 = request1.get(request);
            Field request2 = o1.getClass().getDeclaredField("request");
            request2.setAccessible(true);
            Object o2 = request2.get(o1);
            Field coyoteRequest = o2.getClass().getDeclaredField("coyoteRequest");
            coyoteRequest.setAccessible(true);
            Object o3 = coyoteRequest.get(o2);
            Field headers = o3.getClass().getDeclaredField("headers");
            headers.setAccessible(true);
            MimeHeaders o4 = (MimeHeaders)headers.get(o3);
            MessageBytes oldValue = o4.getValue(key);
            if(Objects.isNull(oldValue)){
                o4.addValue(key).setString(value);
            }else{
                o4.setValue(key).setString(value);
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```