---
title: 在 GitHub Pages 中使用 Vue Router 
date: 2023-01-25 16:43:45.324
updated: 2023-01-25 16:45:53.612
categories: 
- Bug
tags: 
- Vue
- GitHub
---

本文主要是由于这个[倒计时项目](https://github.com/hikariyo/countdown)以路径参数的形式接收自定义日期，出于美观的原因我不想用 `hash router`，而是使用 `history router` 。如果用 `hash router` 就不会有这个问题了。~~此文终结~~

GitHub Pages 只能帮你生成一个静态网站，但它支持用 `404.html` 作为一个 `fallback`（原谅我不知道怎么用中文描述比较贴切）利用这个机制，和 `sessionStorage` 或者 `localStorage` 就可以实现我们的目的。

## 404.html

可以把它放到项目中的 `/public` 目录，或者放到你为 GitHub Pages 准备的分支里面，我个人选择 `/public`，方便管理。

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>YOUR TITLE</title>
    <script type="text/javascript">
        sessionStorage.setItem('redirect', location.pathname)
        location.href = '/'
    </script>
</head>
<body>
</body>
</html>
```

如果你的 `GitHub Pages` 没有用自定义域名，就是说它的链接是 `https://foo.github.io/bar`，就要把代码里的 `/` 替换成 `/bar`。至于具体是用 `localStorage` 还是 `sessionStorage` 这个随你了。


## App.vue

这里只贴出来代码部分，模板不再展示，如下：

```typescript
import { onBeforeMount } from 'vue'
import { useRouter } from 'vue-router'
import { useSessionStorage } from '@vueuse/core'

const router = useRouter()
const redirect = useSessionStorage('redirect', '')

onBeforeMount(async () => {
  if (redirect.value) {
    await router.push(redirect.value)
    redirect.value = ''
  }
})
```

这里我只是从逻辑上认为，应该是挂载前先加载到正确的路径，所以使用了 `onBeforeMount` 而不是 `onMounted`，具体选哪个看你了。

如果不想用 `vueuse` 那就直接操作 `sessionStorage` 或者 `localStorage` 对象，不再赘述。

## 总结

在 `404.html` 里保存了当前的路径，通过 `sessionStorage` 或者 `localStorage` 传给 `Vue`，之后在 `Vue` 中判断，如果存在就加载到这个路径，从而达到使用 `history router` 的目的。
