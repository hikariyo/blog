---
title: JS统计中英文字数
date: 2022-05-15 00:00:00.0
updated: 2022-12-10 19:30:05.158
categories: 
- Dev
tags: 
- JS
---

## 源码
> 来自半年后的说明：我把代码转放到 gists 里了，当时代码风格受 Python 影响较大，这里就不再更改了，毕竟大家本地都有自己的格式化工具。

点击[这里](https://gist.github.com/hikariyo/b838dff3bdf0d280b25f50a66abd699c)前往Github获取本文源码。

## 需求

众所周知，英文里的字数是`单词数`，而中文的字数就是`字的个数`。

那么我的第一想法自然是分开统计，之后汇总。

## 方案

匹配中文字符的正则是`[\u4e00-\u9fa5]`，因此写出下列代码：

```js
function countWords(str) {
    const chinese = Array.from(str)
        .filter(ch => /[\u4e00-\u9fa5]/.test(ch))
        .length
    
    const english = Array.from(str)
        .map(ch => /[a-zA-Z0-9\s]/.test(ch) ? ch : ' ')
        .join('').split(/\s+/).filter(s => s)
        .length

    return chinese + english
}
```

先拆开看看，我们最后调用`length`字段前的对象长什么样子：

```js
const str = 'Hello, world, 你好, 世界'
const res = Array.from(str)
    .map(ch => /[a-zA-Z0-9\s]/.test(ch) ? ch : ' ')
    .join('').split(/\s+/).filter(s => s)
console.log(res, res.length)
// [ 'Hello', 'world' ] 2
```

需求达成，再来看看中文的：

```js
const str = 'Hello, world, 你好, 世界'
const res = Array.from(str)
    .filter(ch => /[\u4e00-\u9fa5]/.test(ch))
console.log(res, res.length)
// [ '你', '好', '世', '界' ] 4
```

暂不考虑运行效率，我认为这样写出来是比较直观的了。

又不是不能跑.jpg