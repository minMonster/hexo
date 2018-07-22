---
title: NodeJS 读书笔记 第4章
date: 2018-06-30 00:34:06
tags: [NodeJS]
description: 异步编程
categories: NodeJS
---

## 函数式编程基础

### 高阶函数  

高阶函数式可以接受函数作为参数或者返回值的函数,这样在编写程序的时候就增加了灵活性,还能形成一种后续传递风格的结果接受方式,将业务从返回值转移到回到函数中

````javascript
function test(x) {
    return x + 1;
}
function test2(x) {
    return x + 2;
}
function func(x,bar) {
    return bar(x);
}
console.log(func(2,test)); // 3
console.log(func(2,test2)); // 4
````

### 偏函数 

偏函数指创建一个调用另外一个部分(参数或者变量已经预置的函数)的函数

````javascript
let toString = Object.prototype.toString;
let isType = function(type) {
    return function(obj) {
        return toString.call(obj) == '[object ' + type + ']'
    }
}  // 这里我们通过预置了type参数 来实现了偏函数
````

## 异步编程的优势和难点

### 优势

Node中异步编程的最大优势是基于时间驱动的非阻塞的I/O模型,这样就能使CPU和I/O互相不依赖,更好的利用资源,从而达到并行的目的(主要是减少对CPU的占用)

### 难点

#### 异常处理  
在实现异步I/O的过程中主要包含两个阶段 提交请求和处理结果 我们在第一个阶段提交请求后立即返回,错误有可能出现在处理结果这个阶段,所以当我们对第一个阶段进行异常处理的时候发挥不了作用
Node的解决方案是将异常作为回调函数的第一个实参传回,如果为空就代表异步调用没有异常,在编写异步方法的时候 要遵守以下的两个原则

必须执行调用者传入的回调函数
正确传递回异常供调用者判断

```javascript
var async = function(callback) {
    //一些操作  获取一些数据或者数据处理等
    var results = something;
    if(error) {
        return callback(error);
    }
    callback(null,results);
}
```

#### 函数嵌套过深 

在我们获取资源的时候,资源之间是没有依赖关系的,在进行结果处理的时候却需要三者,这样就会造成函数的嵌套

#### 阻塞代码 

对于javascript编程来说,是不存在阻塞代码的,Node中单线程的原因,线程需要运行事件循环的调度,长时间的占用主线程会破坏事件循环,解决方案是统一业务逻辑,使用setTimeout()来完成类似的效果

#### 多线程编程 
 
类似于前端提出的web workers,Node中提出了child_process,cluster是更深层的多线程方式

#### 异步转同步  

Node中同步的API较少,对于异步的调用,通过良好的流程控制,还是能够将流程梳理成顺序式的

## 异步编程解决方案 

现在主要有三种典型的异步编程解决方案

* 事件发布/订阅模式
* Promise/Deferred模式
* 流程控制库(流程控制库这部分介绍性的东西偏多 我没有整理)

### 事件发布/订阅模式
发布/订阅模式被广泛的应用于异步编程,它通过将回调函数事件化的执行,这样就使得事件与具体的逻辑解耦和关联.在进行组件的设计的时候,通过事件的方式将自定义的部分通过事件的方式暴露给外部.并且事件模式也提供了钩子机制,利用钩子我们能导出内部数据或状态给外部的调用者

Node中当设置过多的监听器的时候,会收到一条警告.设计者认为这样会造成内存的泄漏,同时一系列的监听器的执行有可能过多的占用CPU,导致其他的异步调用无法执行
必须对异常事件做好处理,否则会引发主线程的退出
可以通过继承events模块来实现发布/订阅模式来解决业务中的问题

```javascript
var events = require("events");
var util = require("util");
function Stream() {
    events.EventEmitter.call(this);
}
util.inherits(Stream,events.EventEmitter); 
```
### 解决雪崩问题
雪崩问题是突然间大量相同的操作进行访问的时候,导致同时进行相同的查询或者操作,数据库无法同时承受如此大的查询请求,进而影响网站的总体速度

一种比较简单的方案就是使用状态锁

```javascript

var status = "ready";
var select = function(callback){
    if(status === "ready") {
        status = "pending";
        db.select("SQL",function(err,results){
            status = "ready";
            callback(results);
        });
    }
} // 这样当同时进行多次调用的时候,只有首次的调用是有数据服务的 这样的方式并不合理
```
结合状态锁和发布/订阅模式来解决雪崩问题

```javascript
var events = require("events");
var proxy = new events.EventEmitter();
var status = "ready";
var select = function(callback){
    proxy.once("selected",callback);
    if(status === "ready") {
        status = "pending";
        db.select("SQL",function(error,results){
            proxy.emit("selected",results);
            status = "ready";
        });
    }
}//在这种模式下同样的请求之后执行一次,后续的请求会被加入到事件的队列中,当数据可用时,每个请求的回调函数都会被执行一次
```
### 多异步之间的协作方案  

