---
title: 记一次JSON和ts-node的坑
date: 2022-05-02 00:00:00.0
updated: 2022-08-10 15:28:58.559
categories: 
- Dev
tags: 
- TS
---

首先列出目录结构：

```
example/
 config.ts
 config.json
 app.ts
```

config.ts

```typescript
import fs from 'fs'

const conf = JSON.parse(fs.readFileSync('config.json', 'utf-8'))

export const foo = conf.foo
```

config.json

```json
{
    "foo": true
}
```

app.ts

```typescript
import { foo } from './config'

console.log('Foo is', foo)
```

此时运行`ts-node app.ts`不会有任何问题。

## 问题复现

现在我有一个新的需求，需要我第一次运行希望`foo`这个属性是`true`，然后把它修改成`false`，把`foo`这个字段当作初始化的标志。

于是把下面这段代码加到`config.ts`里面：

```typescript
export function resetFoo() {
    if (foo) {
        conf.foo = false
        fs.writeFileSync('config.json', JSON.stringify(conf))
    }
}
```

然后在`app.ts`里面加入下面这行代码：

```typescript
import { resetFoo } from './config'

resetFoo()
// TypeError: (0 , config_1.resetFoo) is not a function
```

那么我的第一反应是看一下`resetFoo`是个什么东西，就在`app.ts`出错的那句话上面加上了：

```typescript
console.log('ResetFoo is', resetFoo)
// ResetFoo is undefined
```

## 解决方案

我在Stack Overflow上面查到的是[这里](https://stackoverflow.com/questions/45909695/ts-node-import-not-defined-at-runtime)。

长话短说，这个问题的根本原因在于`app.ts`的第一行：

```typescript
import { foo, resetFoo } from './config'
```

它导入的其实是`config.json`，和我们写的ts没有半毛钱关系。

话是这样说，但是如何验证呢? 只需要把整个`config.ts`内容清空，只留下一句：

```typescript
export const foo = 'THIS IS NOT JSON FILE!'
```

然后把`app.ts`改成：

```typescript
import { foo } from './config'

console.log('Foo is', foo)
// Foo is true
```

所以说解决方案就很明显了：把`config.ts`改名。由于这个操作过于简单这里就不演示了。

## 后记

虽然它能给我出bug，但是如果没了`config.ts`我还不能直接导入JSON文件，需要在`tsconfig.json`的`compilerOptions`里面加上一句：

```json
"resolveJsonModule": true
```

然后引入的时候需要这样引：

```typescript
import config from './config.json'
// or
import { foo } from './config.json'
```