---
title: iframe加载的页面判断是否加载完成的方法
date: 2018-08-27 19:56:24
tags: 
    - JS
---
<meta name="referrer" content="no-referrer" />
head中添加
```html
<head><meta http-equiv="X-UA-Compatible" content="IE=10" /></head>
```

（1）
```javascript
var iframe = document.createElement("iframe");
iframe.src = "http://sc.jb51.net";
if (iframe.attachEvent) {
    iframe.attachEvent("onload",
    function() {
        alert("Local iframe is now loaded.");
    });
} else {
    iframe.onload = function() {
        alert("Local iframe is now loaded.");
    };
}
```

（2）

```javascript
element.onload = element.onreadystatechange = function(ev) {
    if (ev.readyState && ev.readyState != 'complete') {
        return;
    } else {
        var target = ev.currentTarget;
        setTimeout(function() {
            iframes.remove(target);
        },
        500);
    }
}
```


