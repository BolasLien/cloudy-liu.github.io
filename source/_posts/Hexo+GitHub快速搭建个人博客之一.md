---
title: Hexo+GitHub 快速搭建个人博客之一
date: 2018-02-026 20:25:33
tags: [Hexo,github,博客]
---

你可能听过一句话，叫做**输出倒逼输入** ，<!-- more--> 如果你能把某个主题的写出来，并且别人还能够看得懂，那么说明你真的掌握了这个事情，写博客既可以方便分享，又可以作为自己日后查阅复盘的记录，一举多得。对我而言，除了学习，建立个人的连接渠道之外，练习把事情精炼的说清楚是最重要的目的。至于为什么要建立个人博客，可以看看这个[知乎贴子](https://www.zhihu.com/question/19916345)的讨论。

好了，现在我告诉你只需最低花费 2 块钱就可以拥有一个属于个人域名的博客，你要不要。我这里使用的是 *Hexo* + *GitHub Pages*  的搭建方式。 *Hexo* 是一个快速，简单，强大的静态博客框架，支持 *Markdown* ，插件众多，部署快速，安装也非常友好，这就使得本地部署一个博客非常简单，但是我们需要别人也能够访问我们的博客，所以需要一台服务器，但是这需要花钱购买。 

不过为了免费，我们可以使用 *GitHub Pages*  或是 *Coding Pages*  提供的托管服务， *GitHub Pages*  是用来为项目作展示的，也可以用来作为托管博客，这样两者结合就可以搭建出一个免费的博客网站，当然目前它的域名还是*Github* 项目专有域名，我们还需要有个人域名，这就是唯一需要花钱的地方，当然花2块钱，你就可以租一个域名一年，有了个人域名之后，在将其解析到*GitHub Pages* 网址上，以后就可以通过你的个人域名直接访问博客了，接下来，我们就开始动手搭建。

我的本机环境以及此次需要安装的软件如下
* windows 7 ,64 位
* Node.js 
* Cmder

##  Windows 超赞的终端 Cmder
如果你在 Windows 上经常使用控制台的话，你一定会对 Windows 自带的 `cmd.exe` 深恶痛绝，漆黑的背景，看久了眼睛难受，无法窗口多开，多任务处理难受。`cmder` 就是 windows 控制台终端的福音，有了它，你就可以享受漂亮可自定义的 UI 界面，免安装，下载解压即用，使用完整版，还整合了 *Git* ，*linux*  下大部分命令也可以直接使用，比如 `ls`  `grep`之类的，一个 `Cmder` 全搞定，于是我就把 `Git Bash` 给卸载了。

[官网](http://cmder.net/) 下载**完整版**，然后解压，找到 `cmder.exe` 双击即可启动，以后我们就用它作为默认的终端，若 `GitHub release` 下载很慢，可以先去下载  [freedownloadmanager](https://www.freedownloadmanager.org/) ，这个下载 `GitHub release` 软件超快的。以后我们的命令都是在 `cmder.exe `中下的。
![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/cmder.png)

## 创建 GitHub Pages
### 注册 GitHub 账号

注册[GitHub](https://github.com) 账号的步骤，我就不贴了，去官网注册即可

### cmder 终端初始 git
*GitHub* 是通过 *git* 进行操作的，它是一个分布式的版本控制系统，现在大部分开源软件都是由 *Git* 作为版本控制系统。所以之前，我们需要做一次全局的账号和邮箱的设定，用户名与邮箱和*GitHub* 账号注册时一致。

```
$ git config --global user.name cloudy-liu
$ git config --global user.email cloudy-liuu@gmail.com
```

### ssh 密钥绑定
这步目的是为安全性验证。
* 在`cmder` 终端中输入以下命令产生 ssh 密钥，邮箱为*GitHub*  注册邮箱，有提示的，可一路回车键

```
ssh-keygen -t rsa -C "cloudy-liuu@gmail.com"
```

![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/coding_ssh_1_2_create.png)

* 进入到*GitHub*  网站中， `Settings` 左边栏`SSH and GPG keys` `New SSh Key ` ，将刚才产生的公钥内容(`C:\Users\Administrator\.ssh\id_rsa.pub`) 粘贴进去即可。
* `cmder` 中测试。输入以下命令，遇到确认信息，输入 `yes` ，成功如下所

```
λ ssh -T git@github.com
The authenticity of host 'github.com (13.229.188.59)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,13.229.188.59' (RSA) to the list of known hosts.
Hi cloudy-liu! You've successfully authenticated, but GitHub does not provide shell access.
```

### 创建github.io项目
*GitHub Pages* 是通过创建一个名为 *username.github.io* 的项目达成的，每个账号只能创建一个，可以参照官网(https://pages.github.com/）操作
* 登入`GitHub` 账号
* 新建一个仓库。命名为`username.github.io` ，参看以下我的创建，勾选一下初始化 `README` 文件

![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/cloudy_github_io.png)

* 创建成功后，该 `Pages` 就可以使用了，可以在 `Settings`  `GitHub Pages` 中看到 Publish 了

![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/cloudy_io_publish.png)

* 此时任何人都可以通过浏览器打开访问上述网址了，查看内容了，当然目前这里只有一个 `README `文件

![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/visit_github_io.png)

##  Hexo 搭建并部署博客 
### 下载 Node.js
*Hexo* 是使用`Node.js`开发的，所以为了安装它，我们需要先安装 `Node.js `工具。去[官网]( https://nodejs.org/en/download/) 下载安装最新版本的，安装就一路 next 即可，最后它会被加入到系统环境 path 中。

![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/nodejs_install_1.png)

 输入以下命令，查看是否安装成功

```
C:\Users\Administrator
λ node -v
v8.9.4
```

### 安装 Hexo
`Node.js` 安装好以后，同时为我们装好了 `npm` 工具， 这是一个包管理工具，通过它我们可以下载各种插件，*cmder.exe* 中输入以下命令进行安装，查看是否安装成功可以输入 `hexo -v` 命令  ，至此 *Hexo*  就安装好了。

```
 C:\Users\Administrator
λ npm install hexo-cli -g
C:\Users\Administrator\AppData\Roaming\npm\hexo -> C:\Users\Admi
...//省略
+ hexo-cli@1.0.4
updated 5 packages in 53.458s

C:\Users\Administrator
λ hexo -v
hexo-cli: 1.0.4
..//省略
tz: 2017b
```

### 创建本地博客框架
*Hexo* 安装好后，我们就可以轻松的开始创建博客了，常用的就几条命令。
* **初始化博客**。用来初始一个博客系统，我们新建一个目录 `blog`，并且在 `cmder`中进入到该目录，执行初始化`hexo init`命令。

```
E:\blog
λ hexo init
INFO  Cloning hexo-starter to E:\blog
Cloning into 'E:\blog'...
remote: Counting objects: 62, done.
remote: Total 62 (delta 0), reused 0 (delta 0), pack-reused 62
...
added 338 packages in 69.358s
INFO  Start blogging with Hexo!
```

* **查看文件目录结构**。初始化后，它的文件结构如下。其中 *source/_posts* 文件件，这个就是博客的内容，我们以后可以在这个目录新建 `.md` 文件，就可以了。初始状态下有个默认的`hello-world.md` 文件。

![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/init_file_hexo.png)

* **本地部署**。 通过 `hexo g` `hexo s`  就可以查看了，然后在浏览器中输入 *http://localhost:4000/*  就可以查看刚才本地部署的博客，至此本地博客系统已经成功架起来了，按下`ctrl+c` 终止，接下来，我们将此博客部署到 GitHub Pages 上去。

![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/bushu.png)
![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/rst_github_io.png)


**部署博客到 GitHub Pages上** 
*Hexo*  中可以通过修改配置文件，来指定需要部署到哪里去，它使用的是 `yaml`格式文件，对格式很严格，记住冒号后需要加一个空格。
- 修改 `_config.yaml` 文件。这个文件可以修改标题，作者等信息，拉到最下面是部署的目的地，按照如下格式设定，我的部署如下

    ```
    deploy:
      type: git
      repository:
        github: git@github.com:cloudy-liu/cloudy-liu.github.io.git
      branch: master
    ```

- 安装 `Hexo git ` 部署插件，让 `Hexo `知道通过什么类型部署，这里是 `git` 

```
E:\blog
λ npm install hexo-deployer-git --save
```

- 开始部署到 *GitHub Pages*，通过如下命令 `hexo clean` `hexo g` `hexo d`  

  ```
  E:\blog
  λ hexo clean
  INFO  Deleted database.
  INFO  Deleted public folder.

  E:\blog
  λ hexo g
  INFO  Start processing
  ...
  INFO  28 files generated in 641 ms

  E:\blog
  λ hexo d
  INFO  Deploying: git
  INFO  Setting up Git deployment...
  ....
  To git@github.com:lynnbest/lynnbest.github.io.git
   + bbfcc20...08c6344 HEAD -> master (forced update)
  INFO  Deploy done: git
  ```

- 浏览器打开 `https://cloudy-liu.github.io/` ，就可以看到部署到结果。

  ![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/rst_github_io.png)

至此，别人就可以通过类似 `https://cloudy-liu.github.io` 方式访问你的博客了。

## 小结
本文详细记录了如何使用*Hexo* 结合 *GitHub Pages* 快速搭建个人博客，同时推荐了一个 Windows 超好用的终端 Cmder ，如果觉得本文对你有用，那就一个转发或者一个赞吧。同时欢迎关注微信公众号 "YunShell"。
![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/wechat_qrcode.jpg)

（全文完）