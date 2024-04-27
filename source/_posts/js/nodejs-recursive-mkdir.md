---
title: NodeJS递归创建目录
date: 2022-05-06 00:00:00.0
updated: 2022-08-10 15:27:11.542
categories: 
- Dev
tags: 
- JS
- Node
---



看到很多博主都自己写一个方法实现递归调用，其实不需要，在`v10.12.0`以上的`node`环境可以配置一个参数：

```js
fs.mkdirSync('./foo/bar', { recursive: true })
```

这样就可以直接实现递归创建文件夹了。