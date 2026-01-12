---
title: Vue Router 搭配 NaiveUI 的进度条
date: 2022-10-04 02:20:03.262
updated: 2022-10-04 02:56:51.968
categories: 
- Dev
tags: 
- Vue
---

## 介绍

由于 naive ui 的进度条需要依赖于一个 `NLoadingBarProvider`，所以我们在定义 `router` 的时候是无法使用 `useLoadingBar()` 获取到进度条对象的。

我的解决方法是利用 `pinia` 把 `loadingBar` 示例获取到，然后存起来，这样就可以访问到了。

这里用 `pinia` 的原因单纯是因为它简单好用，并且还能在浏览器插件里看到变量的值，而单纯的全局变量是没有这些优势的。但是，用跨文件全局变量也未尝不可，不过我这里就不作验证了。

## 环境

除了常规 `pnpm create vite` 外，还需要安装以下：

```bash
pnpm add naive-ui vue-router pinia
```


顺便把 `main.ts` 也给出来：

```ts
// src/main.ts
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import { createPinia } from 'pinia'

createApp(App).use(createPinia()).use(router).mount('#app')
```

## 创建 Store

一个简单的 `pinia store`：

```ts
// src/stores/provider.ts
import { defineStore } from 'pinia'
import { ref } from 'vue'
import { LoadingBarApi } from 'naive-ui'

export const useProviderStore = defineStore('provider', () => {
    const loadingBar = ref<LoadingBarApi>()

    function setLoadingBar(b: LoadingBarApi) {
        loadingBar.value = b
    }

    return { loadingBar, setLoadingBar }
})

```

## 获取加载条

由于 `NLoadingBarProvider` 的存在，我们不可以直接在 `App.vue` 里使用 `useLoadingBar()`，而应该再创建一个组件，并且把它用 `NLoadingBarProvider` 包起来才可以。

为了简化代码，我使用 `defineComponent` 直接内嵌一个了，代码如下：

```html
<script setup lang="ts">
// src/App.vue
import { NLoadingBarProvider, useLoadingBar } from 'naive-ui'
import { defineComponent, h } from 'vue'
import { RouterView } from 'vue-router'
import { useProviderStore } from './stores/provider'

const provider = useProviderStore()

const ViewComponent = defineComponent({
  render: () => h(RouterView),
  setup: () => {
    provider.setLoadingBar(useLoadingBar())
  }
})
</script>
  
<template>
  <NLoadingBarProvider>
    <ViewComponent/>
  </NLoadingBarProvider>
</template>
```

## 在 Router 中使用

注意我们要在 `beforeEach` 和 `afterEach` 的函数体里调用 `useProviderStore()`，否则的话会报 `pinia` 未创建的错误。

```ts
// src/router/index.ts
import { createRouter, createWebHashHistory, RouteRecordRaw } from 'vue-router'
import { useProviderStore } from '../stores/provider'

const router = createRouter({
  routes: [
    {
      path: '/',
      component: import('../Test.vue')
    }
  ],
  history: createWebHashHistory('/')
})

router.beforeEach(() => {
  useProviderStore().loadingBar?.start()
})

router.afterEach(() => {
  useProviderStore().loadingBar?.finish()
})

export default router
```
