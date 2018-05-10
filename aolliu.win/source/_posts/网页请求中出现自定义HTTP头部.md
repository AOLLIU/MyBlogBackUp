---
title: 网页请求中出现自定义HTTP头部
date: 2015-8-16 20:03:17
tags:
    - web开发

---


#### 近期项目中网页请求中遇到了自定义HTTP头部信息的请求,查了相关资料Ajax资料,不但网页请求需要注意跨域问题,还要考虑如果是自定义头部信息需要和服务器协商解决

- 带预检(Preflighted)的跨域请求需要浏览器在发送真实HTTP请求之前先发送一个OPTIONS的预检请求，检测服务器端是否支持真实请求进行跨域资源访问，真实请求的信息在OPTIONS请求中通过Access-Control-Request-Method Header和Access-Control-Request-Headers Header描述，此外与简单跨域请求一样，浏览器也会添加Origin Header。服务器端接到预检请求后，根据资源权限配置，在响应头中放入Access-Control-Allow-Origin Header、Access-Control-Allow-Methods和Access-Control-Allow-Headers Header，分别表示允许跨域资源请求的域、请求方法和请求头。此外，服务器端还可以加入Access-Control-Max-Age Header，允许浏览器在指定时间内，无需再发送预检请求进行协商，直接用本次协商结果即可。浏览器根据OPTIONS请求返回的结果来决定是否继续发送真实的请求进行跨域资源访问。这个过程对真实请求的调用者来说是透明的。
XMLHttpRequest支持通过withCredentials属性实现在跨域请求携带身份信息(Credential，例如Cookie或者HTTP认证信息)。浏览器将携带Cookie Header的请求发送到服务器端后，如果服务器没有响应Access-Control-Allow-Credentials Header，那么浏览器会忽略掉这次响应。

<!--more-->

- 这里讨论的HTTP请求是指由Ajax XMLHttpRequest对象发起的，所有的CORS HTTP请求头都可由浏览器填充，无需在XMLHttpRequest对象中设置。以下是CORS协议规定的HTTP头，用来进行浏览器发起跨域资源请求时进行协商：
 - 1 Origin。HTTP请求头，任何涉及CORS的请求都必需携带。
 - 2 Access-Control-Request-Method。HTTP请求头，在带预检(Preflighted)的跨域请求中用来表示真实请求的方法。
 - 3 Access-Control-Request-Headers。HTTP请求头，在带预检(Preflighted)的跨域请求中用来表示真实请求的自定义Header列表。
 - 4 Access-Control-Allow-Origin。HTTP响应头，指定服务器端允许进行跨域资源访问的来源域。可以用通配符*表示允许任何域的JavaScript访问资源，但是在响应一个携带身份信息(Credential)的HTTP请求时，Access-Control-Allow-Origin必需指定具体的域，不能用通配符。
 - 5 Access-Control-Allow-Methods。HTTP响应头，指定服务器允许进行跨域资源访问的请求方法列表，一般用在响应预检请求上。
 - 6 Access-Control-Allow-Headers。HTTP响应头，指定服务器允许进行跨域资源访问的请求头列表，一般用在响应预检请求上。
 - 7 Access-Control-Max-Age。HTTP响应头，用在响应预检请求上，表示本次预检响应的有效时间。在此时间内，浏览器都可以根据此次协商结果决定是否有必要直接发送真实请求，而无需再次发送预检请求。
 - 8 Access-Control-Allow-Credentials。HTTP响应头，凡是浏览器请求中携带了身份信息，而响应头中没有返回Access-Control-Allow-Credentials: true的，浏览器都会忽略此次响应。

### 总结：
 - 只要是带自定义header的跨域请求，在发送真实请求前都会先发送OPTIONS请求，浏览器根据OPTIONS请求返回的结果来决定是否继续发送真实的请求进行跨域资源访问。所以复杂请求肯定会两次请求服务端。
 
