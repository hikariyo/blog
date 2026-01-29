---
title: 开发 - 为 Hexo 页脚添加实时更新的运行时间
date: 2023-05-27 21:39:26
updated: 2023-05-27 21:39:26
tags:
  - JS
categories: Dev
---

## 配置文件

本文用 `Dayjs` 来处理日期。当然，完全可以用原生 `Date`，但我不想把时间浪费在处理日期上面。

我使用的是 `butterfly` 主题。如果读者正在使用别的主题，请查阅对应官方文档寻找如何插入自定义标签与自定义页脚信息。

在主题配置文件（即`_config.butterfly.yml`）中引入：

```yaml
inject:
  bottom:
    - <script src="https://cdn.jsdelivr.net/npm/dayjs@1.11.7/dayjs.min.js"></script>
    - <script src="https://cdn.jsdelivr.net/npm/dayjs@1.11.7/plugin/duration.min.js"></script>
    - <script src="/scripts/realtime.js"></script>
```

在自定义页脚中加入一个 `span` 元素提供日期的挂载位点（hhh 最近做过 tRNA 结合位点的题，觉得这个描述还挺贴切）：

```yaml
footer:
  custom_text:
    <span id="realtime_duration"></span>
```

接下来主要通过 `setInterval` 实现这样一个**实时更新**的功能。

## 实现功能

在 `source` 目录下创建 `scripts/realtime.js`，写入以下代码：

```js
(() => {
  dayjs.extend(window.dayjs_plugin_duration)
  const el = document.getElementById('realtime_duration')
  // 改成自己的时间
  const date = dayjs('2022-05-19')

  setInterval(() => {
    const dur = dayjs.duration(dayjs().diff(date))
    const days = String(Math.floor(dur.asDays()))
    el.innerHTML = '已运行' + days + dur.format('天HH时mm分ss秒')
  }, 1000)
})()
```

现在拉到页底就能看到效果。

当然，如果是用 RSS 阅读器或者别的搬运站的话肯定就看不到了。
