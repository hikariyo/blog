---
title: 开发 - 记一次前端多语言配置
date: 2022-04-16 00:00:00.0
updated: 2022-12-10 19:03:54.479
categories: 
- Dev
tags: 
- JS
---

## Dataset
首先介绍一下向标签中添加自定义数据的方法。

可以通过添加`data-foo`属性在HTML的标签上，如：

```html
<div id="id" data-foo="bar"></div>
```

然后就可以在JS里通过`DOM.dataset.foo`获取到属性的内容了：

```js
const dom = document.getElementById('id')
console.log(dom.dataset)
console.log(dom.dataset.foo)
```

输出：

![Chrome输出](/img/2022/08/image-20220501202916587.png)

特别地，如果在HTML中添加如`data-foo-bar`的属性，在JS中需要通过camelCase的形式访问到值：

```js
dom.dataset.fooBar
```

## JSON资源文件

我的想法是把所有的文本都写到`json`文件里，之后进行文本填充操作。

+ `assets/zh.json`
  
  ```json
  {
      "msg": "你好"
  }
  ```

+ `assets/en.json`
  
  ```json
  {
      "msg": "Hello"
  }
  ```

## JS逻辑

首先要获取到我们配置好的`json`，这里为了方便直接把`navigator.language`截取了。

而实际应用中可能需要配置简体`zh-CN`和繁体`zh-TW`，至于文件名怎么取就不是本文重点了。

```js
async function getI18N() {
    const lang = navigator.language.substring(0, 2)
    
    const res = await fetch(`assets/${lang}.json`)

    return await res.json()
}

getI18N().then(data => {
    document.querySelectorAll('*[data-i18n]').forEach(el => {
        el.innerText = data[el.dataset.i18n]
    }
})
```

再接下来就只需要在标签里添加`data-i18n`属性并且把键对应配置完毕即可。

示例数据的键为`msg`，那么这里需要这么写：

```html
<h1 data-i18n="msg"></h1>
```

完工!