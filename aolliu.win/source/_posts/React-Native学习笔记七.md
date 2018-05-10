---
title: React Native学习笔记七
date: 2016-08-15 20:38:10
tags:
    - 跨平台开发
---

## 简介:
- ES5,ES6都是对ECMAScript规范的补充,ES5已经大规模的使用,ES6目前可能在个别平台存在浏览器兼容问题.

## 他们之间的区别:
### 1.创建组件:
- ES5:

```
//创建组件类 ES5
var LVFirstDemoFirstItem = React.createClass({

    render() {//返回对应创建的控件 JSX语法 界面搭建
      return (
        <View>
            <Text>我是第一个应用ES5创建的组件</Text>
        </View>
   );
  }
})
```

<!--more-->

- ES6:

```
//创建组件类 ES6
class LVFirstDemoSecondItem extends React.Component {
    //相当于viewdidload
    render() {//返回对应创建的控件 JSX语法 界面搭建
        return (
            //只能返回一个顶级的view 封装控件
            <View>
                <Text>我是第一个应用ES6创建的组件</Text>
            </View>
        );
    }
}
```

### 2.组件的属性props所在位置不同
在ES6中,props为属性:`defaultProps`可以标识static定义在class内,也可以定义在class外),而在ES5中,其为方法:getDefaultProps:function(){return{name:value}};


- ES5:

```
//创建组件类 ES5
var LVFirstDemoFirstItem = React.createClass({

    getDefaultProps(){
        return{
            name:'liuwei',
        }
    },
    propTypes(){
      name:React.PropTypes.string
    },//属性校验器,说明name必须是string
    
    render() {//返回对应创建的控件 JSX语法 界面搭建
      return (
        <View>
            <Text>我是第一个应用ES5创建的组件</Text>
            {/*变量用大括号标示*/}
            <Text>my name is {this.props.name}</Text>
        </View>
   );
  }
})
```


- ES6:

```
//创建组件类 ES6
class LVFirstDemoSecondItem extends React.Component {
    //相当于viewdidload
    render() {//不能再定义组件的时候定义一个属性
        return (
            <View>
                <Text>我是第一个应用ES6创建的组件</Text>
                {/*变量用大括号标示*/}
                <Text>my name is {this.props.name}</Text>
            </View>
        );//开头的花括号一定和小括号隔开一个空格,否则识别不出来
    }
}

LVFirstDemoSecondItem.defaultProps={name:'liuwei'};//默认属性值
LVFirstDemoSecondItem.prototype={name:React.PropTypes.string};//属性校验器,说明name必须是string
```

### 3.组件的状态state

- ES5:

```
//创建组件类 ES5
var LVFirstDemoFirstItem = React.createClass({

    getDefaultProps(){
        return{
            name:'liuwei',
        }
    },
    propTypes(){
      name:React.PropTypes.string
    },//属性校验器,说明name必须是string

    getInitialState(){
        return{
            boy:false
        }
    },

    render() {//返回对应创建的控件 JSX语法 界面搭建
      return (
        <View>
            <Text>我是第一个应用ES5创建的组件</Text>
            {/*变量用大括号标示*/}
            <Text>my name is {this.props.name}</Text>
        </View>
   );
  }
})
```
- ES6:

```
//创建组件类 ES6
class LVFirstDemoSecondItem extends React.Component {
    //构造
    constructor(props){
        super(props);
         //初始状态
        this.state={boy:false}
    }

    //相当于viewdidload
    render() {//不能再定义组件的时候定义一个属性
        return (
            <View>
                <Text>我是第一个应用ES6创建的组件</Text>
                {/*变量用大括号标示*/}
                <Text>my name is {this.props.name}</Text>
            </View>
        );//开头的花括号一定和小括号隔开一个空格,否则识别不出来
    }
}
```






