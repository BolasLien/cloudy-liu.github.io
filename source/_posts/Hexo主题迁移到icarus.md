---
title: Hexo 主题迁移到 icarus
date: 2019-6-23 23:07:28
tags: [Hexo,icarus]
---

`yilia` 在大屏下被拉伸的太长了，一直想换三栏模式，最近看到 `icarus`主题，设计的非常漂亮，于是果断换 (怎么现在尽是在折腾Hexo 主题啊，Orz...)   <!-- more --> 

## 配置 icarus 主题

下载 [icarus](https://github.com/ppoffice/hexo-theme-icarus/releases) ，在 `hexo` 的 `themes`  文件夹，新加入 [icarus](https://github.com/ppoffice/hexo-theme-icarus) 目录，配置 `hexo` 的根目录的 `_config.yml` 

```diff _config.yml
-theme: yilia
+theme: icarus
```

然后更新部署就好了

```javascript
hexo d -g
```

## 自定义 icarus 主题

默认配置也基本能用了，但是有一个痛点就是，阅读模式文章宽度太短了，还是根据个人习惯做下配置下

### 配置 gitalk 评论

`icarus` 默认是支持 `gitalk` 的，所以只需要在 `themes/icarus/_config.yml` 中设置一下即可

```diff themes/icarus/_config.yml
comment:
    # Name of the comment plugin
    type: gitalk
    owner: cloudy-liu         # (required) GitHub user name
    repo: cloudy-liu.github.io   # (required) GitHub repository name
    client_id: xxxx # (required) OAuth application client id
    client_secret: xxx # (required) OAuth application client secret
    admin: cloudy-liu
```

还没配置的可以[参考这篇](https://www.liuyun.fun/2018/07/14/Hexo%E6%9B%B4%E6%8D%A2%E4%B8%BAGitTalk%E8%AF%84%E8%AE%BA%E7%B3%BB%E7%BB%9F/)做配置

### 代码高亮

[代码高亮款式](https://github.com/highlightjs/highlight.js/tree/master/src/styles) , [预览参见这里](https://highlightjs.org/)，`Android` 程序员选择 `androidstudio` 效果不错

```themes/icarus/_config.yml
highlight: androidstudio
```

### 调整阅读模式双栏

判断 `post` 页面，显示目录 `toc` ，修改宽度

```diff themes/icarus/includes/helpers/layout.js
-        return widgets.filter(widget => widget.hasOwnProperty('position') && widget.position === position);
+        if (this.page.layout !== 'post') {
+            return widgets.filter(widget => widget.hasOwnProperty('position') && widget.position === position);
+        }
+        if (position === 'left') {
+            return widgets.filter(widget => widget.hasOwnProperty('position') && (widget.type === 'toc'));
+        } else {
+            return []
+        }
```

```diff themes/icarus/layout/common/widget.ejs
         case 2:
-            return 'is-4-tablet is-4-desktop is-4-widescreen';
+            return 'is-4-tablet is-4-desktop is-3-widescreen';
```

```diff themes/icarus/layout/layout.ejs
 <head>
     <%- partial('common/head') %>
 </head>
-<body class="is-<%= column_count() %>-column">
+<body class="is-3-column">
     <%- partial('common/navbar', { page }) %>
             case 2:
-                return 'is-8-tablet is-8-desktop is-8-widescreen';
+                return 'is-8-tablet is-8-desktop is-9-widescreen';
             case 3:
                 return 'is-8-tablet is-8-desktop is-6-widescreen'
         }
```

```diff themes/icarus/source/css/style.styl
     .is-2-column .container
         max-width: screen-desktop - 2 * gap
         width: screen-desktop - 2 * gap
+    .is-3-column .container
+        max-width: screen-widescreen - gap
+        width: screen-widescreen - gap
 @media screen and (min-width: screen-fullhd)
+    .is-3-column .container
+        max-width: screen-fullhd - 2 * gap
+        width: screen-fullhd - 2 * gap

```


### 增加文章版权

```diff themes/icarus/layout/common/article.ejs
         <div class="content">
             <%- index && post.excerpt ? post.excerpt : post.content %>
         </div>
+        <% if (!index && post.layout === 'post' && post.copyright !== false) { %>
+            <ul class="post-copyright">
+            <li><strong>本文标题：</strong><a href="<%= post.permalink %>"><%= page.title %></a></li>
+            <li><strong>本文作者：</strong><a href="<%= theme.url %>"><%= theme.author %></a></li>
+            <li><strong>本文链接：</strong><a href="<%= post.permalink %>"><%= post.permalink %></a></li>
+            <li><strong>发布时间：</strong><%= post.date.format("YYYY-MM-DD") %></li>
+            <li><strong>版权声明：</strong>本博客所有文章除特别声明外，均采用 <a href="https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh" rel="external nofollow" target="_blank">CC BY-NC-SA 4.0</a> 许可协议。转载请注明出处！
+            </li>
+            </ul>
+        <% } %>
         <% if (!index && post.tags && post.tags.length) { %>
```

下面文件加入格式

```diff themes/icarus/source/css/style.styl
             border: none
     .file
         all: initial
+
+.post-copyright
+    font-size: 1rem
+    letter-spacing: 0.02rem
+    word-break: break-all
+    margin: 2.5rem 0 0
+    padding: 1rem 1rem
+    border-left: 3px solid #FF1700
+    background-color: #F9F9F9
```

这里大部分是参照 [自定义icarus](https://www.alphalxy.com/2019/03/customize-icarus/) 做的修改(点赞)

###  更新logo

`icarus` 有默认的 logo，这里想改变一下，logo 是在 `themes\icarus\source\images` 里面`logo.svg` 文件，我们只需要替换成自己的 logo 文件即可，可以从[这里](https://icomoon.io/app/#/select) 去获取。

### 总访问量统计

`icarus` 是用卜算子统计，默认只统计了访问人数，并没有访问量统计，需要在页脚修改下

```diff themes/icarus/layout/common/footer.ejs
@@ -15,9 +15,8 @@
                         href="https://github.com/ppoffice/hexo-theme-icarus" target="_blank">Icarus</a>
                 <% if (has_config('plugins.busuanzi') ? get_config('plugins.busuanzi') : false) { %>
                 <br>
-                <span id="busuanzi_container_site_uv">
-                <%- _p('plugin.visitor', '<span id="busuanzi_value_site_uv">0</span>') %>
-                </span>
+                <span id="busuanzi_container_site_uv"> 来访 <span id="busuanzi_value_site_uv"></span>人</span>
+                <span id="busuanzi_container_site_pv">, 总访问 <span id="busuanzi_value_site_pv"></span>次</span>
                 <% } %>
                 </p>
```

修改主题后，建议先进行清理，不然可能由于缓存问题，导致本地预览与部署不一致，KO !

```
hexo clean
hexo d -g
```

## 参考资料

* [icarus文档]( https://blog.zhangruipeng.me/hexo-theme-icarus/)
* [自定义 icarus](https://www.alphalxy.com/2019/03/customize-icarus/)