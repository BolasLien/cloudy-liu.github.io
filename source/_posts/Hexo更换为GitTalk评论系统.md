---
title: Hexo-yilia使用gitalk/gitment评论系统
date: 2018-07-14 22:07:28
tags: [Hexo,yilia,gitment,gitalk]
---

来必力总是显示异常，现在抽时间找些资料，将博客评论系统更换为 `gitment/gitalk`。<!-- more --> 。最好两种都尝试之后，选择使用`gitalk` ，界面更加好看，配置也简单。下面就是两种配置的详细步骤。

 [Gitment](https://github.com/imsun/gitment) 的原理是利用 github 上的 issue 系统，也就是你提的评论都会对应生成 issue。当然账户只能是通过 github 账号。这点跟来必力有点差，不过博客的受众大部分都是程序员，所以也可以接受。

##  Hexo-yilia  gitment 配置

`yilia` 主题默认是支持 `gitments`的，所以只需要进行配置。

### 注册 OAuth Application

当别人评论你的文章时，会需要它是授权。点击 https://github.com/settings/applications/new

进行注册。注册界面如下。

![](https://coding.net/u/cloudy-liu/p/BlogPicBed/git/raw/master/OAuthRegister1.png)

注册成功后，会获取到 `Client ID/scerct` 。如下图所示，接下来就是将信息填入配置文件中。

![](https://coding.net/u/cloudy-liu/p/BlogPicBed/git/raw/master/clientID.png)

###  配置 `_config.yml` 文件

打开`themes\yilia\_config.yml` ,在如下位置填入正确的信息即可。

![](https://coding.net/u/cloudy-liu/p/BlogPicBed/git/raw/master/config.png)

* `gitment_owner`: 填写你的 github 账户名即可
* `gitment_repo`: repo 名字为可新建一个repo 或者使用博客托管的 repo 都行。
* `client_id`和`client_secret` : 就是上步骤中注册的获取的信息。

### 重新部署

配置完毕后，重新部署，即可看到效果。

```
hexo g -d 
```

### 解决Valid Fail

部署后，每次文章的评论都会需要初始化，但是试验是发现初始化失败，查找资料后，发现是 github issue 本身的规则限制（label 的长度最长为50），可参考[Gitment评论功能接入踩坑教程](https://www.jianshu.com/p/57afa4844aaa) 。

![](https://coding.net/u/cloudy-liu/p/BlogPicBed/git/raw/master/validfail.png)

解决方法，将 github issue 的 id 改为按照日期方式。

文件位置：`themes\yilia\layout\_partial\post\gitment.ejs`

```
ar gitment = new Gitment({
  // id: "<%=url%>",
  id: '<%= page.date %>',
  owner: '<%=theme.gitment_owner%>',
```

重新部署即可。

## Hexo-yilia gitalk 配置

[gitalk](https://github.com/gitalk/gitalk/) 风格感觉更加好看，维护的也比较勤。具体的配置方法，请参考 [Hexo主题yilia增加gitalk评论插件 ](https://ziven.cc/2018/07/03/Hexo%E4%B8%BB%E9%A2%98yilia%E5%A2%9E%E5%8A%A0gitalk%E8%AF%84%E8%AE%BA%E6%8F%92%E4%BB%B6/) 。然后重新部署`hexo d -g` 生成之后的样式如下。

![](https://coding.net/u/cloudy-liu/p/BlogPicBed/git/raw/master/gitalk_test.png)

(完）