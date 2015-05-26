# webpack

## 插件

webpack 有着丰富的[插件][plugin]接口，这让 webpack 更具有扩展性，内建插件也大多基于这些接口。

## 性能

webpack 使用异步 I/O 和多层缓存，从而运行得很快，尤其是增量构建。


## 加载器（Loaders）

webpack 允许使用[加载器][loaders]来预处理文件，从而可以打包任何文件而不仅是 JavaScript。可以用 node.js 来轻易地自定义加载器。

## 支持

webpack 支持 [AMD][AMD] 和 [CommonJs][CommonJs] 这两种模块机制。可以通过 AST 来对代码来做静态分析。还可以通过一个运算引擎来做简单表达式运算，从而支持绝大多数的现存 JavaScript 库。

## 代码分离

webpack 支持把代码库[拆分][split]成多块，从而可以按需加载，减少初次加载的耗时。

## 优化

webpack 通过多种[优化][optimization]来减少输出体积，同时还支持使用 hashes 来做[请求缓存][request-caching]。

## 开发工具

webpack 支持 SourceUrls 和 SourceMaps，从而让 debug 更友好。通过 watch 和 [开发中间件][webpack-dev-middleware]以及[开发服务器][webpack-dev-server]来实现自动更新。

## 多目标

webpack 的主要目标是 web，但同时也支持 [WebWorker][worker-loader] 和 node.js 的打包。


[plugin]: ./plugin.md
[loaders]: ./loaders.md
[AMD]: ./amd.md
[CommonJs]: ./commonjs.md
[split]: ./code-splitting.md
[optimization]: ./optimization.md
[request-caching]: ./long-term-caching.md
[webpack-dev-middleware]: ./webpack-dev-middleware.md
[webpack-dev-server]: ./webpack-dev-server.md
[worker-loader]: ./worker-loader.md
