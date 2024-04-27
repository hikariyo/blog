---
title: JS写睡眠排序
date: 2022-05-09 00:00:00.0
updated: 2022-12-10 19:13:27.2

categories: 
- Learn
tags: 
- JS
---

## 实现

代码看起来很简单，这里就直接给出来了：

```js
function sleepSort(nums) {
    const result = []
    return new Promise(resolve => {
        for (const num of nums) {
            setTimeout(() => {
                result.push(num)
                if (result.length === nums.length) {
                    resolve(result)
                }
            }, num * 100)
        }
    })
}


sleepSort([4, 1, 3, 2, 9]).then(console.log)
// [ 1, 2, 3, 4, 9 ]
```

## 原理

就是说当前数字是多少，就在多少*100毫秒后添加到`result`数组。不过从这个例子里也能小小练习一下`Promise`的用法。

这里要乘100的原因是防止几毫秒差别太小被抢占。

同时，得益于`Promise`，我们在异步函数里可以这样写：

```js
(async () => {
    const data = await sleepSort([4, 5, 3, 6, 8])
    console.log(data)
    // [ 3, 4, 5, 6, 8 ]
})()
```

Just for fun.