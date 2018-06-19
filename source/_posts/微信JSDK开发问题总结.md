---
title: 微信JSDK开发问题总结
date: 2018-02-24 18:08:36
tags: [微信开发]
description: 微信JSDK开发问题总结
categories: 微信
---
## 步骤一：绑定域名
先登录微信公众平台进入“公众号设置”的“功能设置”里填写“JS接口安全域名”。
备注：登录后可在“开发者中心”查看对应的接口权限。

## 步骤二：引入JS文件
Github地址 https://github.com/yanxi-me/weixin-js-sdk
在需要调用JS接口的页面引入如下JS文件（支持https）：http://res.wx.qq.com/open/js/jweixin-1.2.0.js
备注：支持使用 AMD/CMD 标准模块加载方法加载

## 步骤三：通过config接口注入权限验证配置

````bash
this.$wx.config({
  debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
  appId: APPID, // 必填，公众号的唯一标识
  timestamp: data['timestamp'], // 必填，生成签名的时间戳
  nonceStr: data['noncestr'], // 必填，生成签名的随机串
  signature: data['signature'], // 必填，签名，见附录1
  jsApiList: ['scanQRCode'] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
})
````

调试的时候一定要打开 debug 看报错检查问题

## 步骤四：通过ready接口处理成功验证
wx.ready(function(){
    // config信息验证后会执行ready方法，所有接口调用都必须在config接口获得结果之后，config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，则须把相关接口放在ready函数中调用来确保正确执行。对于用户触发时才调用的接口，则可以直接调用，不需要放在ready函数中。
});

## 步骤五：通过error接口处理失败验证
````bash
wx.error(function(res){
    // config信息验证失败会执行error函数，如签名过期导致验证失败，具体错误信息可以打开config的debug模式查看，也可以在返回的res参数中查看，对于SPA可以在这里更新签名。
});
this.$wx.checkJsApi({
  jsApiList: [
    'scanQRCode'
  ],
  success: function (res) {
    console.log(JSON.stringify(res))
  }
})
this.$wx.scanQRCode({
  needResult: 1, // 默认为0，扫描结果由微信处理，1则直接返回扫描结果，
  scanType: ['qrCode', 'barCode'], // 可以指定扫二维码还是一维码，默认二者都有
  success: function (res) {
    let result = res.resultStr // 当needResult 为 1 时，扫码返回的结果
    console.log(result)
    alert(result)
  }
})
````

**重点：一定要检查 appid 是否与后台相同**
**一定要检查当前插件是否能够启用**






