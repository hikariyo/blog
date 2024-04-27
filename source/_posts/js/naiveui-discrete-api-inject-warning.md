---
title: Naive UI 使用独立 API 后警告 inject
date: 2022-11-05 00:00:14.251
updated: 2022-11-05 00:00:14.251
categories: 
- Bug
tags: 
- Vue
---

## 解决方法

抽离出来一个单独的文件，用来储存 `pinia` 对象，如下：

```ts
import { createPinia } from 'pinia'

export const pinia = createPinia()
```

注意 `main.ts` 中的 `app.use` 也要用这个单独的文件暴露的 `pinia` 变量（也就是跨文件全局变量），之后再在调用 `createDiscreteApi` 的地方改成下面这种形式：

```ts
const { message, notification, dialog, loadingBar, app } = createDiscreteApi(
  ['message', 'dialog', 'notification', 'loadingBar'],
)

app.use(pinia)
```

## 现象及原因

报出的警告类似下面的形式：

```
[Vue warn]: injection "Symbol(pinia)" not found. 
  at <...>
```

那么我在用 Devtools 的时候观察到了两个 `App` ：

![两个 App](/img/2022/11/discrete-api-2apps.jpg)

这时候我才意识到下面这个是由我使用独立 API 而创建的，才找到了引发这个警告的原因。

所以说 Devtools 还是很有用的。就这样，拜拜。
