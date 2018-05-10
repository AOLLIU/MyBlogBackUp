---
title: React Native学习笔记九
date: 2016-08-20 15:38:34
tags:
    - 跨平台开发
---

## 一.延展操作符(...)

RN是面向组件开发的,我们会在该组件的defaultProps中公开一些属性方便外界进行数据传递,如果只有几个props直接传递即可,但是当传递大量的props时会显得很乱,容易出错,那么`...`(延展操作符,取出参数对象的所有可遍历属性)来进行传递是一种很好的选择.

- 1.常规写法
```
<HomeNav
  title = "首页"
  subTitle = "子标题"
  intro = "首页简介"
  detail = "首页详情介绍"
/>
```

<!--more-->

- 2.使用延展符操作符写法

```
var params = {
  title = "首页",
  subTitle = "子标题",
  intro = "首页简介",
  detail = "首页详情介绍",
};

return (
  <HomeNav {...params}/>
);
```

## 二.真机调试

### iOS:
- 1.让运行的手机和电脑的WIFI在同一个局域网,获取到电脑的IP地址
- 2.打开Xcode,在AppDelegate中添加如下代码:

```
NSURL *jsCodeLocation;
    
    [RCTBundleURLProvider sharedSettings].jsLocation  = @"192.168.20.123";//ip 地址
    
    jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index.ios" fallbackResource:nil];
    
    
    RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation
                                                        moduleName:@"Project"
                                                 initialProperties:nil
                                                     launchOptions:launchOptions];
    rootView.backgroundColor = [[UIColor alloc] initWithRed:1.0f green:1.0f blue:1.0f alpha:1];
    self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
    UIViewController *rootViewController = [UIViewController new];
    rootViewController.view = rootView;
    self.window.rootViewController = rootViewController;
    [self.window makeKeyAndVisible];
    return YES;

```

- 3.run即可

### Android:

- 1.Android 5.0 以上版本,可以直接通过adb reverse运行调试;
- 2.在Android设备上打开USB debugging并连接上电脑启动调试;
- 3.打开终端,命令`react-native run-android`编译运行项目;
- 4.保证手机电脑在同一局域网(摇一摇 - Dev Setting - Debug server host for device - 输入当前电脑ip地址 = 点击Reload JS)


## 三.ref深入

用ref可以获取到真实的Dom节点,其实ref还有更加高级的用法,他的属性值不仅仅可以是string,也可以是function

```
render(){
  return <View ref={(e) => this._view = e}/> //将组件View作为参数赋值给了this._view
}

componentDidMount(){
  this._view.style = {backgroundColor:'red',width:100,height:200}
}
```
ref在RN中左右相当于CSS中的选择器,我们可以任意去拿任何组件,从而获取他的属性和方法做相应的操作.


## 四.优化界面切换卡顿

1.如果使用NavigatorIOS来进行界面切换很流畅,因为动画是在主线程上运行的,但如果使用Navigator有时会卡顿,因为他是跑在js线程上的,切换动画会使JS线程出现严重的掉帧

2.在RN中有一个InteractionManager组件,其作用让一些js操作在过渡动画完成之后执行,保证了动画的流程性,这是典型的牺牲时间换空间从而保证了帧数的高复用,比如界面的跳转:

```
dealWithGoView(obj){
  let detailUrl = obj.detailUrl;
  const {navigator} = this.props;
  InteractionManager.runAfterInterations(() => {
    navigator.push({
      component:GoDetailView,
      name:'Login',
      props:{detailUrl}
    });
  });
}
```

## 五.让组件做到局部刷新

1.利用RN的状态机机制,可以通过`this.setState`来控制界面刷新,但是一定会触发render方法,那如何保证不调用render方法还能做到局部刷新呢?

2.那就是通过`setNativeProps`,不适用state和props,直接修改RN自带的组件,比如View,Text,Image...而且可以做到不触发RN组件生命周期中的方法.

```
var HomePage = React.creatClass({
  setNativeProps(nativeProps){
    this._view.setNativeProps({
      width:50,
      height:50,
      backgroundColor:'yellow'
    });
  },
  
  render(){
    return(
      <View ref={(e) => this._view = e}/>
    )
  }
})
```