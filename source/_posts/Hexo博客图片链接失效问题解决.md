---
title: Hexo博客图片链接失效问题解决
date: 2019-5-23 23:07:28
tags: [Hexo,七牛云]
---



博客中的图片失效很久了，一直没时间来处理。现在把解决过程整理了一下，供同样问题的人参考<!-- more --> 

### 失效原因

因为我的博客图床是托管在七牛云存储上(个人免费10G)，但是目前七牛云提供的测试域名均只有 30天的试用期，过了试用期会被回收，导致图片链接失效。解决的方法就是按照七牛云的指导，绑定自定义域名，并且该域名必须要工信部备案，而这个备案过程至少要20+天，时间非常长，我这目的也仅仅是为博客做图床而已，因此不想去申请麻烦的备案。因此另谋出路。

### 需求整理

* 另找一个可简单使用的图床，不需要备案之类的
* 可一直存储图片
* 访问速度要还凑合

最后经过选择，使用 `coding` 项目作为图床，`coding`跟腾讯云开发者绑定后，项目数和速度都免费畅享，也就是项目可以一直开下去，下图就是升级之后的好处

![](https://coding.net/u/cloudy-liu/p/BlogPicBed/git/raw/master/coding_update_to_tencent_cloud.png)

###  图片转移 Step

1. 新建 `coding` 项目，将之前博客中的图片都上传上去

我因为本地没有保存之前的图片了，这个可以在七牛云上申请工单处理，可延长你的测试域名期限，我的延长了3天，然后批量下载下来了。

上传完了图片后，随便点击一张图片，查看这个图片的完整链接 比如 这里：` https://coding.net/u/cloudy-liu/p/BlogPicBed/git/blob/master/1.png`，我们将其中的`blob`  替换成 `raw` ，然后在修改为 `markdown`的链接语法： `![](https://coding.net/u/cloudy-liu/p/BlogPicBed/git/raw/master/1.png)` 就可以在 markdown 支持的文本中看到这张图片了

![](https://coding.net/u/cloudy-liu/p/BlogPicBed/git/raw/master/coding_update_to_tencent_cloud.png)

2. 使用新的图片链接替换你原博文中的图片链接

因为用 `markdown` 写的博文，图片都是链接的形式，比如之前的博文图片链接都是七牛云域名的图片，如 `![](http://p5sfmckwy.bkt.clouddn.com/img/2_top_2_rmb.png)`，现在仅需要替换为新的域名即可，因为图片的名字不用改，只需要修改之前的域名为 `coding` 新建项目上的域名即可，当然博文非常多，我这里写了简单的 python 脚本来一次性批量处理

* `replace_pic.py` 代码如下

```python
# coding=utf-8
import argparse
import os
import re

OLD = "http://xxx.clouddn.com/img/" #修改为你原来图片地址
NEW = "https://xxx/git/raw/master/" #修改为你coding项目图床地址

def get_arg():
    parser = argparse.ArgumentParser()
    parser.add_argument("-p", "--path", type=str,
                        dest="path", help="blog article path")
    args = parser.parse_args()
    path = args.path
    print("path: ", path)
    replace_path = path.replace("\\", "/")
    print("replaced_path:", replace_path)
    return replace_path


def main(path):
    file_list = [os.path.abspath(os.path.join(root, file)) for root, _, files in os.walk(path) for file in files]
    print(file_list)

    print("start ..")
    for f in file_list:
        content = ""
        with open(f, "rb") as fp:
            file_content = fp.read()
            content = re.sub(OLD, NEW, file_content)
        with open(f, "wb") as fp:
            fp.write(content)


if __name__ == '__main__':
    main(get_arg())

```

* 使用方式，-p 指定目录后，会将该目录下的所有文件，将旧的链接替换为新的链接

  ```
  python 路径/replace_pic.py  -p 你博文的目录
  ```

* 修正后，在更新 `hexo d -g` 进行更新博客即可，图床即更换了