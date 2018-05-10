---
title: React Native学习笔记四
date: 2016-08-06 21:12:57
tags:
    - 跨平台开发
---

## 网络请求
 - 在开发中,从网络上加载数据是重点难点,尤其是在做细节优化方面,在React Native中通常使用Ajax请求从服务器获取数据,然后在
componenrDidMount方法中创建Ajax请求,等到请求成功,再调用this.setState方法重新渲染UI.

## 什么是fetch
-  fetch目前还不是W3C规范.与Ajax不同的是,他的API不是事件机制,而是采用了目前流行的Promise方法处理

[fetch学习资料](https://segmentfault.com/a/1190000003810652)

<!--more-->

## 使用fetch
- React Native提供了和web标准一致的Fetch API，用于满足开发者访问网络的需求。

## 发起网络请求

- 要从任意地址获取内容的话，只需简单地将网址作为参数传递给fetch方法即可:`fetch('https://www.baidu.com')`
- Fetch还有可选的第二个参数，可以用来定制HTTP请求一些参数。你可以指定header参数，或是指定使用POST方法，又或是提交数据等等;

```
fetch('http://www.baidu.com',{
  method: 'POST',
  header: {
    'Accept':'application/json',
    'Content-Type':'application/json',
  },
  body:JSON.stringify({
    firstParam:'yourValue',
    SecondParam:'yourOtherValue',
  })
})
```
- 译注：如果你的服务器无法识别上面POST的数据格式，那么可以尝试传统的form格式，示例如下：

```
fetch('http://www.baidu.com',{
  method:'POST',
  header:{
    'Content-Type':'application/x-www-form-urlencoded',
  },
  body:'key1=value1&key2=value2'
})
```

## 处理服务器的响应数据
- 网络请求天然是一种异步操作,fetch方法会返回一个Promise,这中模式可以简化异步风格的代码.

```
getMoviesFromApiAsync(){
  return fetch('http://http://facebook.github.io/react-native/movies.json')
  .then((response) => response.json())
  .then((responseJson) => {
    resultData.shopData = responseJson.data;
    this.setState({
      dataSource:this.getDataSource(responseJson.data),
    });
    <!--return responseJson.movies;-->
  })
  .catch((error) => {
    console.error(error);
    resultData.shopData = undifined;
    this.setState({
      dataSource:this.getDataSource([]),
      isLoading:false
    });
  });
  .done();
}
```

- 也可以在React Native应用中使用async/await 语法:

```
//注意在这个方法前面有async关键字
 async getMoviesFromApi(){
   try{
      //注意这里的await语句,器所在的函数必须有async关键字声明
      let response = await fetch('http://http://facebook.github.io/react-native/movies.json');
      let responseJson = await response.json();
      return responseJson.movies;
   }catch(error){
      console.error(error)
   }
 }
```
- 在这里catch住fetch可能抛出的异常,不然出错的时候看不到任何的提示.


### 其他的网络库
- React Native中已经内置了XMLHttpRequest API(也就是俗称的ajax)。一些基于XMLHttpRequest封装的第三方库也可以使用，例如frisbee或是axios等。但注意不能使用jQuery，因为jQuery中还使用了很多浏览器中才有而RN中没有的东西（所以也不是所有web中的ajax库都可以直接使用）。

```
var request = new XMLHttpRequest();
request.onreadystatechange = (e) => {
  if (request.readyState !== 4) {
    return;
  }

  if (request.status === 200) {
    console.log('success', request.responseText);
  } else {
    console.warn('error');
  }
};

request.open('GET', 'https://mywebsite.com/endpoint/');
request.send();
```
- 注意:安全机制与网页环境有所不同,在应用中你可以访问任何网站,没有跨域的限制.

### WebSocket支持
- React Native还支持WebSocket,这种协议可以在当个TCP连接上提供双工的通信信道.

```
  var ws = new WebSocket('ws://host.com/path');
  ws.onopen = () => {//打开一个链接
    ws.send('something');//发送一条消息
};
  ws.onmessage = (e) => {//接受到了一条消息
    console.log(e.data);
};
  ws.onerror = (e) => {//发生了一个错误
    console.log(e.message);
};
  ws.onclose = (e) => {//连接被关闭
    console.log(e.code,e.reason);
};
```










