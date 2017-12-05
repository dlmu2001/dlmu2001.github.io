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
图1是一个简单的http get请求，浏览器或者客户端想要获取
某个资源，就发送一个请求，在请求参数里面包含了资源的位置，
服务器端收到请求，将对应的响应数据发送给浏览器或者客户端。
<br>

![post请求](/assets/media/http_post.png)
<div align= "center">
图2 post请求
</div>
<br>
图2是一个http post请求，可以用来提交信息给服务器，服务器收到
了数据，处理数据（比如持久化），然后将结果返回给客户端。

<br>
* http协议是应用层协议，通常运行在tcp协议之上

* http协议被广泛应用于传输各种类型的数据

* http协议是一个请求--响应模型

* http协议是无状态的

&emsp;&emsp;http协议是无状态的，它没有事务处理能力，一个请求
响应的过程就完成全部的生命周期。当服务器端希望在两个或多个请
求之间实现状态的时候，要自己来处理（比如可以使用cookie或者url
的query携带状态信息).

* 明文传输

&emsp;&emsp;http协议本身，包括头和体部是通过明文传输的。引入
https可以对传输的协议数据进行加密。

# 2. http和tcp的关系

http是应用层协议，tcp是传输层协议，http通常运行与tcp协议之上。
<br>

如图3是http端到端的数据流,可以看到http协议数据放在tcp协议体部传递。

![http数据](http://upload-images.jianshu.io/upload_images/2824193-dbaa6fa64a03922e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

<div align="center"> 图3. http端到端数据流</div>

图4是wireshark抓到的一个服务器端http请求的tcp流，可以看到客户端首先进行tcp三次握手（连接），成功以后
客户端发送http请求数据，服务器端收到Http请求，利用tcp发送响应数据，然后服务器端断开连接。

![一次http请求的tcp流](/assets/media/http_tcp.png)

  <div align="center"> 图4. 一次http请求的tcp流 </div>

在http协议的客户端具体实现里面，一般会调用socket来连接服务器，发送和接受数据，所以存在三个超时可能（
连接超时，发送超时，接受超时）。

# 3. 请求与响应

# 4. 缓存

# 5. cookie

# 6. https

# 7. 其它

## 7.1 永久连接

## 7.2 内容协商

## 7.3 断点续传

## 7.4 重定向

## 7.5 压缩

## 7.6 chunk编码

## 7.7 http代理

## 7.8 HTTP 2.0

# 参考
* 1.[rfc2616](http://www.ietf.org/rfc/rfc2616.txt)

* 2.[rfc6265:HTTP State Management Mechanism](https://tools.ietf.org/search/rfc6265)

* 3.[HTTP2.0](https://datatracker.ietf.org/doc/rfc7540/)
