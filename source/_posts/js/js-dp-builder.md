---
title: JS设计模式之建造者模式
date: 2022-06-02 09:42:26.0
updated: 2022-12-10 20:06:17.076
categories: 
- Learn
tags: 
- JS
---

## 意图

> 将一个复杂对象的构建与它的表示（即实例对象）分离，使得同样的构建过程可以创建不同的表示
>
> ——《设计模式：可复用面向对象软件的基础》中文版第74页

这句话里包含着两个要素：第一个是分离构建与表示，也就是说我们要单独创建一个新的类`XxxBuilder`来进行构建操作；第二个是这个`XxxBuilder`可以被继承然后重写方法，使得调用同样的接口却创建了不同的对象。下面通过一段代码来解释清楚。

## 示例

我们创建一个`BookBuilder`用来构建一个书对象，再拓展一个`ShopBookBuilder`用来构建出售的书对象（价格是1.5倍），代码如下：

```js
class BookBuilder {
    name = ''
    author = ''
    price = 0

    withName(name) {
        this.name = name
        return this
    }

    withPrice(price) {
        if (price < 0) {
            throw new RangeError('price must be positive number')
        }
        this.price = price
        return this
    }

    withAuthor(author) {
        this.author = author
        return this
    }

    build() {
        const { name, price, author } = this
        return { name, price, author }
    }
}

class ShopBookBuilder extends BookBuilder {
    withPrice(price) {
        return super.withPrice(price * 1.5)
    }
}

function buildBook(builder) {
    return builder
        .withAuthor('Author')
        .withName('Name')
        .withPrice(50)
        .build()
}

const book1 = buildBook(new BookBuilder())
console.log(book1)
// { name: 'Name', price: 50, author: 'Author' }

const book2 = buildBook(new ShopBookBuilder())
console.log(book2)
// { name: 'Name', price: 75, author: 'Author' }
```

这样我们可以通过给`buildBook`传不同的`Builder`来得到不同的结果了。

此外，还有一种JS中特有的方式，如下：

```js
function ajax({ method, url, success }) {
    const xhr = new XMLHttpRequest()
    xhr.open(method, url)
    xhr.send()

    xhr.onreadystatechange = () => {
        success(xhr.response)
    }
}

ajax({
    method: 'GET',
    url: 'https://www.baidu.com',
    success: data => {
        console.log(data)
    }
})
```

这也是将一个复杂的对象分解成了多个简单的对象进行构建，虽然看上去不是那么正统，但毕竟它也是开发中非常常见的一个形式。

## 参考

[设计模式：可复用面向对象软件的基础](https://book.douban.com/subject/34262305/)

[Using the Builder Pattern in JavaScript (With Examples)](https://www.sohamkamani.com/javascript/builder-pattern/)