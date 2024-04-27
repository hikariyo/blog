---
title: 拓展 marked 支持自定义表情
date: 2022-11-09 21:59:27.459
updated: 2022-11-20 22:39:43.858
categories: 
- Dev
tags: 
- JS
---

## 原理

为了方便，我只是拓展 `renderer` 里对于图片渲染的逻辑，处理我们自定义表情，官方文档地址：https://marked.js.org/using_pro#renderer

我们要把类似这样的图片：`![@](12)`，也就是方框里填 `@` 就认为它是我们的自定义表情，然后以后面的 `href` 为表情 `id`。

## 实现

```ts
// emoji.ts
export interface Emoji {
  id: number
  name: string
  url: string
}

export const EMOJI_SIZE_PX: number

export function getEmoji(id: number): Emoji | undefined

// emoji-renderer.ts
export const emojiRenderer: marked.RendererObject = {
  image(href, _, text) {
    if (text !== '@')
      return false

    const id = parseInt(href ?? '')
    const emoji = getEmoji(id)
    if (emoji === undefined) {
      // 这里可以抛一个警告
      return false
    }

    return `<img alt="${emoji.name}" src="${emoji.url}" style="width: ${EMOJI_SIZE_PX}px; height: ${EMOJI_SIZE_PX}px;vertical-align: bottom; padding: 0 2px;">`
  },
}

// main.ts
import { emojiRenderer as renderer } from 'xxx'

marked.use({ renderer })
```

这样就可以在全局使用到我们自定义的规则了。
