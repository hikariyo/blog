---
title: TS中keyof和typeof
date: 2022-05-21 00:00:00.0
updated: 2022-08-10 15:24:22.487

categories: 
- Learn
tags: 
- TS
---



这两个东西是用来进行类型推断的。

在JS中，如果想要动态访问属性，很容易写出下面的代码：

```js
function getProp(obj, key) {
    return obj[key]
}
```

但是TS呢? 如果使用`any`大法，就写出下面这样的代码了：

```typescript
function getProp(obj: any, key: string) {
    return obj[key]
}
```

这样固然可以，但是指不定哪天就蹦出来一个`undefined`，就不能体现出TS的优势了。

## 方案

使用`keyof`解决这种问题：

```typescript
function getProp<T>(obj: T, key: keyof T) {
    return obj[key]
}

console.log(getProp([1, 2, 3], 'length'))
// 3
```

## 进阶

那么我有下面这样的代码：

```typescript
const Colors = {
    Red: '#FF0000',
    White: '#FFFFFF'
} as const

function getColor(key: string) {
    return (Colors as any)[key]
}

console.log(getColor('Red'))
```

注意上面的`as const`，这是定义一个常量字面量，也就是告诉TS编译器这个玩意后面不会再改，方便它进行类型推断。

我的目的就是，把那句`Colors as any`改掉，如果想用`keyof`的话，那么我们起码要获取到`Colors`的类型，但是这里他是一个字面量对象，所以本文要提到的另一个东西就引出来了，它就是`typeof`。

注意这里的`typeof`在原生JS里依然存在，用来获取一个变量的类型。但是TS中的`typeof`还有新的用途，那就是获取一个变量的类型并且能够用它声明新的变量：

```typescript
type Color = typeof Colors

type Color2 = {
    Red: string
    White: string
}
```

上面代码中`Color`和`Color2`是完全等价的。

所以我们的`getColor`方法可以这样写：

```typescript
function getColor(key: keyof Color) {
    return Colors[key]
}
```

综合一下，变成下面这样：

```typescript
function getColor(key: keyof typeof Colors) {
    return Colors[key]
}
```

## 再进阶

如果我想让输入`#FF0000`这样的字符串并且把`Red`返回去，那么在不改变`Colors`对象的基础上，应该怎么办呢?

答案是下面这样：

```typescript
const Colors = {
    Red: '#FF0000',
    White: '#FFFFFF'
} as const

type Color = typeof Colors

function getColorName(hex: Color[keyof Color]) {
    return Object.entries(Colors).find(item => item[1] === hex)![0]
}

console.log(getColorName('#FF0000'))
// Red
```

一定要有`as const`，不然TS编译器只能推导出`Color[keyof Color]`是`string`类型，而不是确切的`#FF0000`这种。

除此之外，`Object.entries`方法会把对象的键值对逐个返回，变成一个`[key，value]`的数组，所以这里的`[1]`代表着值，`[0]`代表着键。