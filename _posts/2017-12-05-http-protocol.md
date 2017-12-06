---
layout: post
title: 写给应用开发人员的http协议介绍 
categories: [Mobile]
tags: [基础]
description: team内部培训，面向应用开发者的http协议介绍 
---

tomorrow.cyz@gmail.com 

# 1. http协议概览

先通过两个协议包认识下http协议

![请求响应模型](/assets/media/http_req_res.png)
<div align = "center">
图1 请求响应模型
</div>
<br>
图1是一个简单的http get请求，UA(浏览器或者客户端)想要获取
某个资源，就发送一个请求，在请求参数里面包含了资源的位置，
服务器端收到请求，将对应的响应数据发送给浏览器或者客户端。
<br>

![post请求](/assets/media/http_post.png)
<div align= "center">
图2 post请求
</div>
<br>
图2是一个http post请求，可以用来提交信息给服务器，服务器收到
了数据，处理数据（比如持久化），然后将结果返回给UA。

<br>
* http协议是应用层协议，通常运行在tcp协议之上

* http协议被广泛应用于传输各种类型的数据

* http协议是一个请求--响应模型

* http协议是无状态的

&emsp;&emsp;http协议是无状态的，它没有事务处理能力，一个请求
响应的过程就完成全部的生命周期。当服务器端希望在两个或多个请
求之间实现状态的时候，要自己来处理（比如可以使用cookie或者url
的query携带状态信息).

在移动应用里面，http协议是前端和后端的桥梁。

* 明文传输

&emsp;&emsp;http协议本身，包括头和体部是通过明文传输的。引入
https可以对传输的协议数据进行加密。

# 2. http和tcp的关系

http是应用层协议，tcp是传输层协议，http通常运行与tcp协议之上。
<br>

如图3是http端到端的数据流,可以看到http协议数据放在tcp协议体部传递。

![http数据](http://upload-images.jianshu.io/upload_images/2824193-dbaa6fa64a03922e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

<div align="center"> 图3. http端到端数据流</div>

图4是wireshark抓到的一个服务器端http请求的tcp流，可以看到UA首先进行tcp三次握手（连接），成功以后
UA发送http请求数据，服务器端收到Http请求，利用tcp发送响应数据，然后UA断开tcp连接。

![一次http请求的tcp流](/assets/media/http_tcp.png)

  <div align="center"> 图4. 一次http请求的tcp流 </div>

在http协议的客户端具体实现里面，一般会调用socket来连接服务器，发送和接受数据，所以存在三个超时可能（
连接超时，发送超时，接收超时）。

通常一次http请求的时间会包含tcp三次握手的时间，为了优化掉这个时间，HTTP1.1引入了永久连接的概念(keep-alive)。

这也是很多应用网络优化里面经常提到的“HTTP常连接”，其实HTTP本身是没有连接的，这个说法不严谨。

# 3. 请求与响应

## 3.1 请求

http请求从UA向server发起，包含方法，资源标识，请求头和请求体部，请求体部是可选的。

![http请求](/assets/media/http_request.png)
  <div align="center">图5. http请求</div>
<br>
请求头部和体部之间的分隔是两个CRLF（\r\n)

### 3.1.1 请求方法

请求方法包括GET,POST,HEAD,PUT,DELETE,TRACE,CONNECT,OPTIONS，这些方法都有一定的语法意义，但是
具体的行为更多地依赖于服务器的实现。

现在主流的API是RESTFul风格的，针对CRUD，分别对应post,get,put和delete方法。

### 3.1.2 内容协商

同一个资源可能提供多种形式的版本，比如现在的APP，早期的接口都是JSON格式了，后期可能实现了Protocolbuf，
就会有两种格式的版本，对低版本的APP的请求提供JSON格式，对高版本的提供Protocolbuf格式的。这个就是在客户
端和服务端协商的结果。

内容协商的原理，通常是UA告诉服务端自己的偏好，服务端根据UA的偏好选择合适的版本给UA。

常见的内容协商相关的请求/响应头部有:

    请求头字段	    说明	                      响应头字段
    Accept	        告知服务器发送何种媒体类型	Content-Type
    Accept-Language	告知服务器发送何种语言	    Content-Language
    Accept-Charset	告知服务器发送何种字符集	Content-Type
    Accept-Encoding	告知服务器采用何种压缩方式	Content-Encoding

