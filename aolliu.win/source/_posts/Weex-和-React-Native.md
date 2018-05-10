---
title: Weex 和 React Native
date: 2016-08-01 21:35:52
tags:
    - 跨平台开发
---


# 写在前面
 - 目前主流的应用大体分成三类：Native App, Web App, Hybrid App. 

![三大主流的应用](http://upload-images.jianshu.io/upload_images/1494773-e34c5f726703fbd9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!--more-->

 - Native App特点:
   - 性能好
   - 完美的用户体验
   - 开发成本高，无法跨平台
   - 升级困难(审核),维护成本高
 - Web App特点:
   - 开发成本低,更新快,版本升级容易,自动升级 
   - 跨平台，Write Once , Run Anywhere
   - 无法调用系统级的API
   - 临时入口，用户留存度低
   - 性能差,体验差,设计受限制
   - 相比Native App，Web App体验中受限于以上5个因素：网络环境，渲染性能，平台特性，受限于浏览器，系统限制。
  
 - Hybrid App特点:
   - Native App 和 Web App 折中的方案，保留了 Native App 和 Web App 的优点。
   - 但是还是性能差。页面渲染效率低，在Webview中绘制界面，实现动画，资源消耗都比较大,受限于技术,网速等因素


![Snip20161028_3.png](http://upload-images.jianshu.io/upload_images/1494773-598d03580d959973.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  
- 为了解决上述问题,一套高效率,高性能的跨平台方案成为了大家热衷的话题,也就有了下面要比较的weex和react native.

# 基本概念
---
## [weex](https://alibaba.github.io/weex/)
- 简介:
weex是阿里巴巴公司与2016年6月开源的一种用于构建移动跨平台的UI框架
- 特点:
  - 1.Lightweight:轻量级,语法简单,易于使用
  - 2.Extendable:可扩展,丰富内置组件,可扩展的API,
  - 3.High Performance:高性能
 - 核心理念:
  - Write Once Run Everywhere
 - 基于JS开发框架:
  - weex基于vue.js

## [React Native](https://facebook.github.io/react-native/)
- 简介:
 - Facebook在2015年3月在F8开发者大会上开源的跨平台UI框架
- 核心理念:LEARN ONCE, WRITE ANYWHERE
- 基于JS开发框架:
 - React Native基于React
 
### 知识拓展:vue.js和React

#### [Vue](https://cn.vuejs.org/):
  - 是一个构建数据驱动的 web 界面的库。Vue.js 的目标是通过尽可能简单的 API 实现响应的数据绑定和组合的视图组件.
  
#### [React](https://facebook.github.io/react/):
  - 基于HTML的前端界面开发正变得越来越复杂，其本质问题基本都可以归结于如何将来自于服务器端或者用户输入的动态数据高效的反映到复杂的用户界面上。而来自Facebook的React框架正是完全面向此问题的一个解决方案，按官网描述，其出发点为：用于开发数据不断变化的大型应用程序。相比传统型的前端开发，React开辟了一个相当另类的途径，实现了前端界面的高效率高性能开发。
  
#### Vue.js和React的异同:
 
[Vue和React的区别](https://cn.vuejs.org/guide/comparison.html)

# Weex和React Native的异同
## 相同点:

 - 都采用Web的开发模式，使用JS开发；
 - 都可以直接在Chrome中调试JS代码；
 - 都支持跨平台的开发；
 - 都可以实现hot reload，边更新代码边查看效果；

## 不同点:

### JS引擎
[什么是JS引擎](https://zh.wikipedia.org/wiki/JavaScript%E5%BC%95%E6%93%8E)

 
### 学习成本
 - 1.环境配置：
   - ReactNative需要按照文档安装配置很多依赖的工具，相对比较麻烦。 weex安装cli之后就可以使用
 - 2.vue vs react:上面已经做过对比
   - react模板JSX学习使用有一定的成本 vue更接近常用的web开发方式，模板就是普通的html，数据绑定使用mustache风格，样式直接使用css


### 社区支持

 - Weex开源较晚，互联网上相关资料还比较少，社区规模较小；
 - React Native社区则比较活跃，可以参考的项目和资料也比较丰富

**一张图:从渲染时间,内存使用,CPU占用,帧率(图形处理器每秒钟能够刷新几次,高的帧率可以得到更流畅、更逼真的动画。每秒钟帧数 （fps） 愈多，所显示的动作就会愈流畅。)**

![各种类型应用对比](http://upload-images.jianshu.io/upload_images/1494773-b0456449011ffa04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 文章推荐
---

[weex vs react-native](https://yq.aliyun.com/articles/57996)
[Weex & ReactNative & JSPatch](http://awhisper.github.io/2016/07/22/Weex-ReactNative-JSPatch/)
[Weex和react native对比](https://xiaomaer.gitbooks.io/compare-weex-to-react-native/content/compare.html)
[对无线电商动态化方案的思考（一）](http://div.io/topic/1478)系列文章




