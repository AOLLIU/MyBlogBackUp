---
title: React Native学习笔记五
date: 2016-08-10 20:18:01
tags:
    - 跨平台开发
---

# Navigator
- React Native目前有几个内置的导航器组件，一般来说首推Navigator。它使用纯JavaScript实现了一个导航栈，因此可以跨平台工作，同时也便于定制。

### 场景(Scene)的概念与使用
- 无论是View中包含Text，还是一个排满了图片的ScrollView，渲染各种组件现在来说应该已经得心应手了。这些摆放在一个屏幕中的组件，就共同构成了一个“场景（Scene）”。
- 场景简单来说其实就是一个全屏的React组件。

<!--more-->

```
export default class MyScene extends Component{
  static defaultProps = {
    title:'myScene'
  };
  render() {
    return (
        <View style={{paddingTop:22,paddingBottom:5,justifyContent:'center',alignItems:'center',backgroundColor:'red'}}>
          <Text>this is {this.props.title}</Text>
        </View>
    );
  }
}
```
- 注意组件声明前面的export default关键字。它的意思是导出(export)当前组件，以允许其他组件引入(import)和使用当前组件，就像下面这样

```
import React, { Component } from 'react';
import { AppRegistry } from 'react-native';

// ./MyScene表示的是当前目录下的MyScene.js文件，也就是我们刚刚创建的文件
// 注意即便当前文件和MyScene.js在同一个目录中，"./"两个符号也是不能省略的！
// 但是.js后缀是可以省略的

import MyScene from './MyScene';

class YoDawgApp extends Component {
  render() {
    return (
      <MyScene />
    )
  }
}
AppRegistry.registerComponent('YoDawgApp', () => YoDawgApp);
```
- 现在已经创建了只有单个场景的App。其中的MyScene同时也是一个可复用的Reac组件。

### 使用Navigator
- 首先要做的是渲染一个Navigator组件，然后通过此组件的renderScene属性方法来渲染其他场景。

```
render() {
  return (
    <Navigator
      initialRoute={{ title: 'My Initial Scene', index: 0 }}
      renderScene={(route, navigator) => {
        <MyScene title={route.title} />
      }}
    />
  );
}
```
- 使用导航器经常会碰到“路由(route)”的概念。“路由”抽象自现实生活中的路牌，在RN中专指包含了场景信息的对象。renderScene方法是完全根据路由提供的信息来渲染场景的。你可以在路由中任意自定义参数以区分标记不同的场景.

#### 将场景推入导航栈


