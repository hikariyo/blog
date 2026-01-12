---
title: 开发 - Nginx 自定义错误页面
date: 2022-05-30 00:03:05.0
updated: 2022-08-10 15:18:36.094

categories: 
- Dev
tags: 
- Nginx
---



打开配置文件，找到你想自定义错误页面的`location`块：

```nginx
location / {
    error_page 404 /404.html;
    # 其它配置
}
```

确保你有`404.html`这一文件，如果还想捕捉更多错误，可以这样写：

```nginx
error_page 404 403 500 /error.html;
```

这样就完成了，不用再看经典的Nginx默认错误页面了。