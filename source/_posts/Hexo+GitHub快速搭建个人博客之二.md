---
title: Hexo+GitHub 快速搭建个人博客之二
date: 2018-02-027 20:25:33
tags: [Hexo,github,博客,独立域名]
---

在上一篇 [Hexo+GitHub 快速搭建个人博客之一](https://www.liuyun.fun/2018/02/27/Hexo+GitHub 快速搭建个人博客之二/ ) 中，介绍了如何使用 GitHub Pages 和 Hexo 搭建一个博客，<!-- more--> 但是还没有绑定个人域名，如果你没有个人域名，那就要去购买一个了，这也是唯一一个需要花钱的地方，当然最低只需要花费2块钱就可以了，这个接下来会谈到具体的操作。

另外，本篇还会顺带讲解一下，如何将博客同时部署在 Coding 上。Coding.net  是提供类似于 GitHub 服务的国内网站，好处在于访问速度快很多，同时一样提供了好用的 Coding Pages 功能，用于托管个人博客服务。后面会讲解到 Hexo 如何同时部署到 GitHub Pages 和 Coding Pages，其实类似于 GitHub Pages 的操作。

下面将按照以下顺序进行动手操作
* GitHub Pages 绑定个人域名
  * 阿里云购买域名
  * 将域名映射到 GitHub Pages 上
* Hexo 同时部署博客到 GitHub Pages 和 Coding  Pages 上

## GitHub Pages 绑定个人域名
### 阿里云购买域名
因为我在阿里云购买了域名，所以这里以阿里云为例。当然也可以在其他地方购买，比如腾讯云。进入阿里云官网，找到 *产品/域名与网站* ，点击进入，如果没有阿里云账号，需要先注册，如下图所示。

![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/1_yumming_setup.png)

此时你可以看到各个后缀的域名了，有 `.com` `.top` 之类的，可以看到 `.top` 的域名最低只需要2块钱就可以，购买前先查询一下你的域名是否已被注册了，购买流程就不赘述。购买完毕后，域名需要实名认证，这个过程大概一至两天就可以通过了，之后这个域名就是你属于你个人专有了。

![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/2_top_2_rmb.png)

### 将域名映射到GitHub Pages上
这步的目的是，当我们在浏览器中直接输入我们刚才购买的域名（比如我的 www.liuyun.fun），就可以直接跳转到我们部署到 GitHub Pages 上的博客网站，而无须输入类似 username.github.io 这样不好记的网址了。那么要如何操作呢？在上篇中，我们已经能通过 username.github.io 的网址来访问托管的博客了，为了达到刚才的目的，我们需要做的是，建立个人域名与 username.github.io 的映射关系，要完成这个映射关系，需要按照以下几个步骤进行操作。

#### 获取你 github.io 的 IP 地址
通过 Ping  你的github.io 这个网址，如下图所示

   ![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/5_ping_ip.png)

#### 域名服务商进行域名解析
我这里是阿里云的后台管理界面，进入到 *云服务 DNS*  界面，点击 *解析设置* 

![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/3_parse_setting.png)

然后点击添加 *添加解析*

![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/4_添加parser.png)

在弹出的界面中，选择记录类型为 `A`  类，意思是该域名的地址会跳转到我们设定的目的地，记录值方框填写刚才的 Ping 出的 IP 地址。这里我们添加两个 一个是 `www`，一个是 `@`

![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/6_yuming_add_parse.png)

添加完毕后，结果是这样的

![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/7_ailiyun_setting.png)

#### GitHub Pages 设定
进入到你自己的 GitHub Pages 项目，我这里的是  cloudy-liu.github.io  ，进入该项目的 Settings ，向下拖动到 GitHub Pages 位置，目前你看到的是该网站 publish 到 https://cloudy-liu.github.io
我们找到 Custom domain 进行自定义域名绑定，这里输入你购买的自定义域名，我这里是 www.liuyun.fun ，如下图所示。

![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/7_github_setting.png)

保存完毕后，它 publish 的网址就更改为你刚才修改过的域名地址了，需要注意的是，刚才我们的动作，其实在 GitHub Pages 仓库中添加了一个 CNAME 的文件，该文件内容就是保存自定义域名的地址，这也就是，很多人也可以通过添加 CNAME 文件来绑定域名，其实这是一回事，只不过现在 Github 直接支持绑定自定义域名了，就不同在 push 代码了。需要注意的是，你每次更新文章时，会重新 clean 一次，因此需要将 CNAME 文件加入到你Hexo source 中，这样确保每次可以 push 到 github 仓库中, [了解更多](https://www.zhihu.com/question/28814437)

![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/8_after_setting_github.png)

此时，你在浏览器中输入你自己的域名网址，就可以直接跳转到 Github Pages 的个人博客了，至此，个人博客的就绑定完毕了。

## 同时部署博客 Coding  Pages 上
这步其实是个备选题，如果你对博客的访问速度有需求，那么就可以考虑同时在部署博客到 Coding 上，毕竟国内的服务器速度要快。

开启 Coding Pages 的步骤其实和 GitHub Pages 大同小异，它们官网说明的非常清楚，只需要照着做就好了，没什么困难。

### 注册一个 Coding 账号

需要提醒的是，为了一致性，可以将 Coding 的账号邮箱设定和 Github 一样，这样一同部署就方便多了。

### 开启 Pages 服务
官网已经讲解非常清楚，照做就好， https://coding.net/pages/

###  在 Coding 网站， 绑定 ssh 
可以复用之前在 GitHub 绑定已经生成好的 ssh 的内容。绑定好后，在通过以下命令，检查一下

```
$ ssh -T git@git.coding.net
```

### 增加部署到coding 网站

修改  Hexo 的 `_config.yml`  文件

```
deploy:
  type: git
  repository:
    github: git@github.com:cloudy-liu/cloudy-liu.github.io.git
    coding: git@git.coding.net:cloudy-liu/cloudy-liu.coding.me.git
  branch: master
```

接下来，就可以使用 `hexo d` 部署到新博文到 Github 和 Coding ，当然这里 Coding 还是需要通过 Coding Pages 的域名直接访问，比如我的是 `cloudy-liu.coding.me` 访问刚才更新的博客内容。

## 小结

本篇讲解了 Github Pages  如何绑定个人域名，以后就可以通过个人域名直接访问博客，同时顺带提了一下同时将博客部署到  Coding 中的几个步骤，希望对你有帮助。

(全文完）



相关链接：

- [Hexo+GitHub 快速搭建个人博客之一](https://www.liuyun.fun/2018/02/27/Hexo+GitHub 快速搭建个人博客之一/ )