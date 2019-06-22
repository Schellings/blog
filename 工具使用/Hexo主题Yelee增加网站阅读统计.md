---
title: Hexo主题yelee增加网站阅读统计
date: 2019-04-09 16:56:24
tags: 工具使用
---
Hexo的主题yelee默认没有对网站访问信息的统计，修改方法如下：

## 修改yelee关于不蒜子网站计数设置

```yaml
visit_counter:
  on: true
  site_visit: true
  page_visit: true
```
## 修改底部样式文件

修改文件：themes/yelee/layout/_partial/footer.ejs
替换内容：
```ejs
<script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
<footer id="footer">
    <div class="outer">
        <div id="footer-info">
            <div class="footer-left">
                <i class="fa fa-copyright"></i>
                <% if (theme.since && !isNaN(theme.since) && theme.since < date(new Date(), 'YYYY')) { %><%- theme.since%>-<% } %><%= date(new Date(), 'YYYY') %> <%= config.author || config.title %>
            </div>
            <div class="footer-right">
                <a href="http://hexo.io/" target="_blank" title="<%= __('tooltip.Hexo') %>">Hexo</a>  Theme <a href="https://github.com/MOxFIVE/hexo-theme-yelee" target="_blank" title="<%= __('tooltip.Yelee') %>  v<%= theme.Yelee %>">Yelee</a> by MOxFIVE <i class="fa fa-heart animated infinite pulse"></i>
            </div>
        </div>


        <% if (theme.visit_counter.on) { %>


            <div class="visit">
                <% if (theme.visit_counter.site_visit) { %>
                    <span style='display:inline'>
                        <span id="site-visit" title="<%= __('visit_counter.site') %>"><i class="fa fa-user" aria-hidden="true"></i>本站总访问>量<span id="busuanzi_value_site_pv"></span>次
                        </span>
                    </span>
                <% } %>


                <% if (theme.visit_counter.site_visit && theme.visit_counter.page_visit) { %>
                    <span>| </span>
                <% } %>


                <% if (theme.visit_counter.page_visit) { %>
                    <span style='display:inline'>
                        <span id="page-visit"  title="<%= __('visit_counter.page') %>"><i class="fa fa-eye animated infinite pulse" aria-hidden="true"></i>本页面被访问<span id="busuanzi_value_page_pv"></span>次
                        </span>
```
将整个文件替换以上内容

## 重启hexo

hexo clean

hexo s