---
title: React Native学习笔记八
date: 2016-08-16 20:58:15
tags:
    - 跨平台开发
---

在开发中TabBar应用及其广泛,那么在React Native中如何做呢,使用TabBarIOS,可以看到是iOS的,Android是不可以用的,随后封装一个既可以在iOS上运行,又能在安卓上运行的TabBar

## 一.TabBarIOS常见的属性
- 1.View所有的属性都被继承
- 2.`barTinColor` color 设置tabBar的背景颜色
- 3.`tintColor` color 设置tabBar上被选中图标的颜色
- 4.`translucent` bool 设置tabBar栏是不是半透明的效果

<!--more-->

## 二.TabBarIOS.Item常见属性
- 1.`badge` number 在图标右上方显示的红色小点
- 2.`icon` image.propTypes.source Tab按钮自定义图标,如果systemicon属性被定义了,该属性会被忽略
- 3.`onPress` function 当Tab按钮被选中的时候进行调用,你可以设置selected={true}来设置组件被选中
- 4.`selected` bool 该属性标志子页面是否可见,如果是一个空白的内容页,那么一定是忘了选中任何一个页面的标签Tab
- 5.`selectedIcon` image.propTypes.source 设置当Tab按钮选中的时候显示的自定义图标,如果systemicon属性被定义了,该属性会被忽略,如果定义了icon属性,但是当前的selectedIcon属性没有设置,那么该图标会被设置为蓝色
- 6.`style` 设置样式风格,继承view的样式各种风格
- 7.`systemIcon` enum('bookmarks','contacts','downloads','favorites'等),系统预定义的图标,如果是使用这些图标,那么你上面设置的标题,选中的图标都会被覆盖
- 8.`title` string 在tab图标下显示的标题,如果systemicon属性被定义了,该属性会被忽略

**在开发中我们需要多个界面切换,这时候需要一个Navigate来实现各种效果的切换,在ReactNative中有两个组件实现这一效果Navigator和  NavigatorIOS,Navigator适配Android和iOS,而NavigatorIOS是包装了 UIKit中的导航功能,可以实现左滑返回**

## 一.Navigator
 - 如何实现不同页面(场景)的切换呢,他是通过路由对象来分辨不同的场景的,我们采用renderScene方法,根据指定的路由来渲染
 
###  1.常用属性

- 1.initialRoute={name:'home',component:HomeScene}这个指定了默认的页面,也就是启动的组件页面
- 2.`configureScene={()=>{returnNavigator.SceneConfigs.HorizontalSwipeJump;}}`页面之间跳转时候的动画手势,可以看这个目录:`node_modules/react-native/Libraries/CustomComponents/Navigator/NavigatorSceneConfigs.js`,比如:PushFromRight FloatFromRight FloatFromLeft FloatFromBottom FloatFromBottomAndroid HorizontalSwipeJump HorizontalSwipeJumpFromRight VerticalUpSwipeJump VerticalDownSwipeJump 等等
- 3.`renderScene`具体方法:`(route,navigator)=><MySceneComponent title={route.title}navigator={navigator}/>`两个参数中的route包含的是inital的时候传递的name和component,而navigator是一个我们需要用的Navigator的对象;所以当我们拿到route中的component的时候,我们就可以将navigator传递给她,正因为如此,我们的HomeScene才能通过this.props.navigator,拿到路由.
- 4.`initialRouteStack[object]`参数对象数组,这是一个初始化的路由数组进行初始化,如果`initialRoute`属性没有设置的话,就必须设置`initialRouteStack`属性,使用该最后一项作为初始路由,如果`initialRouteStack`属性没有设置的话,会生成值包含`initialRoute`值的数组
- 5.`navigatorBar-node`可选参数,在页面切换中用来提供一个导航栏
- 6.`navigatorBar-object`可选参数,可以从父类导航其中获取导航器对象
- 7.`sceneStyle`样式风格该属性继承View所有样式,用于设置每个页面容器的样式
 

### 2. 常用导航器方法

当获取了导航器对象的引用,我们可以进行调用一下方法实现页面导航功能:

