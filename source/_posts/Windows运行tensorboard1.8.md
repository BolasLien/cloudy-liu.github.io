---
title: Windows 运行 tensorboard 1.6
date: 2018-05-31 23:10:31
tags: [ML,Tensorflow]	
---

本篇讲述一下 Windows 环境下如何运行 Tensorboard 可视化 graph.<!-- more -->

## 环境

* Windows 7
* tensorflow 1.6/1.8

`tensorlfow` 1.6 在 windows 上通过 `pip3 install tensorflow` 安装，同时会顺带安装 `tensorboard ` 工具，1.6 版本已经修改过了文件路径，具体的使用方式，参看如下例子。本机是1.8版本，1.6 也同样适用。

## 例子

### 创建 model

首先我们新建一个 `test.py`, 编写一个最简单的矩阵乘法，最后将其graph 保存下来，代码如下，

```python
import tensorflow as tf

x = tf.constant([1, 2, 3, 4, 5, 6], shape=[2, 3], name="x")
w = tf.constant([1, 2, 3, 4, 5, 6], shape=[3, 2], name="w")
b = tf.constant([1, 2, 3, 4], shape=[2, 2], name="b")
y = tf.matmul(x, w)+b # 计算 y=x*w +b 的模型

with tf.Session() as sess:
    rst = sess.run(y)
    print("rst:", rst)
    tf.summary.FileWriter("log", graph=sess.graph) # 保存graph 到 当前文件 log目录下

>>> 输出结果
rst: [[23 30]
 [52 68]]
```

运行完毕后，会在文件的当前路径，创建一个 `log` 文件夹，里面存有 graph 的信息，我本地目录结构如下

```
-- tmp
  -- log
    --- events.out.tfevents.1527778757.PC007
  -- test.py
```

### 使用 tensorboard 查看可视化图形

因为安装了 tensorflow 后，tensorboard 是会自动安装，如何运行文件呢？在终端中运行。

`ctrl + R` -> `输入cmd` -> 回车，进行终端。然后运行如下命令，最后启动一个本地服务器。如下图所示，其实运行的是 `tensorboard `包下 `main.py` 文件。格式为 `py -3 xx/tensorboard/main.py --logdir=path` path就是你在tensorflow 保存时的路径。

```python
py -3 C:\Users\Administrator\AppData\Local\Programs\Python\Python36\Lib\site-packages\tensorboard\main.py --logdir=D:/tmp/log
```

![](https://coding.net/u/cloudy-liu/p/BlogPicBed/git/raw/master/1.png)

然后在 chrome 浏览器中输入终端出现的信息 `http:cloudy:6066` ，回车即可看到图形了

![](https://coding.net/u/cloudy-liu/p/BlogPicBed/git/raw/master/2.png)

### 遇到的错误

* chrome 浏览器需要升级。刚开始时候，我chrome 浏览器啥都不可见，升级一下 chrome 浏览器 就好了，我目前的版本是 `版本 67.0.3396.62（正式版本） （32 位） `。