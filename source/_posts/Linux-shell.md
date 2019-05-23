---
title: Linux shell
date: 2018-01-23 18:57:42
update: 2019-04-28 16:43:26
description: linux 命令汇总
tags: [shell]
categories: Linux
---
## 磁盘管理

### 删除文件

```bash
$ rm -rf demo.txt
```

**-rf**:强制删除并且不提醒

### 切换目录

``` bash 
$ cd [dirName]
```

**dirName**：要切换的目标目录。

### 查看一个文件

``` bash
$ cat demo.txt
```

### 触碰一个文件

``` bash
$ touch demo.txt
```
**如果没有这个文件则新建**

### 新建一个文件夹

``` bash
$ mkdir demo
```

## 服务器管理

### 登录服务器

**端口号**: -P2222

**username**: root

**IP**: 47.94.238.16

```bash
$ ssh -P2222 root@47.94.238.16
```

### 查看进程

-A 　显示所有程序。 

-e 　此参数的效果和指定"A"参数相同。

-f 　显示[UID](https://www.baidu.com/s?wd=UID&tn=67012150_cpr&fenlei=mv6quAkxTZn0IZRqIHcvrjTdrjb0T1Y1n16krjF9nHn3uhuBuynv0ZwV5fKWUMw85HmLnjDznHRsgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EPW6sPjmLPWR1njDsrjRsn1Tz),PPIP,C与S[TIME](https://www.baidu.com/s?wd=TIME&tn=67012150_cpr&fenlei=mv6quAkxTZn0IZRqIHcvrjTdrjb0T1Y1n16krjF9nHn3uhuBuynv0ZwV5fKWUMw85HmLnjDznHRsgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EPW6sPjmLPWR1njDsrjRsn1Tz)栏位。 

grep命令是查找

中间的|是管道命令

是指ps命令与grep同时执行

```bash
$ ps -ef | grep java 
```

### 杀掉进程

> **-9强制杀掉进程**

```bash
$ kill -9 java_id
```

### 上传文件

> **-r** 可以上传文件夹

```bash
$ scp -p2222 -r demo.txt root@47.94.238.16:/usr/local/tomcat/tomcat_api/webapps
```

### 搜索文件命令

> **-name** 制定用什么搜索

```bash
 find / -name log.datetime
```



