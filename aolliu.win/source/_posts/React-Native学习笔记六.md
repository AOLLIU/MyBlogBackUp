---
title: React Native学习笔记六
date: 2016-08-12 21:48:06
tags:
    - 跨平台开发
---

- 在oc中生命周期:
oc:loadview  viewdidload  viewwillappear 等等

- React Native中的组件也是有生命周期的,可以根据下图的执行顺序在对应的函数中做对应的操作,可以大致分为3个阶段


![Snip20161219_1.png](http://upload-images.jianshu.io/upload_images/1494773-4f044db35dd228e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!--more-->

# 一.实例化阶段:
## 1. `getDefaultProps` : 创建展示
### 作用:
- 1.该函数用户初始化一些默认的属性,通常会将固定的内容放到这个函数中进行初始化和赋值
- 2.上下界面传值

### 描述:
在组件中,可以使用this.props获取在这里初始化他的属性,由于组件初始化后,再次使用该组件不会调用这个函数,所以组件自己不可以自己修改props(只读),只可以由其他组件外部调用他时在外部修改

## 2.`getinitialState`: 设置可改变的值
### 作用:
- 该函数是用于对组件的一些状态进行初始化;

### 描述:
一旦调用了this.setState方法.组件一定会调用render方法,对组件进行再次渲染,不过React框架会自动根据DOM的状态来判断是否需要真正的渲染,局部刷新
- 由于该函数不同于getDefaultProps,在以后的工程中,会再次调用,所以可以将控制控件的状态的一些变量放到这里初始化,如控件上显示的文字,可以通过this.state来获取,通过this.setState来修改state的值,比如:

```
changeValue(){
  this.setState({
	age:this.state.age+1
 });
}
```
## 3.`componentWillMount`:相当于viewWillAppear

## 4.`render`:
### 描述:
- render是一个组件中必须有的方法,本质是一个函数,并返回JSX或其他组件来构成DOM,和Android的XML布局类似.
- 只能返回一个顶级的元素
- 在render函数中,只可以通过this.state和this.props来访问在之前函数中初始化的数据值

## 5.`componentDidMount`
### 描述:
-  在调用了render方法后,组件加载成功并被成功渲染出来以后,所要执行的后续操作,一般会这个函数中处理网络请求等加载数据的操作
因为UI已经成功被渲染出来,所以放在这个函数中进行请求操作,不会出现UI上的错误

```
componentDidMout(){
	this.fetchData();
},
fetchData(){
	var me = this;
	fetch(me.props.api)
	.then(( response) => response.json())
	.then(( responseData) => {
		me.setState({
			dataSource: responseData.data
		});
	})
	.done(function(){
		me.start();
	});
}
```

# 二.存在阶段:循环  运行中

# 三.销毁阶段
 
 
# 总结:
### 一:this.state
- 开发中会与用户交互,React的一个创新就是将组件看成一个状态机,一开始有一个初始状态,然后交互,导致状态改变,从而触发重新渲染UI,例如如果用户点击组件,导致状态改变`this.setState`方法修改状态值,每次修改调用`render`方法再次渲染组件
- 注意:this.props表示那些一旦定义就不会再改变的特性,而this.state会随着与用户的交互发生改变
 
### 二.获取真是的DOM节点
- 在React Native中,组件并不是真是的Dom节点,而是存在于内存之中的一种数据结构,叫做虚拟Dom(virtual Dom)
- 只有当它插入文档中以后才会变成真正的Dom
- react中所有的Dom操作都是现在虚拟的Dom上发生,然后在将实际变动的部分反映到真实的Dom上,这种算法叫做Dom diff它可以极大的提高应用的性能
- 从组件中获取真实的Dom,需要`ref`属性,比如点击按钮让其中的输入框获得焦点
  - 1.先给组件绑定一个`ref`属性
  ` <TextInput ref='nameText'/>`  
  - 2.点击事件,获取真实Dom,获取焦点
  ```
  buttonClick(){
    this.refs.nameText.focus();
  }
  ```
  - 如果想拿到输入框中的内容,就必须获取真实的Dom,虚拟的Dom是拿不到用户输入的





 
 



