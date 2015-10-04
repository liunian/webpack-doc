# webpack 是什么

**webpack** 是一个**模块打包器**。

webpack 处理带有依赖关系的模块，生成一系列表示这些模块的静态资源。

![](assets/what-is-webpack.png)

## 为什么再造一个模块打包器

现有的模块打包器并不适用于大项目（如大的单页应用）。最重要的因素是，[代码拆分][Code Spliting]和把静态资源无缝接入模块化。

尝试过扩展已有模块打包器，但无法达成下面所有的目标。

## 目标

- 把依赖树拆成可按需加载的块
- 让初始化加载时间尽可能地少
- 每个静态资源都是一个模块
- 模块化集成第三方库
- 尽可能地自定义打包器的每一部分
- 适合大项目

## webpack 的特别之处

### [代码拆分][Code Spliting]

webpack 的依赖树中有同步和异步两种依赖方式。其中，异步模块将会被拆成一个新的块，并且在被优化后，生成一个对应的文件。

更多参考[代码拆分][Code Spliting]。

### [加载器][Loaders]

webpack 本身只支持处理 JavaScript，但可以通过加载器来把别的资源转为 JavaScript。因此，每个资源都被当作一个模块。

更多参考[使用加载器][Using loaders]和[加载器][Loaders]。

### 智能解析

webpack 有一个基本支持所有第三方库的智能解析器，甚至还支持带有表达式的依赖表述法，如 `require("./templates/" + name + ".jade")`。支持最常用的 [CommonJs][CommonJs] 和 [AMD][AMD] 这两种模块风格。

更多参考[含有表达式的依赖表述][context]、[CommonJs][CommonJs] 和 [AMD][AMD]。

### [插件系统][plugins]

webpack 有一个很出色的插件系统，甚至大部分内置功能都是基于这个插件系统而来的。这个插件系统允许你根据需要来自定义 webpack，以及通过开源的方式来分发通用插件。

更多参考[插件][plugins]。

[Code Spliting]: code-spliting.md
[Loaders]: loaders.md
[Using loaders]: using-loaders.md
[CommonJs]: commonjs.md
[AMD]: amd.md
[context]: context.md
[plugins]: plugins.md
