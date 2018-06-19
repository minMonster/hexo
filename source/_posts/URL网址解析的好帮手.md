---
title: URL网址解析的好帮手
date: 2018-02-24 18:28:38
tags: [NodeJS]
description: node 中 url 解析以及反解析
categories: NodeJS
---
## URI简介:
URI统一资源标识符
URL统一资源定位符
URL是URI的子集，URL肯定是URI,URI不一定是URL

## url地址组成:
**protbcol**:指定底层使用协议，http或ftp等
**slashes**:是否有协议的双斜线
**host**:http的IP地址或域名
**port**:端口
**hostname**:主机名
**hash**:哈希值
**search**:查询字符串参数
**query**:发送给http的数据，参数串
**pathname**:访问资源路径名
**path**:路径
**href**:超链接

## 解析url:

````bash
$ url.parse(URL)

$ url.parse('https://www.imooc.com/course/list?c=nodejs')

Url {
  protocol: 'https:',
  slashes: true,
  auth: null,
  host: 'www.imooc.com',
  port: null,
  hostname: 'www.imooc.com',
  hash: null,
  search: '?c=nodejs',
  query: 'c=nodejs',
  pathname: '/course/list',
  path: '/course/list?c=nodejs',
  href: 'https://www.imooc.com/course/list?c=nodejs'
}
````

## 反解析:

````bash
format(OBJ)
````

## 序列化
````bash
querystring.stringify(obj,para1,para2)//将对象转化成url中query部分的形式（序列化）
````
参数：
    1.要转化的对象
    2.链接符（默认&）
    3.键与值之间的符号（默认=）

## 反序列化
````bash
querystring.parse(string,para1,para2)//将query字符串转化成对象（反序列化）
````
参数：
    1.query字符串
    2.链接符（默认&）
    3.键与值之间的符号（默认=）
    4.参数的个数（默认最多1000个，0就没有限制）
````bash
querystring.escape(string)//文字转译
querystring.unescape(string)//反转译
````

## http协议

DNS 域名系统 (Domain Name System) 的缩写

http客户端发起请求，创建端口
http服务器在端口监听客户端请求
http服务器向客户端返回状态和内容
请求和响应都发送 http 头和正文信息，http 头发送内容类型、http 状态码，正文是提交的数据或者服务器返回的数据
1.chrome搜索自身的DNS缓存
2.搜索操作系统自身的DNS缓存（浏览器没有找到缓存或缓存已经失效）
3.读取本地的host文件
4.浏览器发起一个DNS的一个系统调用：①宽带运营商服务器查看本身缓存；②运营商服务器发起一个迭代DNS解析的请求；③运营商服务器把结果返回操作系统内核同时缓存起来，④操作系统内核把结果返回浏览器；⑤最终浏览器拿到了目标网站对应的IP地址
5.浏览器获得域名对应ip地址后，发起HTTP“三次握手”
6.TCP/IP连接建立起来后，浏览器就可以向服务器发送HTTP请求了。
7.服务器端接受到这个请求，根据路径参数，经过后端处理后，把处理后的一个结果的数据返回给浏览器。
8.浏览器获取到目标网址的数据，例如返回一个HTML文件,HTML文档内的JS/CSS/图片静态资源同样也是一个个HTTP请求，也要包括上述步骤。
9.浏览器根据获取到的资源对页面进行渲染，最终把网页呈献给用户

## http头和正文信息

HTTP头发送的是一些附加的信息：内容类型、服务器发送相应的日期、HTTP状态码

正文就是用户提交的表单数据。
