---
layout: post
title: "【转】在多台电脑上写Octopress博客"
date: 2014-09-26 13:51:01 +0800
comments: true
categories: iOS
tags: [Octopress]
keywords: 多台电脑，octopress
description: 在多台电脑上写Octopress博客
---
原文：http://boboshone.com/blog/2013/06/05/write-octopress-blog-on-multiple-machines/

我将介绍如何在多台电脑上部署Octopress，我假设你已经会在一台电脑上部署Octopress了，如果你还不会或者甚至不知道Octopress是什么，那么就先看看Octopress的官网，然后跟着Document一步一步安装部署就行了。也许你弄坏了本地的Octopress文件，重装了电脑，或和我一样有了一台新电脑:)，下面的内容都会告诉你怎么做。

Octopress原理

我以部署到Github为例，Octopress的git仓库(repository)有两个分支，分别是master和source。master存储的是博客网站本身，而source存储的是生成博客的源文件。master的内容放在根目录的_deploy文件夹内，当你push源文件时会忽略，它使用的是rake deploy命令来更新的。

创建一个本地的Octopress仓库

重新创建一个已经存在的本地Octopress仓库只需要以下几个简单的步骤即可。

克隆(clone)博客到新机器

首先将博客的源文件clone到本地的octopress文件夹内。

```
$ git clone -b source git@github.com:username/username.github.com.git octopress
```

然后将博客文件clone到octopress的_deploy文件夹内。
```
$ cd octopress
$ git clone git@github.com:username/username.github.com.git _deploy
```
然后安装博客，即bundler和需要的rubygem。
```
$ gem install bundler
$ rbenv rehash    # If you use rbenv, rehash to be able to run the bundle command
$ bundle install
$ rake setup_github_pages
```
最后会提示输入仓库的ssh url。
```
Enter the read/write url for your repository
(For example, 'git@github.com:your_username/your_username.github.com)
```
如果你设置了自定义域名，需要将根目录的_config.yml里的url再改成你自己的域名。 这样就建好了一个新的本地仓库了。

更新和推送

当你要在一台电脑写博客或做更改时，首先更新source仓库。更新master并不是必须的，因为你更改源文件之后还是需要rake generate的。
```
$ cd octopress
$ git pull origin source  # update the local source branch
$ cd ./_deploy
$ git pull origin master  # update the local master branch
```
写完博客之后不要忘了推送到remote，下面的步骤在每次更改之后都必须做一遍。
```
$ rake generate
$ git add .
$ git commit -am "Some comment here."
$ git push origin source  # update the remote source branch
$ rake deploy             # update the remote master branch
```

---------------------------------------以下原创----------------------------------------
#####注意，将会遇到错误：
参考：[http://www.lixumin.com/blog/2014/05/07/mac-for-github-octopress/](http://www.lixumin.com/blog/2014/05/07/mac-for-github-octopress/ "link")

同时我也遇到这个错误了，bundle install 一直出错，我是升级ruby到2.1.1才解决了这样的问题。 同时也带来了cocoapods没法用了，于是我
把cocoapods 也升级。一切又是那么美好了。
```
gem update CocoaPods
```
