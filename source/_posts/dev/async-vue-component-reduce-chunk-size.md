---
title: 异步加载 Vue 组件以减小 chunk 体积
date: 2022-11-05 23:19:39.413
updated: 2022-11-06 21:39:05.631
categories: 
- Dev
tags: 
- Vue
---

## 问题

当你的组件过于复杂时，这里指它引用了非常多的第三方库，那么当你打包的时候或许会碰到下面的警告：

```bash
(!) Some chunks are larger than 500 KiB after minification. Consider:
- Using dynamic import() to code-split the application
- Use build.rollupOptions.output.manualChunks to improve chunking: https://rollupjs.org/guide/en/#outputmanualchunks  
- Adjust chunk size limit for this warning via build.chunkSizeWarningLimit.
```

当然一个最简单的方法，就是按照它提示的最后一行，编辑 `vite.config.ts`：

```ts
export default defineConfig({
  // ...
  build: {
    chunkSizeWarningLimit: 1500,
  },
})
```

但是，既然我写了这篇文章，就证明我没有直接忽略这个警告。~~警告竟然得到了尊重，泪目了。~~

## 方案

对一些比较重的组件，尤其是需要我们从**后端获取数据后才显示的组件**，用一层  `defineAsyncComponent` 套起来，如下：

```ts
const AsyncFoo = defineAsyncComponent(() => import('./Foo.vue'))
```

之后再模板里直接当成 `Foo` 组件用就可以了：

```html
<AsyncFoo
  :prop1="val1"
  :prop2="val2"
/>
```

这是在 `Vue` 官网上专门介绍的用法，[点此前往。](
https://vuejs.org/guide/components/async.html#basic-usage)

当然，这个不能操之过急，只要拆分几个**主要的大组件**即可。

## 更多

如果你想要加载时候配置一个组件，比如骨架屏之类的，可以这样：

```ts
const AsyncFoo = defineAsyncComponent({
  loader: () => import('./Foo.vue'),
  loadingComponent: FooSkeleton,
})
```

还可以配置 `error`，`timeout` 等等，请参考[文档](https://vuejs.org/guide/components/async.html#loading-and-error-states)。
