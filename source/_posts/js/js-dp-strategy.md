---
title: JS设计模式之策略模式
date: 2022-06-02 18:48:01.0
updated: 2022-12-10 20:11:51.849

categories: 
- Learn
tags: 
- JS
---

## 意图

> 定义一系列算法， 把它们一个个封装起来，并且使它们可以相互替换。本模式使得算法可独立于使用它的客户而变化。
>
> ——《设计模式：可复用面向对象软件的基础》中文版第234页

由于在JS中，函数是一等公民，所以我们这里直接把函数当作这一个个策略对象即可。

## 示例

我们的编码目的是，根据一个人的等级把他的薪水乘不同的系数：

+ 如果是A级，就乘4。
+ 如果是B级，就乘3。
+ 如果是C级，就乘2。

不要用`if else`或者`switch case`，直接运用我们的策略模式，代码如下：

```js
const strategies = {
    levelA(salary) {
        return salary * 4
    },

    levelB(salary) {
        return salary * 3
    },

    levelC(salary) {
        return salary * 2
    }
}

function calcSalary(level, salary) {
    return strategies[level](salary)
}

console.log(calcSalary('levelA', 25000))
// 100000

console.log(calcSalary('levelB', 15000))
// 45000

console.log(calcSalary('levelC', 10000))
// 20000
```

以上。

## 参考

[设计模式：可复用面向对象软件的基础](https://book.douban.com/subject/34262305/)

[JS设计模式——策略模式](https://www.cnblogs.com/chenwenhao/p/12354176.html)