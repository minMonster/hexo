---
title: NodeJS 读书笔记 第1章
date: 2018-06-19 18:00:00
description: NodeJS
tags: [NodeJS]
categories: NodeJS
---
[Ryan dahl]: https://cn.linkedin.com/in/ryandahl01  "Ryan dahl"

# 异步I/O

在Node中，异步I/O很常见。以读取文件为例。我们可以看到它与前端Ajax调用的方式是极其类似的：

````javascript
var fs = require('fs')
fs.readFile('/path', function(err, file) {
  console.log('读取文件完成')
});
console.log('发起读取文件')

// 发起读取文件
// 读取文件完成
````

这里'发起读取文件'，是在读取文件完成之前执行的，这就是一个典型的异步。

在Node中，绝大多数的操作都以异步的方式进行调用。[Ryan dahl]排除万难，在底层构建了很多异步I/O的API，
从文件读取到网络请求等，均是如此。这样做的意义是我们可以很自然的从语言层面进行并行I/O操作。每个调用之间
无需等待之前的I/O调用结束。在编程模型上可以极大提升效率。

下面的两个文件读取任务的耗时取决于最慢的那个文件读取耗时。

````javascript
fs.readFile('/path', function(err, file) {
  console.log('读取文件1完成')
});

fs.readFile('/path', function(err, file) {
  console.log('读取文件2完成')
});
````

而同步I/O这两个文件的耗时是两个任务耗时之和。

# 事件与回掉函数

## 事件

典型的通过事件来暴露接口

````javascript
var http=require('http');
var querystring=require('querystring');
http.createServer((req,res)=>{
    console.log("request already come");
    var post = "";
    req.on('data',(chunk)=>{
        post += chunk;
    });
    req.on('end',()=>{
        post =  querystring.parse(post);
        console.log('complete complished');
        //返回请求者一个信息
        res.write(post.name);
        res.end();
    });
}).listen(3000);
````

事件编程优点：
* 轻量级
* 松耦合
* 只关注事物点

事件编程缺点：
* 在多个异步场景中，事件与事件之间相互独立，如何协作是一个问题

## 回掉函数

从上面例子可以看到回掉函数是无处不在的。这是因为在javaScript中，我们把函数看作为第一等攻门来对待，可以将函数作为对象传递给方法作为实参进行调用。

回掉函数优点：
* 最好的接收异步调用返回数据的方法

回掉函数缺点：
* 易读性差（代码的编写顺序和执行顺序无关）
* 在流程控制方面，因为穿插了异步方法和回掉函数，与常规的同步方式相比变得不是那么一目了然

# 单线程 

Node保持了javaScript在浏览器中的单线程的特点。

Node 单线程的弱点：
* 无法利用多核 CPU
* 错误会引起整个应用退出，应用的健壮性值得考验
* 大量计算占用 CPU 导致无法继续调用异步 I/O

Node 采用了 Web Workers 相同的思路来解决单线程过程中大计算量的问题： child_process。

# 跨平台

Node 是基于libuv 实现跨平台的

# Node 善于处理 I/O 密集型，以及 CPU 密集型应用

单线程同步编程模型会阻塞I/O导致硬件资源得不到更优的使用。多线程编程模型也因为编程中的死锁，状态同步等问题让开发人员头疼。
Node在两者之间给出了答案：利用单线程，远离多线程死锁，状态同步等问题；利用异步I/O，让单线程远离阻塞，以更好地使用CPU。

# 与遗留系统和平共处

Node 非常适合利用原本的数据源，发挥异步并行的优势，来编写与前端交互的中间层。

比如国内的雪球财经，雪球财经是从原有的JAVA项目中分出来的一个子项目。在这个子项目中没有继续采用JAVA/JSP而是采用Node 来完成WEB端的开发，
使得前端工程师咋在HTTP协议栈的两端能够高效灵活的开发。避免了JAVA繁琐的表达；另一方面，又利用JAVA作为后端接口和中间件，使其具有良好的稳定性，两者互相结合，取长补短。

# 分布式应用

因为非阻塞，反应迅速，而且拥有很好的网络库让网络节点之间实现通信，所以很好扩展，所谓快速、可伸缩嘛……很适合做分布式应用。

# Node 使用者

前后端编程语言环境统一
  * 雅虎Cocktail
  
Node带来的高性能I/O用于实时应用
  * Voxer的实时语音
  * 花瓣网
  * 蘑菇街等公司通过socket.io实现实时通知的能力
  
并行I/O使得使用者可以更高效地利用分布式环境
  * 阿里巴巴的NodeFox
  * eBay的ql.io
  
并行I/O，有效利用稳定接口提升Web渲染能力
* 雪球财经
* LinkedIn的移动版网站
  
云计算平台提供Node支持
* Joyent
* Azure
* 阿里云
* 百度云
    
游戏开发领域
* 网易的pomelo
    
工具类应用
* Browserify 
* grunt 
* gulp
* less
* uglifyjs
* WebPack

