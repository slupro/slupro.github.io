---
layout: post
title:  "第一篇blog，windows下用jekyll docker写blog"
subtitle:   "Write blog by using jekyll docker in windows"
date:   2021-06-27 12:00:00 +10:00
author: "Steven Lu @slupro"
categories: [建站, 随便说说]  # up to 2.
tags: [docker, jekyll]  # TAG names should always be lowercase, 0 to infinity.
---

准确的说并不是用jekyll写blog，而是用jekyll docker在本地验证自己的blog，然后好推送到github page上去。

### 需求：我想有个blog

一直想建个blog记录东西，可以当作笔记一样记录工作和学习中的一些足迹，还可以分享。之前也简单调研过一些建blog的信息，说说我的感受，主要是缺点哈：:joy:

#### wordpress

用wordpress、joomla之类的感觉太重，虚拟主机便宜且操作起来简单，但是还是需要自己维护，包括安全性、备份等。blog内容和图片等在MySQL上，以后要是迁移到别的系统，可能都有未知的坑。感觉又要依赖MySQL里的数据，又要依赖wordpress本身，缺一个，数据的重新获取就会有麻烦。想到了西红柿炒鸡蛋，如果西红柿和鸡蛋分别放在两个篮子里，那有一个篮子出了问题，西红柿炒鸡蛋就做不出来了，而两个篮子里任意一个出问题的概率比一个篮子要大。

#### 简书

CSDN、简书、头条之类的blog，感觉界面上有些东西自己不可控，比如广告、弹窗、推送什么的，体验不好。有些人可能会觉得这上面的推荐也会给自己带来流量，这个仁者见仁智者见智了，比起那一点流量来说，我还是喜欢给自己一个更清爽的浏览界面。

#### Github pages

因为我的blog会记录些技术的东西，所以 Github pages 当时也有考虑，它有两种建blog的方式：

1. 以jekyll为代表，提交markdown文档和脚本到github，由github来编译并发布。
2. 以Hugo为代表，在本机编译好页面后，提交到github。

我倾向于第一种方式，提交markdown文件上去由github来编译，否则用Hugo编译的话本地需要保存原始的markdown文件和编译后的页面文件。

但是我当时在windows环境折腾jekyll，各种版本兼容性问题，我对ruby又不熟，最后放弃了，毕竟我是想用blog来记录，而不是想折腾blog :rofl:

#### 不是blog的笔记本

> 可能是我要求太多了：支持markdown，编辑方便，跨iphone、安卓、windows、mac无缝同步。

1. 印象笔记：缺点是分享不方便，手机端不能编辑markdown的笔记。不过之前一直在用它，还充了很久会员，就当网页保存器吧。
2. Notion：好看，但是担心公司小，哪天数据没了，就算文本和图片都有备份，如何导入到别的系统也会有麻烦。

### 最终方案

前几天偶然看到jekyll有docker版了，这样就又可以用jekyll，又不用在windows环境折腾ruby和jekyll的库了。最终找到一个blog方案可以满足我的全部要求了：:smiley:

> 本地测试页面简单，一条docker语句就好。
> 
> markdown里面的图片链接用相对路径，这样只要有markdown文件和图片，很容易切换到别的环境建站。
> 
> 可以离线写，github用来保存每次变更，相当于备份了。
> 
> 编辑markdown方便，vscode+extensions，预览、插图、版本管理等等在windows和mac下可以有一样的体验。
> 
> 在github pages上也可以很容易就实现：评论，统计，搜索，打赏，广告等等。

一句话，实在找不到不用github pages的理由，还免费 :+1:

### 大概的流程

1. 安装[Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/install/)
2. 安装[Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/install-win10)，指定WSL2为默认。
3. 选择一套jekyll模板，我用的是jekyll-theme-chirpy，可以通过该链接来使用[chirpy模板](https://github.com/cotes2020/chirpy-starter)。注意第一次需要参考链接里面的步骤来部署到github pages。
4. 参考下面的命令用docker启动jekyll。

#### 相关命令

* 在本地进入jekyll项目的目录后，下面的命令会启动jekyll serve，成功后就可以通过4000端口访问本地的开启的web服务：

```
docker run --rm --label=jekyll --volume="%CD%:/srv/jekyll" -p 4000:4000 jekyll/jekyll:4.2.0 jekyll serve
```

类似的，可以用 jekyll build，或者给serve添加watch, trace等参数。

* 如果环境里面有错误，可以通过下面的命令进入jekyll docker查看。

```
docker run --rm --label=jekyll --volume="%CD%:/srv/jekyll" -it jekyll/jekyll:4.2.0 /bin/bash
```

#### 常见问题

1. 提示 /srv/jekyll 目录没有权限：

```
jekyll 3.9.0 | Error:  Operation not permitted @ apply2files - /srv/jekyll
/usr/local/lib/ruby/2.7.0/fileutils.rb:1346:in `chmod': Operation not permitted @ apply2files - /srv/jekyll (Errno::EPERM)
```

需要进入到jekyll docker中将/srv/jekyll目录改为jekyll用户：

```
docker run --rm --label=jekyll --volume="%CD%:/srv/jekyll" -it jekyll/jekyll:4.2.0 /bin/bash
chown -R jekyll:jekyll /srv/jekyll
```

