---
title: React Native 之 安装配置
date: 2016-08-01 23:36:12
tags:
    - 跨平台开发
---


# 前言:

- 2015年3月React Native for iOS发布,同年9月React Native for Android发布,React Native真正的可以实现了跨平台开发,代码多端复用,实时热部署.
- ReactNative需要安装配置很多依赖的工具，相对比较麻烦。所以这里主要环境的配置问题.

<!--more-->

# 环境配置

## 1.[Homebrew](http://brew.sh/index_zh-cn.html)
### 废话提示:
 - 本人安装时安装失败,按照网上某篇文章进行解决,没有考虑直接按文章在命令行输入命令,但是沙比了,电脑上所有的配置文件都没有了,后来看了下原来命令是删除所有的配置文件然后重新进行安装,拷,这不坑跌吗,由于之前没有进行备份处理,所有的应用都跟刚安装上一样,太坑了,由此希望大家引以为戒,第一,在遇到问题从网上找解决方案时一定要先仔细看下,到底是干什么的,是否真的能解决你遇到的问题,第二,一定要对数据进行备份,苹果自带的Time Machine就是一个很好的做备份的应用
 
- 1.在终端输入命令`brew -v`,查看之前是否安装过.
如果安装过如下显示:

![Snip20161014_5.png](http://upload-images.jianshu.io/upload_images/1494773-56aa0d45e6fd7a30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 1.1,如果之前安装过最好在终端输入命令升级一下`brew update`,因为如果版本低,可能会影响后续软件的安装.
- 2.如果没有安装在终端输入命令进行安装
 `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
- 2.1再在终端输入命令`brew -v`查看如果有了版本号就说明安装成功
 
[HomeBrew官网](http://brew.sh/index_zh-cn.html)


## 2.[Node.js ](https://nodejs.org/zh-cn/)
- 1.Node.js 安装需要首先安装nvm,有两种方式
 - 1)在终端输入命令:
   `$ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.25.2/install.sh | bash`
 - 2)利用Homebrew,在终端输入命令:
    `brew install nvm`

- 2.安装Node.js,有两种方式
 - 1)在终端输入命令:
   `nvm install node && nvm alias default node`
 - 2)下载安装包安装:
   [下载地址](https://nodejs.org/zh-cn/)

- 3.在终端分别输入命令`node -v`和`npm -v`,如果显示下图表示安装成功



![Snip20161014_6.png](http://upload-images.jianshu.io/upload_images/1494773-95d510dee5fa4a65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 3.安装 [watchman](https://facebook.github.io/watchman/docs/install.html) 和 [flow](https://www.flowtype.org/)
- 1.安装watchman,在终端输入`brew install watchman`,显示下图安装成功
 

![Snip20161014_7.png](http://upload-images.jianshu.io/upload_images/1494773-fa20fb54068075c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 2.安装flow,在终端输入`brew install flow`,显示下图安装成功
 

![Snip20161014_8.png](http://upload-images.jianshu.io/upload_images/1494773-5ec98c7dfc4dbdef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 ## 4.安装 [React-Native](https://facebook.github.io/react-native/)
 - 终端输入命令`npm install -g react-native-cli`
 如果显示失败显示下图,说明没有权限:


![Snip20161014_9.png](http://upload-images.jianshu.io/upload_images/1494773-c8c8d1f730e99138.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 - 那么在终端输入命令`sudo npm install -g react-native-cli`获取权限安装
 这个过程需要输入密码获取权限,如果显示下图则安装成功


![Snip20161014_10.png](http://upload-images.jianshu.io/upload_images/1494773-c1e71edd34bc1a8f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


相关文章链接:
[React Native: 配置和起步](http://www.liaohuqiu.net/cn/posts/react-native-1/)
[React Native开发环境配置](http://www.jianshu.com/p/cd2a8c5ee1c7)