可以使用发布/订阅模式来解决嵌套过深和梳理业务逻辑   
哨兵变量指用于检测次数的变量 
我们可以使用偏函数和哨兵变量的模式来梳理异步作业中多对一的场景

```javascript
var after = function(times,callback){
    var counts = 0,results = {};
    return function(key,value){
        results[key] = value;
        count++;
        if(count === times) {
            callback(results);
        }
    }
}  //这样在完成times调用的时候,才会调用回调函数 并且会把之前的结果传入   还有朴大写的EventProxy模块也是通过补充发布订阅模式,来协同多异步操作
```

### Promise/Deferred模式  

Promise/Deferred模式它是一种先执行异步延时传递处理的模式,它能弥补发布/订阅模式的不足(必须预先设定好分支),Promise/Deferred模式同时能一定上解决函数调用嵌套的问题,方便我们更好的理解业务逻辑  在ES6中已经提供了对Promise的支持  简单的理解Promise/Deferred模式 then()方法就是把回调函数存放起来 然后通过Deferred(延时)对象在适当的时候去调用保存起来的方法

#### Promises/A简介  

Promises/A对单个异步操作进行了抽象的定义

Promise操作只会存在三种状态：
* 进行中（Pending）
* 完成态（Resolved）
* 失败态（Rejected）

Promise状态只能从未完成状态往失败状态或者完成态转变 并且不能逆转 完成态和失败态不能互相转化(转化后的状态无法更改)
在使用Promise的时候,只需要在Promise的then()方法中传递相应的回调函数即可,它就会在异步操作处于完成或者失败状态的时候调用对应的函数并且将结果作为参数传递进去,then()方法只接受function对象并且执行完then()方法后继续返回Promise()对象来实现链式调用

```javascript
var util = require("util");
var events = require("events");
var Promise = function() {
    events.EventEmitter.call(this);
}
util.inherits(Promise,event.EventEmitter);
Promise.prototype.then = function(fulfilledHandler,errorHandler,progressHandler){
    if(typeof fulfilledHandler === 'function') {
        this.once('success',fulfilledHandler);
    }
    if(typeof errorHandler === 'function') {
        this.once('error',errorHandler);
    }
    if(typeof progressHandler === 'function') {
        this.once('progress',progressHandler);
    }
    return this;
};
var Deferred = function(){
    this.state = 'unfulfilled';
    this.promise = new Promise();
}
Deferred.prototype.resolve = function(obj) {
    this.state = 'fulfilled';
    this.promise.emit('success',obj);
}
Deferred.prototype.reject = function(obj) {
    this.state = 'failed';
    this.promise.emit('failed',obj);
}
Deferred.prototype.progress = function(data) {
    this.promise.emit('progress',data);
} //我们在实际操作的时候,就是通过对Deferred进行封装来实现Promise\Deferred模式  也就是在异步操作成功的时候调用Deferred对象的resolve方法,在失败的时候调用reject方法
下面我们封装一个基本的数据请求
var promisify = function(res) {
    var deferred = new Deferred();
    var results = '';
    res.on('data',function(chunk){
        results += chunk;
        deferred.progress(chunk);
    })
    res.on('end',function(){
        deferred.resolve(results);
    })
    res.on('error',function(err){
        deferred.reject(err);
    })
    return deferred.promise;
}
```

#### 支持序列化的Promise

```javascript
var Promise = function(){
    this.isPromise = true;
    this.queue = [];
}
Promise.prototype.then = function(fulfilledHandler,errorHandler,progressHandler){
    var handler = {};
    if(typeof fulfilledHandler === 'function') {
        handler.fulfilled = fulfilledHandler;
    }
    if(typeof errorHandler === 'function') {
        handler.error = errorHandler;
    }
    this.queue.push(handler);
    return this;
}
var Deferred = function() {
    this.promise = new Promise();
}
Deferred.prototype.resolve = function(obj) {
    var promise = this.promise;
    var handler;
    while((handler = promise.queue.shift())) {
        if(handler && handler.fulfilled) {
            var ret = handler.fulfilled(obj);
            if(ret && ret.isPromise) {
                ret.queue = promise.queue;
                this.promise = ret;
                return;
            }
        }
    }
}
Deferred.prototype.reject = function(err) {
    var promise = this.promise;
    var hanlder;
    while((hanlder = promise.queue.shift())) {
        if(handler && hanlder.error) {
            var ret = hanlder.error(err);
            if(ret && ret.isPromise) {
                ret.queue = promise.queue;
                this.promise = ret;
                return;
            }
        }
    }
}
//上面的的代码进所有的回调函数保存为对象存储在队列中.promise完成是依次的执行回调,如果返回新的promise对象,就将当前deferred对象的promise对象的promise设置为新的
promise对象(这个就是解决回调地狱的解决方案)
```