- 1.`getCurrentRoutes()` 该方法进行返回存在的路由列表信息
- 2.`jumpBack()` 该方法进行回退操作,但是该方法不会卸载(删除)当前的页面
- 3.`jumpForward()` 该方法进行跳转到相当于当前页面的下一个页面
- 4.`jumpTo(route)` 根据传入的一个路由信息,跳转到一个指定的页面(该页面不会被卸载和删除)
- 5.`push(route)` 导航切换到一个新的页面中,新的页面进行压入栈,通过`jumpForward()`方法可以回退回去
- 6.`pop()` 当前页面弹出来,跳转到栈中下一个页面,并且会卸载删除当前页面
- 7.`replace(route)` 只用传入的路由的指定页面进行替换掉当前的页面
- 8.`replaceAtIndex(route,index)` 传入路由以及位置索引,使用该路由指定的页面跳转到指定的页面
- 9.`replacePrevious(route)` 传入路由,通过指定路由的页面替换掉前一个页面
- 10.`resetTo(route)` 进行导航到新的界面,并且重置整个路由栈
- 11.`immediatelyResetRouteStack(routeStack)` 该方法通过一个路由页面数组来进行重置路由栈
- 12.`popToRoute(route)` 进行弹出相关页面,跳转到指定路由的页面,弹出来的页面会被卸载删除
- 13.`popToTop()`进行弹出页面,导航到栈中的第一个页面,弹出来的所有页面会被卸载删除


### 3. 默认写法

```
<Navigator
  initialRoute={{
                name:defaultName,
                component:defaultComponent,
                }}
                configureScene={(route)=>{
                    return Navigator.SceneConfigs.HorizontalSwipeJumpFromRight;
                }}
                renderScene={(route,navigator)=>{
                    let component = route.Component;
                    return <Component {...route.params} navigator={navigator}/>
                }}
            />
```

## 二.Navigator.IOS
Navigator.IOS 包装了UIKit的导航功能,可以使用左滑返回上一个界面

### 1.常用方法

- 1.`push(route)` 跳转到一个新的路由
- 2.`pop()`回到上一个页面
- 3.`popN(n)` 回到N页之前,当N=1时,效果和`pop()`一样
- 4.`replace(route)` 替换当前路由,并立即加载新路由的视图
- 5.`replacePrevious(route)` 替换上一页的路由/视图
- 6.`replacePreviousAndPop(route)` 替换上一页的路由/视图并且立即切换到上一页
- 7.`resetTo(route)` 替换最顶级的路由并且回到他,
- 8.`popToRoute(route)` 一直回到某个指定的路由
- 9.`popToTop()` 回到最顶层的路由

### 2.常用属性

- 1.`barTintColor` string 导航条的背景颜色
- 2.`initialRoute`

```
initialRoute{
  component:function,//路由到对应的板块
  title:sting,//标题
  passProps:object,//传递的参数
  backButtonIcon:image.propTypes.source,//返回按钮
  backButtonTitle:string,//返回按钮标题
  leftButtonIcon:image.propTypes.source,
  leftButtonTitle:string,
  onLeftButtonPress:function,
  rightButtonIcon:image.propTypes.source,
  rightButtonTitle:string,
  onRightButtonPress:function,
  wrapperStyle:[object Object],
}
```

NavigatorIOS使用路由对象来包含要渲染的子视图,他们的属性,以及导航条配置,push和任何其他的导航函数的参数都是这样的路由对象,例如
  
  
```
  pushToNewsDetail(rowData){
    this.props.navigator.push({
      title:'新闻详情页',
      component:'LVNewsDetail',
      passProps:{rowData},
    });
  }
```
  
  - 3.`itemWrapperStyle View #style` 导航器中的组件的默认属性,一个常见的用途是设置所有页面的背景颜色
  - 4.`navigationBarHidden` bool 决定导航条是否隐藏
  - 5.`shadowHidden` bool 决定是否隐藏1像素的阴影
  - 6.`tintColor` sting 导航栏上按钮的颜色
  - 7.`titleTextColor` string 导航器标题的文字颜色
  - 8.`translucent` bool 决定导航条是否半透明


