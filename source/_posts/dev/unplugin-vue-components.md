---
title: 开发 - 关于 unplugin-vue-components 代码提示问题
date: 2022-04-30 00:00:00.0
updated: 2022-08-10 15:40:02.076

categories: 
- Dev
tags: 
- Vue
- VSCode
---



今天第一次体验`unplugin-vue-components`，遇到了代码提示的问题(VS Code)，如下：

 ![VSCode飘红](image-20220430160849222.png)

这里明显是由于`Volar`插件没有解析到`RouterLink`是何方神圣导致的。当然这不是`Volar`的问题，只是我没有配置好。

## 解决方案

其实作者本人在[Readme.md](https://github.com/antfu/unplugin-vue-components#typescript)里已经提示过这个问题了，只是我没有仔细去看。

 ![Github下的README.md](image-20220430161209538.png)

那么问题就基本上解决了。上面已经说了如果说`typescript`(应该指的是`npm`包)安装后，`dts`的配置项就会自动生效，所以这里我们只需要关心下面说的`tsconfig.json`。在`"include"`字段下(这里我不知道为什么作者写的是`includes`，或许是版本不一样) 添加`"components.d.ts"`，如：

```json
"include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue", "components.d.ts"]
```

这前面都是默认生成的配置项，新添加的只是最后一项。

之后再次启动`npm run dev`，等待片刻(它需要响应时间)，我这里就正常提示了。

 ![现在VSCode成功解析了](image-20220430161604391.png)

## 补充

如果使用框架如`navie ui`，还有一个配置项可以更好的让`Volar`给出提示。

以`naive ui`为例，在`tsconfig.json`的`"compilerOptions"`中添加：

```json
"types": ["naive-ui/volar"]
```

大功告成。