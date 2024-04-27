---
title: 自建图床并支持 PicGo 上传
date: 2023-05-13 19:19:34
update: 2023-12-13 00:37:00
tags:
- Python
- VPS
category: Dev
---

## 简介

由于 `Hexo` 的图片管理系统让人比较一言难尽，所以要使用`图床`来统一管理图片。一般使用 `sm.ms` 或者 `github` 仓库作为图床比较多。本着多折腾的心态，再加上我的图不是很多，所以就尝试自己搭建，本文记录一下搭建过程。

补充：现在因为没有服务器的需求，已经改成静态资源了。


## 部署后端

我们需要自己配置一个后端接口，支持上传图片，我这里用 [gist](https://gist.github.com/hikariyo/cc4b53e49ec5ff7f020037514ddd74a9) 分享出来了我写的一个简单的后端程序，将几个全局变量替换为对应值即可。如果想要别的功能，那就自己动手丰衣足食了。


可以通过 `pm2` 或者其它的工具来部署，如下：

```bash
pm2 start "uvicorn picgo-server:app --port 2333" --name picgo
```

## PicGo

到 [GitHub Releases](https://github.com/Molunerfinn/PicGo/releases) 中挑一个稳定版下载就好，我用的是 `2.3.1` 版本。

还需要使用 [web-uploader](https://github.com/yuki-xin/picgo-plugin-web-uploader) 这个插件，可以从 `PicGo` 内部的插件设置中安装，如果网络不太好的话，也可以先到 `Release` 页下载源代码然后本地安装，这里不再赘述。

接着前往 `PicGo` 中配置各项参数，这里要把你自定义的 `TOKEN` 填到自定义请求头中，格式如 `{"authorization": "token"}`，这是防止别人猜到这个接口，毕竟防人之心不可无。

![](/img/2023/05/picgo-web-uploader.jpg)

~~恭喜你，从此电脑上又多了一个次世代电子垃圾 -- `electron`~~

## Nginx

对于图片，最好就不要直接通过 `Python` 代理静态文件了，这里使用 `Nginx` 代理图片。如果你的 `Python` 文件放在 `/foo/bar/picgo-server.py` 下，那么第一个 `location` 块中的 `root` 就要写 `/foo/bar/data`，这是在 `picgo-server.py` 里面定义的，你可以改成自己想要的名字。下面是示例：

```nginx
location / {
  root /path/to/data;
  try_files $uri $uri/ =404;
}

location /upload {
  proxy_set_header HOST $host;
  proxy_set_header X-Forwarded-Proto $scheme;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_pass http://127.0.0.1:2333;
}
```

