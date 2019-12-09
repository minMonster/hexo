---
title: 阿里云服务器（Centos）部署 node 服务，问题记录
date: 2019-05-23 11:13:46
description: 阿里云服务器（Centos）部署 node 服务，问题记录
tags: [问题总结]
categories: Linux
---
最近手里有一个官网展示项目，本地开发还算顺利，阿里云服务器部署遇到了一些问题

### 用node启动server后，发现服务器不稳定，经常crash。我是用ssh远程登录的，ssh远程通道中断，或者Ctrl+C,都会使nodejs server崩溃掉。
使用 nohup 来启动项目（不是最优解决方案）
用途：不挂断地运行命令（也就是后台运营node程序）
```bash
nohup yarn start &
```
#### 使用 nohup 启动服务解决了 断开 shh 服务中断这个问题
但是不能保证 node 服务因为意外的报错中断服务 下文给出了解决方案
### node 是单线程阻塞 io ，意味着有可能服务遇到报错直接停止服务器
单线程的某处产生了“未处理的”异常确实会导致整个Node.JS的崩溃退出，来看个例子, 这里有一个node-error.js的文件： 

```javascript
var http = require(＇http＇);

var server = http.createServer(function (req, res) {

  //这里有个错误，params 是 undefined
  var ok = req.params.ok;

  res.writeHead(200, {＇Content-Type＇: ＇text/plain＇});
  res.end(＇Hello World
＇);
});

server.listen(8080, ＇127.0.0.1＇);

console.log(＇Server running at http://127.0.0.1:8080/＇);
```
启动服务，并在地址栏测试一下发现 http://127.0.0.1:8080/  不出所料，node崩溃了 
```bash
$ node node-error
Server running at http://127.0.0.1:8080/

c:githubscript
ode-error.js:5
  var ok = req.params.ok;
                     ^
TypeError: Cannot read property ＇ok＇ of undefined
    at Server.<anonymous> (c:githubscript
ode-error.js:5:22)
    at Server.EventEmitter.emit (events.js:98:17)
    at HTTPParser.parser.onIncoming (http.js:2108:12)
    at HTTPParser.parserOnHeadersComplete [as onHeadersComplete] (http.js:121:23)
    at Socket.socket.ondata (http.js:1966:22)
    at TCP.onread (net.js:525:27)
```
[原文](https://blog.csdn.net/xiunai78/article/details/40378965/)有写怎么在逻辑层处理报错
这种情况当然不是我们愿意看到的，怎么保持服务的稳定性呢？

#### 使用 pm2 来守护进程 保证服务 不会因为某一个错误，导致全线崩溃
pm2 简介
* PM2是node进程管理工具，可以利用它来简化很多node应用管理的繁琐任务，如性能监控、自动重启、负载均衡等，而且使用非常简单。
pm2 安装
```bash
npm install -g pm2
```
入门教程
使用我们当前的 koa2 项目来举例。一般我们都是使用 yarn start 来启动应用 实际上 是使用 node bin/www 来启动项目
```json
 "scripts": {
    "start": "node bin/www",
    "dev": "./node_modules/.bin/nodemon bin/www",
    "prd": "pm2 start bin/www",
    "test": "echo \"Error: no test specified\" && exit 1"
 }
```
注意，这里用了--watch参数，意味着当你的koa2应用代码发生变化时，pm2会帮你重启服务，多贴心。
```bash
pm2 start ./bin/www --watch
```
入门太简单了，没什么好讲的。直接上[官方文档](http://pm2.keymetrics.io/docs/usage/quick-start)

详细使用教程 参考 [PM2实用入门指南](https://blog.csdn.net/maquealone/article/details/79550120)

我们服务启动了，我们通过 nginx 反向代理 将服务代理到 9010 端口

```bash
server {
        listen  9010;
        server_name 47.***.***.72;
	    root 47.***.***.72;
        location / {
                proxy_pass http://127.0.0.1:3000;
                proxy_http_version 1.1;
                proxy_set_header   Upgrade $http_upgrade;
                proxy_set_header   Connection keep-alive;
                proxy_set_header   Host $http_host;
                proxy_cache_bypass $http_upgrade;
        }
     }
```

### 服务启动了但是通过 ip + 端口号不能访问

但是我发现 通过服务器 ip 端口号访问 并不通 why？？？？

思考了一下列举出了以下可能的问题

#### 首先验证服务是否是正常的

我们通过 ssh 进入服务器
然后查看 进程是否还在

```bash
pm2 list
┌──────┬────┬──────┬─────────┬────┬──────┬───────────┐
│ Name │ id │ mode │ status  │ ↺  │ cpu  │ memory    │
├──────┼────┼──────┼─────────┼────┼──────┼───────────┤
│ app  │ 1  │ fork │ stopped │ 0  │ 0%   │ 0 B       │
│ node │ 2  │ fork │ online  │ 0  │ 0%   │ 17.3 MB   │
│ www  │ 3  │ fork │ online  │ 0  │ 0.2% │ 80.4 MB   │
│ yarn │ 0  │ fork │ stopped │ 75 │ 0%   │ 0 B       │
└──────┴────┴──────┴─────────┴────┴──────┴───────────┘
```
我们看到服务正在正常的跑着

#### 端口号是否是通的

查看 80 默认端口是否通

```bash
server {
        listen  8080;
        server_name 47.***.***.72;
	    root 47.***.***.72;
        location / {
                proxy_pass http://127.0.0.1:3000;
                proxy_http_version 1.1;
                proxy_set_header   Upgrade $http_upgrade;
                proxy_set_header   Connection keep-alive;
                proxy_set_header   Host $http_host;
                proxy_cache_bypass $http_upgrade;
        }
     }
```

将 node 服务代理到 80端口

发现通过 ip 直接访问是通的

这时候就可以得出结论

* 服务是没有问题的
* 9010 端口可能不通

但是为什么80端口是通的9010是不通的？
通过百度大法，以及多方打听列举处两个有可能的点

* 阿里云安全组端口权限

* centos 防火墙端口是否开启

### Centos 防火墙 开放端口
使用 systemd 来管理 firewalld
systemd 是 Linux 下的一款系统和服务管理器，兼容 SysV 和 LSB 的启动脚本。systemd 的特性有：支持并行化任务；同一时候採用 socket 式与 D-Bus 总线式激活服务；按需启动守护进程（daemon）。利用 Linux 的 cgroups 监视进程；支持快照和系统恢复。维护挂载点和自己主动挂载点。各服务间基于依赖关系进行精密控制。
1、firewalld的基本使用
启动： systemctl start firewalld
关闭： systemctl stop firewalld
查看状态： systemctl status firewalld 
开机禁用  ： systemctl disable firewalld
开机启用  ： systemctl enable firewalld
 
 
2.systemctl是CentOS7的服务管理工具中主要的工具，它融合之前service和chkconfig的功能于一体。
启动一个服务：systemctl start firewalld.service
关闭一个服务：systemctl stop firewalld.service
重启一个服务：systemctl restart firewalld.service
显示一个服务的状态：systemctl status firewalld.service
在开机时启用一个服务：systemctl enable firewalld.service
在开机时禁用一个服务：systemctl disable firewalld.service
查看服务是否开机启动：systemctl is-enabled firewalld.service
查看已启动的服务列表：systemctl list-unit-files|grep enabled
查看启动失败的服务列表：systemctl --failed

3.配置firewalld-cmd

查看版本： firewall-cmd --version
查看帮助： firewall-cmd --help
显示状态： firewall-cmd --state
查看所有打开的端口： firewall-cmd --zone=public --list-ports
更新防火墙规则： firewall-cmd --reload
查看区域信息:  firewall-cmd --get-active-zones
查看指定接口所属区域： firewall-cmd --get-zone-of-interface=eth0
拒绝所有包：firewall-cmd --panic-on
取消拒绝状态： firewall-cmd --panic-off
查看是否拒绝： firewall-cmd --query-panic
 
那怎么开启一个端口呢
添加
firewall-cmd --zone=public --add-port=80/tcp --permanent    （--permanent永久生效，没有此参数重启后失效）
重新载入
firewall-cmd --reload
查看
firewall-cmd --zone= public --query-port=80/tcp
删除
firewall-cmd --zone= public --remove-port=80/tcp --permanent

### 阿里云开放安全组

废话少说直接上阿里云文档 [添加安全组规则](https://helpcdn.aliyun.com/document_detail/25471.html)