如下是内容协商的一个请求头的例子

    Accept:*/*
    Accept-Encoding:gzip,deflate,sdch
    Accept-Language:zh-CN,en-US;q=0.8,en;q=0.6

对应的响应头部

    Content-Type: text/javascript
    Content-Encoding: gzip
    Content-Language: en-US


另外，User-Agent这个请求头部在移动互联网里被广泛使用，扩展了越来越多的信息，比如屏幕尺寸，分辨率，厂家信
息，os版本等，服务器端会根据User-Agent产生不同的响应，这也可以看做内容协商的另一种形式。

考虑下这个情况，如果服务端提供了JSON和ProtocolBuf两个版本给APP，低版本的用JSON，高版本用PB,通常为了提高
服务端的负载，会在后端集群前面加代理，比如vanish，这个时候vanish要收到一个请求，怎么来确定缓存的策略呢？
通用的http代理服务器也有同样的问题。

这个时候在响应头部引入了一个Vary头部。
假设我们是通过Accept头部进行JSON和PB的协商的，那么后端可以加

    Vary:Accept

用来告诉缓存服务器，根据不同的Accept头部，缓存和筛选相应的响应版本。

## 3.2 响应

http的响应由状态行，响应头部和响应体部组成

    Response      = Status-Line               
                       *(( general-header        
                        | response-header        
                        | entity-header ) CRLF)  
                       CRLF
                       [ message-body ]          


响应行由版本，状态码和描述组成。
    
    Status-Line = HTTP-Version SP Status-Code SP Reason-Phrase CRLF

### 3.2.1 状态码 

http将状态码分为5类:

     - 1xx: Informational - Request received, continuing process

     - 2xx: Success - The action was successfully received,
       understood, and accepted

     - 3xx: Redirection - Further action must be taken in order to
       complete the request

     - 4xx: Client Error - The request contains bad syntax or cannot
       be fulfilled

     - 5xx: Server Error - The server failed to fulfill an apparently
       valid request


1xx的状态码现在已经用的很少。用的比较多的分别是

    | "200"   OK
    | "301"   Moved Permanently
    | "302"   Found
    | "304"   Not Modified
    | "307"   Temporary Redirect
    | "400"   Bad Request
    | "401"   Unauthorized
    | "403"   Forbidden
    | "404"   Not Found
    | "407"   Proxy Authentication Required
    | "500"   Internal Server Error
    | "502"   Bad Gateway
    | "503"   Service Unavailable
    | "504"   Gateway Time-out

### 3.2.2 压缩和传输编码

通常指的http压缩是内容编码的一种，通过gzip/compress等方式，将http体部数据压缩后传输，
可以达到减少数据传输的目的。

http压缩可以通过内容协商的方式进行协商(Accept-Encoding/Content-Encoding)。

纯文本有很高的压缩率。所以一般对html/css/javascipt，会全站设置使用gzip。

php有个开源项目minify，就是把一个页面的css和js合并到一个文件，然后再进行压缩。

相对于内容编码，传输编码是另外一个概念。传输编码不是为了压缩某一个资源，而是为了保证
资源被“安全传输”。常见的传输编码就是chunked。

早期http1.0是通过服务器端关闭tcp连接来确定http体部结束的，http1.1引入了content-length
头部，当收到体部数据达到content-length长度的时候，就可以判定体部结束了。但是有些情况下，
服务器开启传输的时候，不好确定数据的长度，比如某些情况下的流式数据。这个时候就引入了chunk,
将数据分成一块块，每块包含块信息（块长度，是不是最后一块等），如果最后一块，收到了那块长度
的数据，体部结束。这些块信息包含在体部里面，而不是头部。

chunk一个很经典的应用，比如一个动态页面，包含头部，边栏，尾部，正文，每个区域可能都要根据数
据库动态生成，响应时间会比较长，这个时候一种方法是ajax，后端提供多个接口，分别请求，再渲染，
还有一种方式，可以分成多个chunk来返回,比ajax还省掉了http请求流程。

# 4. 缓存

# 5. cookie

# 6. https

# 7. 其它

## 7.1 永久连接

## 7.3 断点续传

## 7.5 压缩

## 7.6 chunk编码

## 7.7 http代理

## 7.8 HTTP 2.0

# 参考
* 1.[rfc2616](http://www.ietf.org/rfc/rfc2616.txt)

* 2.[rfc6265:HTTP State Management Mechanism](https://tools.ietf.org/search/rfc6265)

* 3.[HTTP2.0](https://datatracker.ietf.org/doc/rfc7540/)
