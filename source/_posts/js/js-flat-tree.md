---
title: 扁平结构和树形结构相互转化
date: 2022-05-29 21:27:33.0
updated: 2022-12-10 20:03:41.357
categories: 
- Dev
tags: 
- JS
- Algo
---

## 背景

假设我们有一堆评论的数据需要存储，通常来说数据库中是上面的扁平形式，而我们显示出来应该是树形结构。

于是就有了这里的内容，扁平结构和树形结构相互转换。

## 扁平转树状

如下，内容都在注释里：

```js
const flatComments = [
    { id: 1, content: 'A', parent: -1 },
    { id: 2, content: 'B', parent: -1 },
    { id: 3, content: 'C', parent: 1 },
    { id: 4, content: 'D', parent: 3 },
    { id: 5, content: 'E', parent: 2 },
]

function toTree(arr) {
    const root = []

    // 复制整个数组，使得后续操作不会影响到原始数据
    arr = arr.map(item => ({ ...item }))

    // 把对象的id作为键，本身作为值存起来
    const map = arr.reduce((pre, cur) => {
        pre[cur.id] = cur
        return pre
    }, {})

    // 这样在获取的时候，如果是-1，就会获取到root，否则就是指定的item
    map[-1] = { children: root }

    arr.forEach(item => {
        // 这里是如果map[item.parent].children是undefined，就把它赋值成空数组
        // 然后获取到map[item.parent].children并把它赋值给一个变量
        // 因为它是引用类型，所以在这个变量上调用push可以影响到原来的数组
        const children = map[item.parent].children ||= []
        children.push(item)
    })

    return root
}

console.log(toTree(flatComments))
```

读者可以打开控制台试一试，看看是否为预期效果。

## 树状转扁平

其实这个要简单得多，直接递归调用即可：

```js
const treeComments = [
    {
        id: 1,
        content: 'A',
        children: [
            {
                id: 3,
                content: 'C',
                children: [{ id: 4, content: 'D', children: [] }]
            }
        ]
    },
    {
        id: 2,
        content: 'B',
        children: [{ id: 5, content: 'E', children: [] }]
    }
]

function toFlat(tree) {
    const result = []

    // 这个函数把给定元素和它的子元素都添加到result数组中
    const convert = ({ id, content, children }, parent) => {
        // 这里添加的是一个新对象，使得后续对返回值的操作不会影响原始数据
        result.push({ id, content, parent })
        // 递归调用这个函数，把每个子元素都添加到result数组
        children.forEach(child => {
            convert(child, id)
        })
    }

    // 由于传入的tree也是一个数组，遍历它调用append函数
    tree.forEach(root => {
        convert(root, -1)
    })
    
    return result
}

console.log(toFlat(treeComments))
```

读者可以打开控制台试一试，看看效果如何。

## 后记

如果要追求效率，不必像本文中一样拷贝原始数据，直接在原对象上修改即可。

不过考虑到诸多bug都是由于对象引用混乱造成的，所以在写代码的时候需要注意这一点。