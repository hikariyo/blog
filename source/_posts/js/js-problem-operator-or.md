---
title: JS关于或运算符的问题
date: 2022-06-07 09:23:22.0
updated: 2022-12-10 19:51:49.656

categories: 
- Bug
tags: 
- JS
---

## 背景

这是在**AVL树**计算高度时遇到的问题。为了方便大家看到问题的本质，这里使用一个**单链表**复现问题。

我们有一个链表，并且把它的深度储存到了每一个节点里（所谓深度就是它拥有的子节点层数，对于一个链表而言就是它的**长度减一**）。

## 复现

先一个个函数来实现一个简单的链表。不直接说的原因是我想让读者带入到当时的场景，思考为什么出现问题，这样才能印象更深刻，同时我在再次回头看这篇文章的时候也能会想起当时的场景。

### 创建节点

就是返回一个对象，这没什么好说的：

```js
function createNode(val) {
    return { val, depth: 0, next: undefined }
}
```

### 获取深度

所谓深度就是它拥有的**子节点层数**，考虑链表`[ 3, 7, 4 ]`：

+ `4`的深度为`0`，
+ `7`的深度为`1`，
+ `3`的深度为`2`。

因此，前一个的深度等于后一个深度加一，所以`undefined`或`null`的深度应该是`-1`（`-1`加`1`等于`0`，这样定义的话，使最后一个节点的**计算更方便**而已）。

```js
function depth(node) {
    return node?.depth || -1
}
```

其实问题就出在这里，文末再说明为什么有问题。

### 插入

我们要实时更新每一个结点的深度，那就必然是在**节点改变时更新**（插入或者删除时）。为了简化代码，这里就只写一个插入的操作，删除就省略了。

```js
function insert(node, val) {
    if (!node) {
        return createNode(val)
    }

    node.next = insert(node.next, val)
    node.depth = depth(node.next) + 1

    return node
}
```

由于我们每次都在设置`depth`之前就递归调用了这个函数本身，直到`node.next`是空为止。所以，我们设置`depth`这个属性是**从后向前**依次进行的。

### 测试

用数组的`reduce`方法为我们生成一个链表，然后进行测试：

```js
const root = [ 3, 7, 4 ].reduce((root, val) => {
    return insert(root, val)
}, undefined)

console.log(root)
```

运行结果：

```js
{
    val: 3,
    depth: 0,
    next: {
        val: 7,
        depth: 0,
        next: {
            val: 4,
            depth: 0,
            next: undefined
        }
    }
}
```

可以看出，**每一层的深度都被设置为了0**，它的原因是当最内层的节点获取深度时，相当于：

```js
node.depth = (undefined || -1) + 1
```

然后外层进行获取时，相当于：

```js
node.depth = (0 || -1) + 1
```

由于`0`也是一个`falsy value`，所以在括号内取到了后面的`-1`，这就是导致bug的根本原因。

### 修改

我们用三目运算符代替原先的或运算符：

```js
function depth(node) {
    return node ? node.depth : -1
}
```

或者说用双问号运算符，它的用法是`a ?? b`，当`a`为`null`或`undefined`时才会取`b`，否则取`a`：

```js
function depth(node) {
    return node?.depth ?? -1
}
```

但是后者看上去问号太多，让我选择的话我会选择前者。

修改完成后再次运行测试代码，可以得到下面的结果：

```js
{
    val: 3,
    depth: 2,
    next: {
        val: 7,
        depth: 1,
        next: {
            val: 4,
            depth: 0,
            next: undefined
        }
    }
}
```

## 后记

JS中由于`null`和`undefined`的存在，我习惯用`if (a)`来判空，然而这导致了`a`是`0`、空字符串时也被误杀，所以判空时要注意自己面对的是一个**对象**还是**基本类型**。