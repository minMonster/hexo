---
title: Promise与异步编程
date: 2018-08-26 22:41:27
tags: [ES6]
description: Promise与异步编程
categories: javaScript
---
# 为什么要异步编程

我们在写前端代码时，经常会对dom做事件处理操作，比如点击、激活焦点、失去焦点等；再比如我们用ajax请求数据，使用回调函数获取返回值。这些都属于异步编程。

也许你已经大概知道Java引擎单线程的概念，那么这种单线程模式和异步编程有什么关系呢？

** Java引擎中，只有一个主线程，当执行Java代码块时，不允许其他代码块执行，而事件机制和回调机制的代码块会被添加到任务队列（或者叫做堆栈）中，当符合某个触发回调或者事件的时候，就会执行该事件或者回调函数。上面这段话的意思可以这样理解，假设你是一个修仙者，你去闯一个秘境，这个秘境就是主线程，你只能一直深入下去，直到找到宝物和出口，而你还有一个自身的储物空间，这个空间就类似堆栈，你在储物空间放了很多你可能用到的法宝或者丹药，这些东西就是回调函数和事件函数，当你遇到危险或者满足某个条件时，就可以从储物空间拿出你当前需要的东西。好吧，不扯这么远，下面看正题。 **

* 事件模型：浏览器初次渲染DOM的时候，我们会给一些DOM绑定事件函数，只有当触发了这些DOM事件函数，才会执行他们。

```javascript
const btn = document.querySelector('.buttn')
btn.onclick = function(event){
	console.log(event)
}
```

* 回调模式：nodejs中可能非常常见这种回调模式，但是对于前端来说，ajax的回调是最熟悉不过了。ajax回调有多个状态，当响应成功和失败都有不同的回调函数。

```javascript
$.post('/router', function(data) {
	console.log(data)
})
```

回调也可能带来一个问题，那就是地狱回调。

# Promise

事件函数没有问题，我们用的很爽，问题出在回调函数，尤其是指地狱回调，Promise的出现正是为了避免地狱回调带来的困扰。

推荐你看Java MDN Promise教程，然后再结合本文看，你就能学会使用Promise了。

## Promise是什么

Promise的中文意思是承诺，也就是说，Java对你许下一个承诺，会在未来某个时刻兑现承诺。

## Promise生命周期

react有生命周期，vue也有生命周期，就连Promise也有生命周期，现在生命周期咋这么流行了。

## Promise的生命周期：进行中（pending），已经完成（fulfilled），拒绝（rejected）

Promise被称作异步结果的占位符，它不能直接返回异步函数的执行结果，需要使用then()，当获取异常回调的时候，使用catch()。

这次我们使用axios插件的代码做例子。axios是前端比较热门的http请求插件之一。

1、创建axios实例instance。

2、使用axios实例 + Promise获取返回值。

```javascript
const promise = instance.get('url')
promise.then(res => console.log(res)).catch(err => console.log(err))
```

## 使用Promise构建函数创建新的Promise

```javascript
const promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

Promise构造函数只有一个参数，该参数是一个函数，被称作执行器，执行器有2个参数，分别是resolve()和reject()，一个表示成功的回调，一个表示失败的回调。

记住，Promise实例只能通过resolve或者reject函数来返回，并且使用then()或者catch()获取，不能在new Promise里面直接return，这样是获取不到Promise返回值的。

```javascript
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('resolved.');
});

console.log('Hi!');

