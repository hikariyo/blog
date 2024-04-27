---
title: NodeJS监视文件
date: 2022-05-01 00:00:00.0
updated: 2022-08-10 15:29:23.848
categories: 
- Learn
tags: 
- JS
- Node
---

对于如何监视文件更改这个问题，我第一时间还是想到了Node内置的`fs`库，果然发现了有这样一个操作。

## fs

使用`fs.watch`来做到最基本的监视文件，这里先写出来一个最简单的示例：

```js
const fs = require('fs')

fs.watch('./file.txt', {}, (event, filename) => {
    console.log(`File: ${event}, ${filename}`)
})
```

在运行之前，现在当前目录下创建一个`file.txt`，然后我打开了`file.txt`，写了几个字保存，看到了这些输出：

```
File: change, file.txt
File: change, file.txt
```

然后把它删掉，看到这些输出：

```
File: rename, file.txt
```

我明明是删掉了，为什么告诉我是改名？还有我明明是改了一下，却给我反回来了两个输出？

这些问题就是下面要说的。

## 原理

那么通过查源码我发现了，对于第一个我们接到的`event`对象，是这样定义的：

```typescript
export type WatchEventType = 'rename' | 'change'
```

所以说，在node眼里他就只有改名和修改两种事件。

其次，因为有些编辑器在做修改工作的时候是把文件内容都清了之后再写入当前文件，所以它监听到了两个`change`事件。

那么怎么解决呢? 其实是有方法的。

## 不依赖第三方库

借助`fs.stat`或者`fs.statSync`获取文件的状态对象，然后可以进行一系列的判断来确定到底发生了什么事情。

不过这不是本文所要探讨的内容，这里只说一下这个思路。

## 使用chokidar

这里贴出来它的[Github仓库地址](https://github.com/paulmillr/chokidar)。

第一步自然是安装：

```bash
npm install chokidar
```

它的API比较简洁，函数只有一个`watch`，返回一个`FSWatcher`对象：

```typescript
export function watch(
  paths: string | ReadonlyArray<string>,
  options?: WatchOptions
): FSWatcher
```

那么对于这个可选的`options`对象，先把代码贴上来，再分析：
```js
const watcher = chokidar.watch('./file.txt', {
    depth: 0,
    ignored: /(^|[\/\\])\../, // ignore dotfiles
})
```
这里我只用到了两个配置项，至于更多的大家可以翻阅文档：

+ `depth` 指的就是监测的文件夹深度了。这里我只需要监视当前文件夹，所以填0。
+ `ignored` 是一个正则，用来匹配忽略的文件。这里写的是官方用来匹配点开头的文件的正则。

那么对于这个`watcher`对象，我们就可以做一个监视的操作了：
```js
watcher.on('all', (event, path) => {
    console.log(`File: ${event}, ${path}`)
}).on('ready', () => console.log('Ready'))
```
此时直接运行，会发现输出了这样的内容：
```
File: add, file.txt
Ready
```
可以注意到，它先检测到了有一个添加的事件，然后才响应到了`ready`。
如果说我们不需要它的初始化，只要这样做就行（我不知道它的原生API支不支持取消初始化，这里是自己添加的逻辑）：

```js
let ready = false

watcher.on('all', (event, path) => {
    if (ready) {
        console.log(`File: ${event}, ${path}`)
    }
}).on('ready', () => {
    ready = true
    console.log('Ready')
})
```
做一个简单的状态检查，就可以达到我们的效果了。