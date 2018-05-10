---
title: React Native学习笔记二
date: 2016-08-03 22:17:00
tags:
    - 跨平台开发
---

## Props（属性）
- 大多数组件在创建时就可以使用各种参数来进行定制。用于定制的这些参数就称为props（属性）。

- 以常见的基础组件Image为例，在创建一个图片时，可以传入一个名为source的prop来指定要显示的图片的地址，以及使用名为style的prop来控制其尺寸。

- 请注意{pic}外围有一层括号，我们需要用括号来把pic这个变量嵌入到JSX语句中。括号的意思是括号内部为一个js变量或表达式，需要执行后取值。因此我们可以把任意合法的JavaScript表达式通过括号嵌入到JSX语句中。

- 自定义的组件也可以使用props。通过在不同的场景使用不同的属性定制，可以尽量提高自定义组件的复用范畴。只需在render函数中引用this.props，然后按需处理即可。

<!--more-->

```
class Greeting extends Component {
  render() {
    return (
      <Text>Hello {this.props.name}!</Text>
    );
  }
}

class LotsOfGreetings extends Component {
  render() {
    return (
      <View style={{alignItems: 'center'}}>
        <Greeting name='Rexxar' />
        <Greeting name='Jaina' />
        <Greeting name='Valeera' />
      </View>
    );
  }
}
```

## image:加载方式

- 1.从当前项目中加载图片:静态图片资源

```
    // ./ 跳到根目录 ../跳到上一级目录
    <Image source={require('路径')}/>
```

- 2.加载app中的图片

```
<Image source={require('image!图片名')}/>
```

- 3.加载网络图片

```
//网络图片一定要指定图片大小
<Image source={{uri:'地址'}} style={{width:120,heitht:80}/>
```

- 4.从app中加载图片(另一种方式)

```
//网络图片一定要指定图片大小
<Image source={{uri:'图片名'}} style={{width:120,heitht:80}/>
```

- 5.把图片设置为背景

```
<Image source={require('./image/img_03.jpg')}> 
  <Text style={{marginTop:30,backgroundColor:'rgba(0,0,0,0)'}}>我是文字1</Text>
  <Text style={{marginTop:30,backgroundColor:'transparent'}}>我是文字2</Text>
</Image>
```

-6.设置图片的内容模式
 - 1.cover:图片居中显示,没有被拉伸,超出部分被截取
 - 2.contain:容器完全容纳图片,图片等比例拉伸
 - 3.stretch:图片被拉伸适应容器大小,有可能会变形
 
```
    // ./ 跳到根目录 ../跳到上一级目录
    <Image source={require('路径')} style={{width:120,height:150,resizeMode:'cover' }}/>

```

常见属性:
- onLayout(function)当image布局发生改变时调用该方法


## State（状态）
- 我们使用两种数据来控制一个组件：props和state。props是在父组件中指定，而且一经指定，在被指定的组件的生命周期中则不再改变。 对于需要改变的数据，我们需要使用state.
- 一般来说，需要在constructor中初始化state，然后在需要修改时调用setState方法。

```
class Blink extends Component {
  constructor(props) {
    super(props);
    this.state = { showText: true };

    // 每1000毫秒对showText状态做一次取反操作
    setInterval(() => {
      this.setState({ showText: !this.state.showText });
    }, 1000);
  }

  render() {
    // 根据当前showText的值决定是否显示text内容
    let display = this.state.showText ? this.props.text : ' ';
    return (
      <Text>{display}</Text>
    );
  }
}

class BlinkApp extends Component {
  render() {
    return (
      <View>
        <Blink text='I love to blink' />
        <Blink text='Yes blinking is so great' />
        <Blink text='Why did they ever take this out of HTML' />
        <Blink text='Look at me look at me look at me' />
      </View>
    );
  }
}
```



## 处理文本框的输入

- TextInput是一个允许用户输入文本的基本组件,他有一个`onChangeText`属性,此属性接受一个函数,这个函数在文本变化的时候会被调用,还有一个`onSubmitEditing`的属性,在文本被提交后调用.
- 在下面例子中,text保存到state中,因为会随着时间变化

```liuwei
class QRCodePay extends Component{
  constructor(props) {
    super(props);
    this.state = {text: ''};
  }

  render() {
    return (
        <View style={{padding: 10}}>
          <TextInput
              style={{height: 40}}
              placeholder="Type here to translate!"
              onChangeText={(text) => this.setState({text})}
          />
          <Text style={{padding: 10, fontSize: 42}}>
            {this.state.text.split(' ').map((word) => word && '🍕').join(' ')}
          </Text>
        </View>
    );
  }
}
```



















