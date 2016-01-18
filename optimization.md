# 优化

## 压缩

可以通过简单的配置来让 webpack 来压缩脚本（通过 css-loader 来支持压缩 css）。

`--optimize-minimize` 或 `new webpack.optimize.UglifyJsPlugin()`

这是一个简单但有效的优化应用的方法。

如果阅读了剩下的文档，那么将了解到，webpack 给每个模块和分块分配了 id 来识别它们。webpack 可以通过下面的简单配置给最常用的 id 分配最简短的 id 来进行优化：

`--optimize-occurence-order` 或 `new webpack.optimize.OccurenceOrderPlugin()`

入口块有更高的文件大小优先级。

## 去重

在使用各有依赖的代码库时，有可能这些库之前有着完全相同的依赖。Webpack 能够找到这些相同的文件并进行去重，这样会在生成的文件中包含一次而在运行时多次使用。这个功能并不会影响代码语义，可以通过下面的配置来开启：

`--optimize-dedupe` 或 `new webpack.optimize.DedupePlugin()`

这个功能会在入口块中添加额外的代码。

## 分块

编写代码的过程中，可能会使用代码分离点来做按需加载。但在编译后却发现过多的小分块反而加重了 HTTP 请求的负担。幸运的是，Webpack 可以通过合并这些小的分块来解决这种问题，相关的配置有两个：

- 限制最大分块数量 `--optimize-max-chunks 15` 或 `new webpack.optimize.LimitChunkCountPlugin({maxChunks: 15})`
- 限制最小的分块大小 `--optimize-min-chunk-size 10000` 或 `new webpack.optimize.MinChunkSizePlugin({minChunkSize: 10000})`

Webpack 会小心地处理分块合并（优先合并有重复模块的分块），不会把这些小的分块合并进入口块中，因为这样有可能会影响初始加载时间。

## 单页应用

Webpack 本身就是为单页应用设计和优化的。

你可能已经把应用代码分离成了根据路由来加载的分块，其中入口块值包含了路由和依赖库而没有实质内容。这个做法对于应用中的浏览切换是挺好的，只是初始页面有两个网络请求：路由和当前页的处理代码。

如果你使用了 HTML5 的 history api 来做页面内容和 url 的一一对应，那么服务端就能根据 url 来知道请求的具体单页应用中的哪一个页面。这样，可以通过服务端直接同时输出请求页面的处理分块，利用浏览器的并行加载来节省网络请求时间。

```html
<script src="entry-chunk.js" type="text/javascript" charset="utf-8"></script>
<script src="3.chunk.js" type="text/javascript" charset="utf-8"></script>
```

分块的名称可以从 webpack 的 stats 中读取（可以使用 [stats-webpack-plugin][stats-webpack-plugin] 来导出构建的 status）。

## 多页应用

当在搭建多页应用时，很自然的是希望能够跨页共享代码。Webpack 可以很简单地实现这个需求，只需要使用多个入口点就可以了：

`webpack p1=./page1 p2=./page2 p3=./page3 [name].entry.chunk.js`

```js
module.exports = {
    entry: {
        p1: "./page1",
        p2: "./page2",
        p3: "./page3"
    },
    output: {
        filename: "[name].entry.chunk.js"
    }
}
```

这样就会创建多个入口块：`p1.entry.chunk.js`，`p2.entry.chunk.js` 和 `p3.entry.chunk.js`，但它们可以共享别的分块。

如果你的入口分块之前有使用相同的模块，那么有个插件会很有用。`CommonsChunkPlugin` 可以找出相同的模块然后把他们提取出来放到一个公共分块中。这样，页面中只需要引入两个 script，一个是公共分块另一个则是该页面的入口块。

```js
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
    entry: {
        p1: "./page1",
        p2: "./page2",
        p3: "./page3"
    },
    output: {
        filename: "[name].entry.chunk.js"
    },
    plugins: [
        new CommonsChunkPlugin("commons.chunk.js")
    ]
}
```

这样会生成 `p1.entry.chunk.js`、`p2.entry.chunk.js` 和 `p3.entry.chunk.js` 以及 `commons.chunk.js`。首先加载 `commons.chunk.js`，然后加载对应的入口块。

还可以通过选择需要的入口分块来生成多个公共块，公共块之间也能嵌套引用。

```js
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
    entry: {
        p1: "./page1",
        p2: "./page2",
        p3: "./page3",
        ap1: "./admin/page1",
        ap2: "./admin/page2"
    },
    output: {
        filename: "[name].js"
    },
    plugins: [
        new CommonsChunkPlugin("admin-commons.js", ["ap1", "ap2"]),
        new CommonsChunkPlugin("commons.js", ["p1", "p2", "admin-commons.js"])
    ]
};
// <script>s required:
// page1.html: commons.js, p1.js
// page2.html: commons.js, p2.js
// page3.html: p3.js
// admin-page1.html: commons.js, admin-commons.js, ap1.js
// admin-page2.html: commons.js, admin-commons.js, ap2.js
```

偷偷告诉你，公共块里的代码也能直接执行：

```js
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
    entry: {
        p1: "./page1",
        p2: "./page2",
        commons: "./entry-for-the-commons-chunk"
    },
    plugins: [
        new CommonsChunkPlugin("commons", "commons.js")
    ]
};
```

更多可参考 [多入口示例][multiple-entry-points] 和 [高级多公共分块示例][multiple-commons-chunks]

[stats-webpack-plugin]: https://www.npmjs.com/package/stats-webpack-plugin
[multiple-entry-points]: https://github.com/webpack/webpack/tree/master/examples/multiple-entry-points
[multiple-commons-chunks]: https://github.com/webpack/webpack/tree/master/examples/multiple-commons-chunks
