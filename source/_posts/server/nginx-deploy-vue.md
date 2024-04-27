---
title: Nginx部署Vue项目
date: 2022-05-03 00:00:00.0
updated: 2022-08-10 15:27:51.013
categories: 
- Dev
tags: 
- Nginx
- Vue
---



用`Vue`开发时，经常配置这样的代理服务器（这里展示`Vite`的配置项）：

```js
{
    server: {
        port: 80,
        proxy: {
            '/api': {
                target: 'http://127.0.0.1:8084/',
                changeOrigin: true,
                rewrite: path => path.replace(/^\/api/, '')
            }
        }
    }
}
```

我们的需求大概是，把类似于这样的请求：

`http://localhost/api/xxx`

转发到：

`http://localhost:8084/xxx`

但是部署到Nginx服务器的时候，显然上面在`Vite`里配置的代理服务器是无效的。

经过我的一番查询，发现下面这些配置是可以起到相同的作用的

## 解决方案


> 本文假设你刚安装好Nginx，还没有进行任何配置。
> 
> 并且，我的服务器是Ubuntu，如果有些命令或者路径不一样，还请去搜一下对应系统版本的命令和路径是什么。

因为Nginx默认是有一个配置文件在生效的，我们需要把它注释掉。

找到配置文件`nginx.conf`，我的服务器路径是`/etc/nginx/nginx.conf`，找到类似于下面的这一项：

```nginx
include /etc/nginx/sites-enabled/*;
```

把这句话注释掉，就是前面加个井号：

```nginx
## include /etc/nginx/sites-enabled/*;
```

刚才注释掉的这句话应该就在`http`配置项里，下一步就是在这个配置项下新加一个`server`项，下面可以这样配置：

```nginx
server {
    # 服务器名字，可以随便填
    server_name  XXX;
    # Nginx去哪里找文件
    root /path/to/web/root;
    # 这里是设置服务器端口为80，并且是默认服务器
    listen 80 default_server;
    # 设置入口文件
    index index.html;
    # 这里相当于Vite里配置的代理服务器
    location ~ /api/ {
        # 把路径头上的/api/去掉
        rewrite "^/api/(.*)$" /$1 break;
        # 把请求转发到本地的8084端口
        proxy_pass http://127.0.0.1:8084;
    }
    # 这个配置项配置这里主要是为了Vue
    # 设置路由为history模式后
    # 刷新界面404的问题
    location / {
        # 这个原理也就是先尝试找到给定的路径
        # 如果找不到就返回index.html
        try_files $uri $uri/ /index.html;
    }
}
```

配置完后重启`nginx`。

```bash
nginx -s reload
```

那么之后就可以把：

`http://localhost/api`

下的请求转发到：

`http://localhost:8084`

## 后记
如果你还需要配置多个代理，如在上文的基础上，还需要把：

`http://localhost/api/test`

下的请求转发到：

`http://localhost:8085`

那么需要进行这样的操作：

```nginx
location ~ /api/test/ {
    rewrite "^/api/test/(.*)$" /$1 break;
    proxy_pass      http://127.0.0.1:8085;
}
```
把这段配置放在之前配置的 `/api` 上面。一定要放在上面，否则你的请求会被转发到：

`http://localhost:8084/test`

就达不到我们的目的了。