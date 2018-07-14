---
title: Hexo yilia 主题一揽子使用方案
date: 2018-04-007 20:25:33
tags: [Hexo,yilia]
---

在用 Hexo 搭建完毕后，接着就寻找主题了，对比了几个主题<!-- more --> ，发现这个[yilia](https://github.com/litten/hexo-theme-yilia)  主题比较干净，简洁，于是就选了这个主题，但是有些细节不太习惯，于是就研究调整了一下，就是现在这个博客的样子。

## 查看所有文件，提示缺失模块 

`yilia` 在首次使用时，点击`所有文章` 时，会出现模块找不到的错误，可按照提示操作即可
注意一下，_config.yml 路径是指 根目录下的，而非 `yilia` 主题下的 config文件

![](http://p5sfmckwy.bkt.clouddn.com/img/3_1_yilia_loss_module.png)

## 配置图片资源
* **添加图片资源文件夹**。 路径为 `themes/yilia/source/`下，可添加一个 `assets` 文件夹，里面存放图片资源即可

- **配置文件中直接引用即可**。路径为 `themes/yilia/_config.yml`，找到如下即可

  ```
  # 微信二维码图片
  weixin:  /assets/img/wechat.png
  # 头像图片
  avatar:  /assets/img/head.jpg
  # 网页图标
  favicon:  /assets/img/head.jpg
  ```

## 文章如何显示摘要

* **问题**。点击主页时，发现所有文章都是全文显示，不利于查找，可控制显示的字数
* **解决办法**。 在你 MD 格式文章正文插入 `<!-- more -->`即可，只会显示它之前的，此后的就不显示，点击文章标题，全文阅读才可看到，同时注释掉以下 ` themes/yilia/_config.yml`，重复

  ```
  # excerpt_link: more
  ```

* 效果

  ![](http://p5sfmckwy.bkt.clouddn.com/img/3_2_yilia_摘要.png)

## 文章显示目录

增加文章目录 TOC(table of content )，方便阅读文章, 在 `themes/yilia/_config.ym`中进行配置 `toc: 2`即可，它会将你 Markdown 语法的标题，生成目录，目录查看在右下角。

![](http://p5sfmckwy.bkt.clouddn.com/img/3_3_yilia_目录.png)

## 增加归档菜单

修改 `themes/yilia/_config.yml` 

```
menu:
    主页:  /
    归档:  /archives/index.html
```

## 修改代码块样式
默认的代码样式太刺眼了，调成稍微柔和一些的，这里是调成 Atom 风格，以下为两种方式都可以，推荐第一种直接修改编译好的文件，不然还需要重新build。
* **直接修改编译好的文件**。路径为： `theme\yilia\source\main.0cf68a.css`
  * 修改代码背景色，搜索 `.article-entry .highlight`, 修改background后面的颜色

    ![](http://p5sfmckwy.bkt.clouddn.com/img/3_4_code_bg_color.png)

  * 修改代码字体颜色 `.article-entry .highlight .line`
  * 
    ![](http://p5sfmckwy.bkt.clouddn.com/img/3_5_code_font_color.png)
* **修改源文件重新build**。上述资源对应源文件为 `yilia\source-src\css\highlight.scss`，按照如下方式build

```
cd 到 yilia 目录下
npm install
npm run dev
npm run dist
```

## [增加不蒜子统计](http://ibruce.info/2015/04/04/busuanzi/)

利用这个统计，可以知道你博客的访问量

### 安装不蒜子脚本

在 `themes\yilia\layout\_partial\after-footer.ejs`最后添加

```
<script  async  src="https://dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>
```

### 添加统计网站访问量

修改 `themes\yilia\layout\_partial\footer.ejs`，包括访客数和站点访问总量

```
# PV方式，单个用户连续点击 n 篇，记录 n 次记录值
<span id="busuanzi_container_site_pv">    本站总访问量<span id="busuanzi_value_site_pv"></span>次</span>

# UV方式，单个用户连续点击 n 篇，记录 1 次记录值
<span id="busuanzi_container_site_uv">  本站访客数<span id="busuanzi_value_site_uv"></span>人次</span>
```

###  单篇文章点击量

 * `themes\yilia\layout\_partial\article.ejs`中 在 `<%- partial('post/title', {class_name: 'article-title'}) %>` 插入如下代码

```
<!--显示阅读次数-->
<% if (!index && post.comments){ %>
  <br/>
  <a class="cloud-tie-join-count" href="javascript:void(0);" style="color:gray;font-size:14px;">
  <span class="icon-sort"></span>
  <span id="busuanzi_container_page_pv" style="color:#ef7522;font-size:14px;">
            阅读数: <span id="busuanzi_value_page_pv"></span>次 &nbsp;&nbsp;
  </span>
  </a>
  <% } %>
```
## 添加来必力评论系统

[点击这个链接]( http://www.zhoujy.me/2017/07/16/livere/) 查看

##  添加版权信息

[点击这个链接](https://blog.zscself.com/2017/01/25/ee4d9ecb/) 查看

##  插入网易云音乐

* 登入网易云音乐网页版，选择一首歌，点击歌曲详情，点击生成外链播放器

  ![](http://p5sfmckwy.bkt.clouddn.com/img/3_5_wangyiyun.png)

* 复制外链代码，插入你需要编辑的 MD 格式文章里面，即可

![](http://p5sfmckwy.bkt.clouddn.com/img/3_5_wangyiyun_2.png)

##  百度/Google统计/SEO

[点击这个链接](http://moxfive.xyz/yelee/5.Vendor/baidu-tongji.html)查看，这几项都是相同的

## 七牛云图床

博客内容最麻烦的就是插入图片，我们可以使用七牛云提供的 10G 的免费存储空间，将图片上传上去，然后生成外链，使用 Markdown 的图片引用方法即可，这样文章就脱离了图片编辑，转为在线了。同样一份文章，你部署在 `csdn` 等其他网页时，直接复制粘贴即可。

### 上传图片到七牛云

* 注册，并完成支付宝实名认证，实名认证后有10G的免费空间，[认证免费额度](https://developer.qiniu.com/af/kb/1574/free-credit-information)

* 添加文件。步骤为添加 `对象存储`, 新建一个存储空间，进入到该空间，点击 `内容管理`,点击`上传文件`

  ![](http://p5sfmckwy.bkt.clouddn.com/img/3_6_qiniu_1.png)

* **生成外链插入到文章中**

  ![](http://p5sfmckwy.bkt.clouddn.com/img/3_6_qiniu_2.png)

### 使用 [PicGo](https://github.com/Molunerfinn/PicGo) 自动生成外链

但是这样通过 web点击上传按钮方式，太效率了，这里使用 `PicGo` 工具，完成拖动自动生成外链，感谢作者。

* **查看你的七牛密钥**。登入七牛云查看密钥 `个人面板` -> `密钥管理`

![](http://p5sfmckwy.bkt.clouddn.com/img/3_6_3_qiniu_key.png)

* **PicGo 配置七牛账户**。然后就在上传图，拖动图片进去，即可生成外链，直接插入到文章中即可。

  ![](http://p5sfmckwy.bkt.clouddn.com/img/3_6_qiniu_config.png)


##  Demo测试

修改配置后，输入以下三条命令即可部署

```
hexo clean
hexo g
hexo d
```

 [点击这里](https://www.liuyun.fun/2018/02/27/test/) 查看博客正文的效果



以上希望对你所有帮助。



（全文完）