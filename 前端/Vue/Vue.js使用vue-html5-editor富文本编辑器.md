---
title: Vue.js使用vue-html5-editor富文本编辑器
date: 2017-07-28 19:56:24
tags: 
    - Vue
---
<meta name="referrer" content="no-referrer" />

## 基础环境安装配置
```yaml


（1）安装node.js
（2）安装cnpm命令
（3）安装vue、vue-cli
（4）安装vue-html-editor、font-awesome

    npm install vue-html5-editor --save-dev
    npm install less less-loader css-loader style-loader file-loader font-awesome --save

（5）创建vue项目
（6）相应编辑器打开项目
（7）run项目 cnpm install & npm run dev
```

## 组件使用

（1）新建common文件用于存放富文本组件配置

（2）新建initHTMLEditor.js

复制以下内容：
```javascript
import Vue from 'vue'
import VueHtml5Editor from 'vue-html5-editor'
export default function () {
  let opt = {
    // 全局组件名称，使用new VueHtml5Editor(options)时该选项无效
    name: "vue-html5-editor",
    // 是否显示模块名称，开启的话会在工具栏的图标后台直接显示名称
    showModuleName: true,
    // 自定义各个图标的class，默认使用的是font-awesome提供的图标
    icons: {
      text: "fa fa-pencil",
      color: "fa fa-paint-brush",
      font: "fa fa-font",
      align: "fa fa-align-justify",
      list: "fa fa-list",
      link: "fa fa-chain",
      unlink: "fa fa-chain-broken",
      tabulation: "fa fa-table",
      image: "fa fa-file-image-o",
      hr: "fa fa-minus",
      eraser: "fa fa-eraser",
      undo: "fa-undo fa",
      "full-screen": "fa fa-arrows-alt",
      info: "fa fa-info",
    },
    // 配置图片模块
    image: {
      // 文件最大体积，单位字节  
      sizeLimit: 512 * 1024 * 10,
      // 上传参数,默认把图片转为base64而不上传
      // upload config,default null and convert image to base64
      upload: {
        url:null,
        headers:{},
        params: {},
        fieldName:{}
      },
      // 压缩参数,默认使用localResizeIMG进行压缩,设置为null禁止压缩
      // width和height是文件的最大宽高
      compress: {
        width: 600,
        height: 600,
        quality: 80
      },
      // 响应数据处理,最终返回图片链接
      uploadHandler(responseText){
//      default accept json data like  {ok:false,msg:"unexpected"} or {ok:true,data:"image url"}
        var json = JSON.parse(responseText);
        if(json.status == 200){
          return json.data
        }else{
          alert(json.error)
        }
      }
    },
    // 语言，内建的有英文（en-us）和中文（zh-cn）
    language: "zh-cn",
    // 自定义语言
    i18n: {
      "zh-cn": {
        "align": "对齐方式",
        "image": "图片",
        "list": "列表",
        "link": "链接",
        "unlink": "去除链接",
        "table": "表格",
        "font": "文字",
        "full screen": "全屏",
        "text": "排版",
        "eraser": "格式清除",
        "info": "关于",
        "color": "颜色",
        "please enter a url": "请输入地址",
        "create link": "创建链接",
        "bold": "加粗",
        "italic": "倾斜",
        "underline": "下划线",
        "strike through": "删除线",
        "subscript": "上标",
        "superscript": "下标",
        "heading": "标题",
        "font name": "字体",
        "font size": "文字大小",
        "left justify": "左对齐",
        "center justify": "居中",
        "right justify": "右对齐",
        "ordered list": "有序列表",
        "unordered list": "无序列表",
        "fore color": "前景色",
        "background color": "背景色",
        "row count": "行数",
        "column count": "列数",
        "save": "确定",
        "upload": "上传",
        "progress": "进度",
        "unknown": "未知",
        "please wait": "请稍等",
        "error": "错误",
        "abort": "中断",
        "reset": "重置"
      }
    },
    // 隐藏不想要显示出来的模块
    hiddenModules: [],
    // 自定义要显示的模块，并控制顺序
    visibleModules: [
//    "text",
//    "color",
      "font",
      "align",
      "list",
      "link",
//    "unlink",
//    "tabulation",
      "image",
//    "hr",
//    "eraser",
      "undo",
//    "full-screen",
//    "info",
    ],
    // 扩展模块，具体可以参考examples或查看源码
    // extended modules
    modules: {
      //omit,reference to source code of build-in modules
    }
  };
  Vue.use(VueHtml5Editor, opt);
}
```

（3）main.js导入模块

```javascript
import initRichText from './common/initHTMLEditor.js'
import '../node_modules/font-awesome/css/font-awesome.css'
initRichText()
```

（4）组件使用 App.vue

```javascript
<template>
  <div class="content">
    <vue-html5-editor :content="content" :height="400"
                      @change="updateData"></vue-html5-editor>
  </div>
</template>
<style scoped>

</style>
<script>
export default {
  data(){

    return {content: '请输入文章内容'}
  },
  methods: {
    updateData(e = '') {
      this.content = e;
      console.info(e);
    }
  }
}
</script>
```

（5）跑项目：

```shell
npm run dev 
```
浏览：http://localhost:8080

