---
title: React Native学习笔记一
date: 2016-08-03 20:36:38
tags:
    - 跨平台开发
---

## React Native的第一个应用

### 一.执行命令`react-native init 项目名称`,生成一个工程

 - 注意:第一次创建项目的时候,之后再创建就会很快,这个过程可能很长很长时间,一定要有耐心,可以先做别的,然后随后再回来看看.

### 二.目录结构分析
 - 1.默认生成android和ios两个平台的原生项目;
 - 2.其中index.Android.js和index.ios.js文件为Android和ios的空壳应用文件;
 - 3.此外,node_modules文件夹是为Node.js存放和管理npm包资源,也包含React Native框架文件
 
<!--more-->

### 三.运行工程文件
- 打开Xcode，运行你的第一个React Native创建的iOS应用
- 把React Native创建的应用跑在Android上


### 四.管理React Native库的版本
- 在开发中，会经常的去控制React Native的版本库，得以适配各种条件下的开发，那该如何查看、控制ReactNative的版本呢？

- 1.命令行输入`react-native --version`查看本地的React Native的版本
- 2.命令行输入`npm update -g react-native-cli`更新本地的React Native的版本
- 3.命令行输入`npm info react-native`查询react-native的npm包最新版本
- 4.升级或者降级npm包的版本.`npm install --save react-native@0.26`


### 五.WebStom设置React Native代码提示
- 在WebStom中是没有React Native代码提示的,可以从网上下载xml插件
- 1.[从gitHub上下载xml插件](https://github.com/virtoolswebplayer/ReactNative-LiveTemplate)

- 2.安装
将ReactNative.xml复制到`~/Library/Preferences/WebStorm10/templates`，然后重启 WebStrom
  - 注意如果没有templates可以自己创建一个叫做templates的文件件,然后复制粘贴进去