// Promise
// Hi!
// resolved
```

1、我们也可以使用Promise直接resolve(value)。

2、也可以使用reject(value)

3、执行器错误通过catch捕捉。

```javascript
new Promise((resolve, reject){
    if(true) {
        throw new Error('error!')
    }
}).catch(v => console.log(v.mewssage))
// error!
```

## 全局的Promise拒绝处理

不重要的内容，不用细看。

这里涉及到nodejs环境和浏览器环境的全局，主要说的是如果执行了Promise.reject()，浏览器或者node环境并不会强制报错，只有在你调用catch的时候，才能知道Promise被拒绝了。

这种行为就像是，你写了一个函数，函数内部有true和false两种状态，而我们希望false的时候抛出错误，但是在Promise中，并不能直接抛出错误，无论Promise是成功还是拒绝状态，你获取Promise生命周期的方法只能通过then()和catch()。

#nodejs环境：

node环境下有个对象叫做process，即使你没写过后端node，如果写过前端node服务器，也应该知道可以使用process.ENV_NODE获取环境变量。为了监听Promise的reject（拒绝）情况，NodeJS提供了一个process.on()，类似jQuery的on方法，事件绑定函数。

process.on()有2个事件

unhandledRjection:在一个事件循环中，当Promise执行reject()，并且没有提供catch()时被调用。

正常情况下，你可以使用catch捕捉reject。

但是，有时候你不总是记得使用catch。你就需要使用process.on()

```javascript
let rejected
rejected = Promise.reject("It was my wrong!" )
process.on("unhandledRjection", function(reason, promise) {
  console.log(reason. message) // It was my wrong !
  console.log(rejected === promise) // true
})
```

rejectionHandled:在一个事件循环后，当Promise执行reject，并且没有提供catch()时被调用。

```javascript
let rejected
rejected = Promise.reject(new Error("It was my wrong!"))
process.on("unhandledRjection", function(reason, promise) {
  console.log(rejected === promise) // true
})
```

# 异同：

事件循环中、事件循环后，你可能很难理解这2个的区别，但是这不重要，重要的是，如果你通过了catch()方法来捕捉reject操作，那么，这2个事件就不会生效。

# 浏览器环境：

和node环境一样，都提供了unhandledRjection、rejectionHandled事件，不同的是浏览器环境是通过window对象来定义事件函数。

```javascript
let rejected
rejected = Promise.reject(new Error("It was my wrong!"))
window.onrejectionhandled = function (event) {
	console.log(event) // true
}
onrejectionhandled()
```

将代码在浏览器控制台执行一遍，你就会发现报错了：Uncaught (in promise) Error: It was my wrong!

耶，你成功了！报错内容正是你写的reject()方法里面的错误提示。

# Promise链式调用

这个例子中，使用了3个then，第一个then返回 s * s，第二个then捕获到上一个then的返回值，最后一个then直接输出end。这就叫链式调用，很好理解的。我只使用了then()，实际开发中，你还应该加上catch()。

```javascript
new Promise(function (resolve, reject){
	try {
		resolve(5)
	} catch (error) {
		reject('It was my wrong!!!')
	}
}).then(s => s * s).then(s2 => console.log(s2)).then(() => console.log('end'))
```

# Promise的其他方法

在Promise的构造函数中，除了reject()和resolve()之外，还有2个方法，Promise.all()、Promise.race()。

Promise.all()：

前面我们的例子都是只有一个Promise，现在我们使用all()方法包装多个Promise实例。

语法很简单：参数只有一个，可迭代对象，可以是数组，或者Symbol类型等。

示例：传入3个Promise实例

```javascript
Promise.all([
	new Promise((resolve, reject) => {
		resolve(1)
	})
	new Promise((resolve, reject) => {
		resolve(2)
	})
	new Promise((resolve, reject) => {
		resolve(3)
	})
]).then(arr => {
	console.log(arr) // [1, 2, 3]
})
```

**Promise.race()：**语法和all()一样，但是返回值有所不同，race根据传入的多个Promise实例，只要有一个实例resolve或者reject，就只返回该结果，其他实例不再执行。

还是使用上面的例子，只是我给每个resolve加了一个定时器，最终结果返回的是3，因为第三个Promise最快执行。

```javascript
Promise.race([
	new Promise((resolve, reject) => {
		setTimeout(() => {
		  resolve(1)
		}, 3000)
	})
	new Promise((resolve, reject) => {
		setTimeout(() => {
          resolve(2)
        }, 2000)
	})
	new Promise((resolve, reject) => {
		setTimeout(() => {
          resolve(3)
        }, 1000)
	})
]).then(value => {
	console.log(value) // 3
})
```

# Promise派生

派生的意思是定义一个新的Promise对象，继承Promise方法和属性。

```javascript
class MyPromise extends Promise {
 	// 重新封装 then()
	success(resolve, reject) {
		return this.then(resolve, reject)
	}
	failer(reject) {
		return this.catch(reject)
	}
}
```

接着我们来使用一下这个派生类。

```javascript
new MyPromise(function(resolve, reject) {
	resolve(10)
}).success(v => console.log(v)) // 10
```

如果只是派生出来和then、catch一样的方法，我想，你不会干这么无聊的事情。

Promise和异步的联系

Promise本身不是异步的，只有他的then()或者catch()方法才是异步，也可以说Promise的返回值是异步的。通常Promise被使用在node，或者是前端的ajax请求、前端DOM渲染顺序等地方。
