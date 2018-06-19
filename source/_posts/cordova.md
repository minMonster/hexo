---
title: cordova
date: 2018-01-29 11:04:13
description: cordova 环境搭建以及基本指令
tags: [cordova, 环境搭建]
categories: cordova
---
[cordova]: http://cordova.axuer.com/docs/zh-cn/latest/  "cordova中文文档"

# [cordova]中文文档

## 安装 cordova

Cordova命令行工具作为npm包分发。

``` bash
$ npm install cordova -g
```

## 创建App

跳转到你维护源代码的目录中，并创建你的cordova项目：

``` bash
$ cordova create hello com.example.hello HelloWorld
```

* 第一个参数是项目目录
* 第二个参数是 app 名称 
* 第三个参数是项目名称

## 添加平台

所有后续命令都需要在项目目录或者项目目录的任何子目录运行:

``` bash
$ cd hello
```

给你的App添加目标平台。我们将会添加'ios'和'android'平台，并确保他们保存在了config.xml中:

``` bash
$ cordova platform add ios --save
$ cordova platform add android --save
```

### 检查你当前平台设置状况:

``` bash
$ cordova platform ls
```

## 安装构建先决条件

要构建和运行App，你需要安装每个你需要平台的SDK。另外，当你使用浏览器开发你可以添加 browser平台，它不需要任何平台SDK。

检测你是否满足构建平台的要求:

``` bash
$ cordova requirements

Requirements check results for android:
Java JDK: installed .
Android SDK: installed
Android target: installed android-19,android-21,android-22,android-23,Google Inc.:Google APIs:19,Google Inc.:Google APIs (x86 System Image):19,Google Inc.:Google APIs:23
Gradle: installed

Requirements check results for ios:
Apple OS X: not installed
Cordova tooling for iOS requires Apple OS X
Error: Some of requirements check failed
```
## 构建App

默认情况下, **cordova create**生产基于web应用程序的骨架，项目开始页面位于**www/index.html** 文件。任何初始化任务应该在**www/js/index.js**文件中的deviceready事件的事件处理函数中。

运行下面命令为所有添加的平台构建:

``` bash 
$ cordova build
```

你可以在每次构建中选择限制平台范围 - 这个例子中是'ios':

``` bash 
$ cordova build ios
```

## 查看插件列表

``` bash 
$ cordova plugin ls
```

## 查看插件列表

``` bash 
cordova plugin add <插件名称 例子>
```

## 调试

``` bash 
$ cordova run android
```

## 生成发布版app

``` bash 
$ cordova build --release
```

## Adb安装应用程序
``` bash 
$ adb  install + app路径
```

例：

> add install /Users/deepera/ws/cordova/client/platforms/android/build/outputs/apk/android-release.apk

## 查看链接设备

``` bash
$ adb devices
```
