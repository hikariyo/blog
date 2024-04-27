---
title: 基于JSON文件的轻量级数据库
date: 2022-06-08 21:43:29.0
updated: 2022-08-10 15:11:52.052

categories: 
- Dev
tags: 
- JS
- TS
---



## 仓库

仓库地址：[Github仓库](https://github.com/hikariyo/json-file-database)

## 安装

已经发布到npm仓库，可以用`npm`、`yarn`、`pnpm`等等安装：

```bash
npm i json-file-database
```

## 介绍

轻量级仓库，其内部实现原理就是把数组包装了一下，使得源数据改变时自动保存。

下面是Github页面上给出的示例代码，这英语也比较简单就不翻译了，毕竟是我写的Chinglish。

```typescript
import { connect } from 'json-file-database'

/**
 * The type of elements must have a `id` property
 * to make them unique and sorted in the collection.
 */
type User = { id: number, name: string }

/**
 * Connect the database.
 * If there is not the file yet, it will create one after running this program.
 */
const db = connect({
    file: './db.json',
    init: {
        users: [
            { id: 1, name: 'San Zhang' },
            { id: 2, name: 'Si Li' },
            { id: 3, name: 'Wu Wang' },
        ]
    }
})

/**
 * Specify the type of elements is User.
 * 
 * You can go to the documentations to see how to customize all
 * options, including the `comparator` to compare the elements.
 */
const users = db<User>('users')

/**
 * Find the element with its id.
 */
console.log('The user whose id is 1:', users.find(1))


/**
 * Find all elements that match given condition.
 */
console.log('All users whose id <= 2 are:', users.findAll(u => u.id <= 2))


/**
 * Check whether this collection has the element.
 */
console.log('Whether the collection has a user whose id is 5:', users.has(5))


/**
 * Insert an element and return whether it has been inserted.
 */
console.log('Insert a user whose id is 2:', users.insert({ id: 2, name: 'Liu Zhao' }))


/**
 * List all elements.
 * 
 * You can also use `[...users]` or `for...of` because
 * it has implemented Iterable<E>.
 */
console.log('All users are:', Array.from(users))


/**
 * Remove the element and return whether it has been removed.
 */
console.log('Remove the user whose id is 1:', users.remove(1))


/**
 * Remove all elements that match the condition, and return the number of them.
 */
console.log('Remove all users whose id < 3, the number of them is:',
        users.removeAll(u => u.id < 3)) 


/**
 * Update the element with id, and return whether it has been updated.
 */
console.log('Update the user whose id is 3:', users.update(3, { name: 'Liu Zhao' }))
```

## 特色

提供两种实现，基于数组和AVL树。

除了使用函数作为条件，导致的无法比较对象之间的大小的操作之外，所有的操作的最优时间复杂度都是`O(log n)`。对于数组来说，我们使用二分搜索来查找；对于AVL树来说，其实也是类似于数组的二分搜索。

它们唯一不同的地方就在于，数组在频繁插入时的复杂度就很差了。但是通常来说，我们的程序并不会频繁的进行插入操作，所以我定义的默认类型是数组，而非AVL树。

如果你想使用AVL树的实现，在创建`Collection`的时候指定一下即可：

```typescript
const users = db<User>({
    name: 'users',
    type: 'avl'
})
```

此外，我还使用了**函数防抖**来保存编辑过的数据，如下：

```typescript
function getDataSaver(data: JSONData, delay: number, file: DatabaseFile, onSaved: () => void) : Save {
    let timeout: NodeJS.Timeout | undefined
    return (name, elements) => {
        clearTimeout(timeout)
        timeout = setTimeout(() => {
            data[name] = elements()
            file.write(JSON.stringify(data))
            onSaved.apply(undefined)
        }, delay)
    }
}
```

这样一来，频繁的数据更新就不会引起频繁的IO操作了。只有当数据更新停止一段时间后它才会真正地把数据写入硬盘。