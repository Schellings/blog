---
title: word文档解析为html工具类
date: 2018-11-20 19:56:24
tags: 
    - Java
---
## 转换方法

使用poi文件解析包

## POM文件

```yaml
        <!--word文件解析相关-->
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi</artifactId>
			<version>3.14</version>
		</dependency>
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi-scratchpad</artifactId>
			<version>3.14</version>
		</dependency>
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi-ooxml</artifactId>
			<version>3.14</version>
		</dependency>
		<dependency>
			<groupId>fr.opensagres.xdocreport</groupId>
			<artifactId>xdocreport</artifactId>
			<version>1.0.6</version>
		</dependency>
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi-ooxml-schemas</artifactId>
			<version>3.14</version>
		</dependency>
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>ooxml-schemas</artifactId>
			<version>1.3</version>
		</dependency>
```

## 工具类

```java
/**
 * @Classname DocToHtmlUtil
 * @Description TODO
 * @Date 2018/11/19 11:29
 * @Created by Schelling
 */

import org.apache.poi.xwpf.converter.core.BasicURIResolver;
import org.apache.poi.xwpf.converter.core.FileImageExtractor;
import org.apache.poi.xwpf.converter.xhtml.XHTMLConverter;
import org.apache.poi.xwpf.converter.xhtml.XHTMLOptions;
import org.apache.poi.xwpf.usermodel.XWPFDocument;

import java.io.*;
public class DocToHtmlUtil {
    public static String wordToHtml(String filePath) throws Exception{
        if(filePath.endsWith(".doc")){
            String content=docToHtml(filePath);
            return content;
        }
        if(filePath.endsWith(".docx")){
            String content=docxToHtml(filePath);
            return content;
        }
        return null;
    }

    private static String docToHtml(String sourceFileName) throws Exception{
        String htmlPath=sourceFileName.substring(0,sourceFileName.indexOf("."))+".html";
        String imagePathStr = new File(sourceFileName).getParent();
        File htmlFile=new File(htmlPath);
        if(! htmlFile.exists()){
            htmlFile.createNewFile();
        }
        File imagePath=new File(imagePathStr);
        if(!imagePath.exists()){
            imagePath.mkdirs();
        }
        ByteArrayOutputStream byteArrayOutputStream = null;
        try {
            XWPFDocument document = new XWPFDocument(new FileInputStream(sourceFileName));
            XHTMLOptions options = XHTMLOptions.create();
            // 存放图片的文件夹
            options.setExtractor(new FileImageExtractor(new File(imagePathStr)));
            // html中图片的路径
            options.URIResolver(new BasicURIResolver("image"));
            byteArrayOutputStream = new ByteArrayOutputStream();
            XHTMLConverter xhtmlConverter = (XHTMLConverter) XHTMLConverter.getInstance();
            xhtmlConverter.convert(document, byteArrayOutputStream, options);

            String content =new String(byteArrayOutputStream.toByteArray());
            String htmlContent1=content.replaceAll("&ldquo;","\"").replaceAll("&rdquo;","\"").replaceAll("&mdash;","-");
            return htmlContent1;
        } finally {
            if (byteArrayOutputStream != null) {
                byteArrayOutputStream.close();
            }
        }
    }

    //docx转html
    //生成html文件
    //输出html标签和内容
    public static String docxToHtml(String sourceFileName) throws Exception {
        String basePathStr = sourceFileName.substring(0,sourceFileName.lastIndexOf("."));
        File basePath = new File(basePathStr);
        if(!basePath.exists()){
            basePath.mkdirs();
        }

        String htmlPath=basePathStr+basePathStr.substring(basePathStr.lastIndexOf("/"))+".html";
        String imagePathStr=basePathStr;


        XWPFDocument document = new XWPFDocument(new FileInputStream(sourceFileName));
        XHTMLOptions options = XHTMLOptions.create().indent(4);
        options.setExtractor(new FileImageExtractor(new File(imagePathStr)));
        File outFile = new File(htmlPath);
        outFile.getParentFile().mkdirs();
        OutputStream out = new FileOutputStream(outFile);
        XHTMLConverter.getInstance().convert(document,out, options);
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        XHTMLConverter.getInstance().convert(document, baos, options);
        baos.close();
        String content =new String(baos.toByteArray());
        //替换无法识别的转义字符
        String htmlContent1=content.replaceAll("&ldquo;","\"").replaceAll("&rdquo;","\"").replaceAll("&mdash;","-");
        return htmlContent1;
    }
}
```
