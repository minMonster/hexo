---
title: NodeJs-读书笔记-第7章
date: 2018-07-15 13:19:26
tags: [NodeJS]
description: 网络编程
categories: NodeJS
---
# TCP服务
## TCP服务事件分为下面两类
### 服务器事件

对于通过net.createServer()创建的服务器而言，它是一个EventEmitter实例，自定义事件有以下几种：
listening：在调用listen()绑定端口或Domain Socket后触发，简写为server.listen(port, listener)，通过第二个参数传入。
connection：每个客户端套接字连接到服务器时触发，简洁写法为通过net.createServer()，最后一个参数传递。
close：当服务器关闭时触发，在调用server.close()后，服务器将停止接受新的套接字连接，但保持当前存在的连接，等待所有连接断开后，会触发该事件。
error：当服务器发生异常时，将会触发该事件。

### 连接事件
服务器可以同时与多个客户端保持连接，对于每个连接而言是典型的可写可读Stream对象。Stream对象可以用于服务端和客户端之间的通信，既可以通过data事件从一端读取另一端发来的数据，也可以通过write()方法从一端向另一端发送数据。

* data：当一端调用write()发送数据时，另一端会触发data事件，事件传递的数据就是write()发送的数据
* end：当连接中的任意一端发送FIN数据时，将会触发该事件。
* connect：该事件用于客户端，当套接字与服务器连接成功时会触发。
* drain：当任意一端调用write()发送数据时，当前这端触发该事件。
* error：当异常发送
* close：当套接字完全关闭时触发
* timeout：当一定时间后连接不再活跃时，触发该事件通知用户该连接被闲置了。

TCP针对网络中的小数据包有一定优化策略：Nagle算法，当数据达到一定量后才触发。

# UDP服务

UDP称为用户数据包协议，其不是面向连接的服务。Node中UDP只是一个EventEmitter实例，而非Stream的实例，具备以下自定义事件：
* message：当UDP套接字监听网卡端口后，接受消息时触发，触发携带的数据为消息Buffer对象和一个远程地址信息。
* listening：当UDP套接字开始侦听时触发该事件。
* close：调用close()方法时触发该事件，并不再触发message事件。若需再次触发message事件，需要重新绑定。
* error：当异常发生时触发，若不监听直接抛出，使进程退出。

# HTTP服务

Node中http模块继承自tcp服务器(net模块)，它能与多个客户端保持连接，由于其不为每个连接创建线程，保持很低的内存占用，所以能实现高并发。HTTP服务和TCP服务区别在于，在开启keepalive之后，一个TCP会话可以用于多次请求和响应。TCP服务以connection为单位进行服务，HTTP服务以request单位进行服务。http模块是将connection到request的过程进行封装。

http模块将连接所用的套接字的读写抽象为ServerRequest和ServerResponse对象，分别对应请求和响应操作。

* HTTP请求ServerRequest

对于TCP连接的读操作，http模块将其封装为ServerRequest对象。如报头部分req.method、req.url、req.headers，报文体数据部分抽象为一个只读的流对象，若业务逻辑需要读取报文体中的数据，则需要这个数据流结束后才能进行操作。

## ServerRequest事件

* data ：当请求体数据到来时，该事件被触发。该事件提供一个参数 chunk，表示接收到的数据。如果该事件没有被监听，那么请求体将会被抛弃。该事件可能会被调 用多次。 
* end ：当请求体数据传输完成时，该事件被触发，此后将不会再有数据到来。
* close： 用户当前请求结束时，该事件被触发。不同于 end，如果用户强制终止了传输，也还是调用close。

## HTTP响应ServerResponse
HTTP响应封装了底层连接的写操作，可以将其看成一个可写的流对象。
响应报文头部信息方法：res.setHeader()和res.writeHeader()方法，可以多次setHeader进行设置，但必须调用writeHeader写入连接才生效。
报文体部分方法：res.write()和res.end()方法

```javascript
response.end([data], [encoding])：结束响应，告知客户端所有发送已经完成。
writeHead(statusCode, [headers])：向请求的客户端发送响应头。statusCode 是HTTP 状态码。headers是一个类似关联数组的对象，表示响应头的每个属性。
write(data, [encoding])：向请求的客户端发送响应内容。data 是一个 Buffer 或字符串，表示要发送的内容。
```

## HTTP服务端事件

* connection：客户端与服务端建立TCP连接时，触发一次connection事件
* request：建立TCP连接后，http模块底层将数据流中抽象出HTTP请求和HTTP响应，当请求数据发送到服务端，在解析出HTTP请求头后触发该事件；在res.end()后，TCP连接可用于下一次请求。
* close：调用server.close方法停止接收新的连接，已有的连接都断开时触发该事件。
* checkContinue：某些客户端在发送较大数据时，先发送一个头部带有Expect: 100-continue的请求到服务器，服务触发该事件；
* connect：当客户端发起CONNECT请求时触发
* upgrade：当客户端要求升级连接的协议时，需要和服务端协商，客户端会在请求头中带上Updagrade字段
* clientError：连接的客户端发送错误，错误传到服务端此时触发该事件

# HTTP客户端

http模块提供http.request(options, connect)，用于构造HTTP客户端。

HTTP客户端和服务端类似，在ClientRequest对象中，它的事件叫做response,ClientRequest在解析响应报文的时，一解析完响应头就触发response事件，同时传递一个响应对象ClientResponse供操作，后续响应报文以只读流的方式提供。

http.ClientRequest由 http.request 或 http.get 返回产生的对象，表示一个已经产生而且正在进行中的HTTP 请求。它提供一个 response 事件，即 http.request 或 http.get 第二个参数指定的回调函数的绑定对象。

http.ClientResponse提供了三个事件 data、end和close，分别在数据到达、传输结束和连接结束时触发，其中 data 事件传递一个参数chunk，表示接收到的数据。

## HTTP客户端事件

* response：与服务端的request事件对应的客户端在请求发出后得到响应时触发该事件。
* socket：当底层连接池中建立的连接分配给当前请求对象时触发；
* connect：当客户端向服务器发送CONNECT请求时，若服务端响应了200状态码，客户端将会触发该事件。
* upgrade：客户端享服务端发送Upgrade请求时，若服务端响应了101 Switching Protocols状态，客户端将会触发该事件。
* continue：客户端向服务端发起Expect: 100-continue头信息后，以试图发送较大数据，若服务端响应100 continue状态，服务端将触发该事件

# WebSocket服务

## WebSocket最早是作为HTML5重要特性出现的，相比HTTP有以下优点：

* 客户端和服务端只建立一次TCP连接，可以使用更少的连接
* WebSocket服务端可以推送数据到客户端，这远比HTTP请求响应模式更灵活、更高效
* 更轻量级的协议头，减少数据传输

Node中没有内置WebSocket的库，但社区的ws模块封装了WebSocket的底层实现如著名的socket.io
