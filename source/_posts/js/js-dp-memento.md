---
title: JS设计模式之备忘录模式
date: 2022-06-02 20:11:45.0
updated: 2022-12-10 20:10:12.675
categories: 
- Learn
tags: 
- JS
---

## 意图

> 在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可将该对象恢复到原先保存的状态。
>
> ——《设计模式：可复用面向对象软件的基础》中文版第212页

这个常常被运用于游戏存档、操作撤销等场景。

## 示例

这是一个悲催的故事，内容是一个人先涨薪然后通过本文提到的功能恢复了原来的薪水。如下：

```js
class Staff {
    constructor(name, salary) {
        this.name = name
        this.salary = salary
    }

    raiseSalary(salary) {
        this.salary += salary
    }

    store() {
        const memento = JSON.stringify(this)
        return memento
    }

    restore(memento) {
        Object.assign(this, JSON.parse(memento))
    }
}

class StaffCareTaker {
    mementos = {}

    set(key, memento) {
        this.mementos[key] = memento
    }

    get(key) {
        return this.mementos[key]
    }
}

const careTaker = new StaffCareTaker()
const staff = new Staff('Bob', 12500)

careTaker.set(1, staff.store())

staff.raiseSalary(5000)
console.log(staff.salary)
// 17500

staff.restore(careTaker.get(1))
console.log(staff.salary)
// 12500
```

+ `Memento`指的是存储下来的数据，本文中它是一个JSON字符串。
+ `CareTaker`负责保存数据，它不能操作这些数据。
+ `Originator`，即本文的`Staff`，是一个可以保存状态并恢复状态的对象。

基本的模型就是上面三个。

## 参考

[设计模式：可复用面向对象软件的基础](https://book.douban.com/subject/34262305/)

[JavaScript Memento](https://www.dofactory.com/javascript/design-patterns/memento)