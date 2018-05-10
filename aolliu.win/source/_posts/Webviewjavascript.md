---
title: Webviewjavascript
date: 2015-3-09 22:36:55
tags:
    - iOS开发
    - OC
---

---
- 来着对官方demo的分析

- 1.首先在控制器的viewdidload方法中

```objc
//     1.初始化一个webview
    UIWebView* webView = [[UIWebView alloc] initWithFrame:self.view.bounds];
    [self.view addSubview:webView];
```

- 2.还是在viewdidload方法中

```objc
//     2.将此webview与WebViewJavascriptBridge关联
    [WebViewJavascriptBridge enableLogging];
    _bridge = [WebViewJavascriptBridge bridgeForWebView:webView];
    [_bridge setWebViewDelegate:self];
```

<!--more-->

- 3.js调用oc方法（可以通过data给oc方法传值，使用responseCallback将值再返回给js）
//    这里注意testObjcCallback这个方法的标示。html那边的命名(前端代码)要跟ios这边相同，才能调到这个方法。当然这个名字可以两边商量着自定义。简单明确即可。

```objc
    [_bridge registerHandler:@"testObjcCallback" handler:^(id data, WVJBResponseCallback responseCallback) {//在这个方法中做我要做的事情
比如点击了web view上的一张图片,js通过data告诉我是那张图片,然后我通过他告诉我的再去我的下载的图片中取出来然后利用mwphotobrowser展示出来
        NSLog(@"testObjcCallback called: %@", data);
        responseCallback(@"Response from testObjcCallback");
    }];
```
- 4.oc调js方法（通过data可以传值，通过response可以接受js那边的返回值）

```objc
    id data = @{ @"greetingFromObjC": @"Hi there, JS!" };
    [_bridge callHandler:@"testJavascriptHandler" data:data responseCallback:^(id response) {
//oc调用js那边的testJavascriptHandler这个方法
//通过data给js传值
        NSLog(@"testJavascriptHandler responded: %@", response);
    }];
```


