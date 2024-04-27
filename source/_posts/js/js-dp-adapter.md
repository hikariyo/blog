---
title: JS设计模式之适配器模式
date: 2022-06-02 15:43:45.0
updated: 2022-12-10 20:05:28.486
categories: 
- Learn
tags: 
- JS
---

## 意图

> 将一个类的接口转换成客户希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。
>
> ——《设计模式：可复用面向对象软件的基础》中文版第106页

值得注意的是，在JS中，我们对于数据类型的适配（把后端传来的JSON转成我们希望的格式）也应属于适配器的范畴，所以下文中分别叙述它们的应用。

## 示例

这两个例子都写在了同一个文件里。

### 适配方法

假设我们有谷歌地图和百度地图的API，但是它们俩提供的方法名字不同，如下：

```js
const googleMap = {
    show() {
        console.log('Shows google map')
    }
}

const baiduMap = {
    display() {
        console.log('Displays baidu map')
    }
}

const baiduMapAdapter = {
    show() {
        baiduMap.display()
    }
}

function showMap(map) {
    map.show()
}

showMap(googleMap)
// Shows google map

showMap(baiduMapAdapter)
// Displays baidu map
```

对于不同的`display`和`show`方法，我们通过为`baiduMap`定义一个适配器来使其支持被调用`show`，从而配合`showMap`方法。

### 适配类型

除了传统的适配相同方法外，在JS中还经常做适配不同类型的操作，如下：

```js
function bookAdapter(book) {
    return {
        name: book[0],
        author: book[1],
        price: book[2]
    }
}

const book = bookAdapter(['BookName', 'BookAuthor', 50])
console.log(book)
// { name: 'BookName', author: 'BookAuthor', price: 50 }
```

这样就可以把后端传来的数据处理成我们想要的形式了。虽然它与传统意义上的适配器有区别，但我认为两者的思想是相同的，故这里也展示一下。

## 参考

[设计模式：可复用面向对象软件的基础](https://book.douban.com/subject/34262305/)

[JavaScript设计模式——适配器模式](https://www.cnblogs.com/dengyao-blogs/p/11703049.html)

[JS 适配器模式](https://segmentfault.com/a/1190000012436538)