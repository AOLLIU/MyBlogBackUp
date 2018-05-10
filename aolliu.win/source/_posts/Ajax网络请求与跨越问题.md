---
title: Ajax网络请求与跨越问题
date: 2015-8-20 21:47:42
tags:
    - web开发
---

## 基础知识:
AJAX:异步的JavaScript和XML,是一种在无需重新加载整个界面的情况下能够更新部分网页的技术

HTTP请求:
- 1.建立TCP链接
- 2.Web浏览器向Web服务器发送请求命令
- 3.Web浏览器发送请求头信息
- 4.Web服务器应答
- 5.Web服务器发送应答头信息
- 6.Web服务器向浏览器发送数据
- 7.Web服务器关闭TCP链接

<!--more-->

## XHR发送一个网络请求
- open(method,url,async)//第三个表示请求是否异步,默认true
- send(string) //发送请求
- responseText:获得字符串形式的响应数据
- responseXML:获得xml形式的响应数据
- status和statusText:以数字和文本形式返回HTTP状态码
- getAllResponseHeader():获取所有的响应报头
- getResponseHeader():查询响应中的某个字段的值
- readyState属性;通过监听属性变化获取请求的信息
  - 0:请求未初始化,open还没有调用
  - 1:服务器链接已建立,open已经调用了
  - 2.请求已接受,也就是接收到了头信息
  - 3.请求处理中,接受到请求主体了
  - 4.请求已经完成,响应就绪,响应完成了

```
var request = new XMLHttpRequest();
request.open("Post","get.php",true);
request.setRequestHeader("Content-type","application/x-www.form-urlencoded");
request.send();//发送请求

request.onreadystatechange = function(){
  if(request.readyState === 4 && request.status === 200){
    //做事情 request.responseText
  }
}
```

## jQuery
- 一般发送网络请求都是用第三方库来做,简单易懂,这里不再说了

## 跨域
---
一个域名地址的组成


![Snip20161121_25.png](http://upload-images.jianshu.io/upload_images/1494773-3f64433ab5021eae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 当协议,子域名,主域名,端口号中的任意一个不相同的时候,就算做在不同的域
- 不同域之间相互请求资源就算作是跨域
- JavaScript出于安全问题的考虑,不允许跨域调用其他页面的对象
- JavaScript同源策略的限制,a.com域名下的js无法操作b.com域名下的对象

### 解决方法
#### 1.通过代理
- 通过在同域名的web服务器端创建一个代理:
  - 北京服务器 www.beijing.com
  - 广州服务器 www.shanghai.com
- 这里需要后台做些事情,比如北京的web的后台,来调用上海服务器的服务,然后再把响应结果返回给前端,这样北京的前端调用北京同域名的服务就和调用广州的服务效果一样了,当然这需要后台来做些事情.

#### 2.jsonP
- 注意:只能解决主流浏览器的get请求的跨域问题
- 需要服务器和前端合作实现
```
$(function(){
  $(#search).click(function(){
    $.ajax({
      type:"GET",
      url:"http://....",
      dataType:"jsonp",  //这里讲返回类型改为jsonp,服务器需要改造返回的数据为jsonp格式
      jsonp:"callBack", //增加一个属性,这里前端参数是什么后台就获取什么
      success:function(data){
        if(data.success){
          alet("成功");
        }else{
          alet("错误");
        }
      }
      error:function(jqXHR){
          alet("错误");
      }
    })
  })
})
```

### XHR2
- H5提供的XMLHttpRequest Level2已经实现了跨域访问以及其他的一些新功能,但是IE10以下的版本不支持,如果忽略IE浏览器可以使用该方法,这样前端无需改动,只需要在服务器做一些小小改造就可以了,更多知识看官方文档

```
header('Access-Control-Allow-Origin:*');
header('Access-Control-Allow-Methods:POST,GET');
```
















