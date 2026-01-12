---
title: 开发 - 记一次 Nginx 调试
date: 2022-09-10 04:29:05.527
updated: 2022-09-10 04:34:51.965
categories: 
- Dev
tags: 
- Nginx
---

## 背景

一般来说 `Nginx` 使用起来挺简单，不出意外的话应该没什么问题，但今天我就出意外了。

我的配置很简单，如下：

```nginx
server {
  server_name www.hikariyo.net;
  root /web/www;
  index index.html;

  location / {
    try_files $uri $uri/ =404;
  }
}
```

但怎么访问就是死活就是报 `404`，于是就有了今天这篇文章。

## 修改日志等级

我演示的用户都是非 `root`，如果你是 `root` 就不用加那么多 `sudo` 了。我主要是怕误操作。

首先查找配置文件的路径：

```bash
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

打开配置文件，找到以下几行：

```nginx
##
# Logging Settings
##

access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;
```

在这里，我们需要把 `error_log` 的级别调到 `debug`：

```nginx
error_log /var/log/nginx/error.log debug;
```

不要忘了重新加载以下 `Nginx`：

```bash
$ sudo nginx -s reload
```

## 查看日志

**再次访问出错的地址**，然后查看 `error.log` 中的内容，我这里是找到了下面几行（里面的一些星号是我自己写的，原来是 `IP` 地址之类的）：

```bash
$ sudo cat /var/log/nginx/error.log
...
2022/09/10 03:59:17 [debug] 69714#69714: *2047 http script var: "/"
2022/09/10 03:59:17 [debug] 69714#69714: *2047 trying to use file: "/" "/web/www/"
2022/09/10 03:59:17 [crit] 69714#69714: *2047 stat() "/web/www/" failed (13: Permission denied), client: ***, server: www.hikariyo.net, request: "GET / HTTP/1.1", host: "www.hikariyo.net"
2022/09/10 03:59:17 [debug] 69714#69714: *2047 http script var: "/"
2022/09/10 03:59:17 [debug] 69714#69714: *2047 trying to use dir: "/" "/web/www/"
2022/09/10 03:59:17 [crit] 69714#69714: *2047 stat() "/web/www/" failed (13: Permission denied), client: ***, server: www.hikariyo.net, request: "GET / HTTP/1.1", host: "www.hikariyo.net"
2022/09/10 03:59:17 [debug] 69714#69714: *2047 trying to use file: "=404" "/web/www=404"
2022/09/10 03:59:17 [debug] 69714#69714: *2047 http finalize request: 404, "/?" a:1, c:1
2022/09/10 03:59:17 [debug] 69714#69714: *2047 http special response: 404, "/?"
2022/09/10 03:59:17 [debug] 69714#69714: *2047 http set discard body
2022/09/10 03:59:17 [debug] 69714#69714: *2047 xslt filter header
2022/09/10 03:59:17 [debug] 69714#69714: *2047 HTTP/1.1 404 Not Found
Server: nginx/1.18.0 (Ubuntu)
Date: Sat, 10 Sep 2022 03:59:17 GMT
Content-Type: text/html
Transfer-Encoding: chunked
Connection: keep-alive
Content-Encoding: gzip
...
```

可以注意到，它是因为访问 `/web/www` 的权限不足才报错，那么我们可以这么办：

```bash
$ sudo chmod -R 777 /web/www
```

再次访问我就没问题了。

## 改回原来的日志等级

Bug 修完了，现在该还原了。打开配置文件，把之前添加的 `debug` 去掉，然后重新加载 `Nginx`：

```bash
$ sudo nginx -s reload
```

## 错误产生原因

我之前是用 `root` 用户创建的这个文件夹，之后就忘了这码事了，所以 `Nginx` 运行的时候才报的权限不足。
