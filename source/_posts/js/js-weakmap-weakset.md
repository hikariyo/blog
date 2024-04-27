---
title: JS中的WeakMap与WeakSet
date: 2022-06-10 16:17:20.0
updated: 2022-12-10 19:46:29.415

categories: 
- Learn
tags: 
- JS
---

## 介绍

`WeakMap`与`WeakSet`都是一种优化使用内存的解决方案。这两个数据结构的引用不会导致这些对象不被回收。上来就说这些有点太枯燥了，还是先聊聊它们能干什么。


## WeakSet

集合就是一堆互异的数据，想必这个读者都早就明白了，这里不多说。

相对于正统`Set`，`WeakSet`只提供三个方法：`add`、`delete`和`has`。正如前文所说，它们的引用GC是不管的，所以它也不清楚自己到底引用了多少个对象，故只能判断这个对象存在与否，和增删对象。

说了这么多，还是来点代码：

```js
const set = new WeakSet()

let mike = { name: 'Mike', age: 17 }
let john = { name: 'John', age: 18 }

set.add(john).add(mike)

john = undefined
```

当运行完这段代码的时候，`john`已经不再引用原来的对象，此时存放于`set`中的原`john`对象就可以被删掉了。但是到底删不删还是由JS引擎说得算的。

既然它对于**对象**都是弱引用，那么它就不能添加一个不是对象的基本类型。

```js
set.add(1)
// TypeError: Invalid value used in weak set
```

## WeakMap

上文提到的`WeakSet`，我是不知道它到底被广泛应用到哪里了。但是`WeakMap`有一个特别有用的地方：存储私有变量。

我们知道，JS中没有真正的私有，但是我在使用TS编译器就运用了`WeakMap`来储存所谓的私有变量。这里演示一下其原理，因为TS编译后的代码不是人看的：

```js
const barStorage = new WeakMap()

class Foo {
    constructor(bar) {
        barStorage.set(this, bar)
    }

    get bar() {
        return barStorage.get(this)
    }

    set bar(_) {
        throw new Error('cannot set private member!')
    }
}

const foo1 = new Foo(2)
const foo2 = new Foo(3)

console.log(foo1.bar, foo2.bar)
// 2 3

foo1.bar = 21
// Error: cannot set private member!
```

由于`WeakMap`对对象的引用不会被GC当回事，所以当我们`foo1`或`foo2`使用完毕被回收后，它们在`WeakMap`里的引用也会被清除掉，这样就节约了内存。

顺带一提，同`WeakSet`一样，`WeakMap`的键必须是对象：

```js
barStorage.set(1, 2)
// TypeError: Invalid value used as weak map key
```

这样就保证了当作为键的对象被回收后，它存的值也会被回收。