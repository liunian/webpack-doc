# 代码分离

在大型的 web 应用中，把所有的代码合并进同一个文件的做法会导致效率低下，特别是有些代码块仅在某些场景才用到。webpack 有个特性能能把代码库拆能可按需加载的块，这些块再其它打包器中会被命名为「层」、「卷」或「片段」。这个特性就是「代码分离」。

这是一个可选功能。可以在代码库中定义分离点，然后 webpack 将会维护好依赖关系、输出文件和运行时文件。

这里需要澄清一个常见的误解，代码分离并不只是提取公共代码出来，更重要的是把代码拆分成可按需加载的块。这样可以让首次加载的代码最少，其余的则按需加载。

## 定义分离点

AMD 和 CommonJs 需要不同的方式来实现按需加载，webpack 两者都支持，并且都是定义为分离点的形式。

### `require.ensure`（CommonJs）

```js
require.ensure(dependencies, callback)
```

`require.ensure` 方法确保 `dependencies` 里的每一个依赖在调用 `callback` 时都被同步地请求进来，其中 `callback` 被调用时将携带 `require` 这个函数作为实参。

示例：

```js
require.ensure(["module-a", "module-b"], function(require) {
	var a = require("module-a");
	// ...
});
```

> 注：`require.ensure` 仅加载模块，但不执行。

### `require`（AMD）

AMD 规范使用下面的方式定义了一个异步的 `require` 方法：

```js
require(dependencies, callback)
```

调用时会加载所有的依赖模块，并把它们作为 `callback` 的实参来传递进去，如：

```js
require(['module-a', 'module-b'], function(a, b) {
	// ...
});
```

> 注：AMD 的 `require` 加载并执行相应的模块，在 webpack 中，模块将自左而右地执行。
> callback 可以为空。

## 分块内容

分离点处的所有模块都会被独立成一个块，依赖会被递归添加。

如果分离点中的 callback 是一个函数，webpack 会把该函数里的依赖都提取到按需加载的代码块中。

## 分块优化

如果两个分块包含的模块完全一致，那么这两个模块会被合并成一个，这意味着分块可能被多个父模块引用。

如果一个模块在一个分块的所有父模块中可用，那么它将会从该分块中移除。

如果一个分块包含了另一个分块的所有模块，它将会被保存起来，可以满足多个分块。

## 分块加载

根据配置文件中的 `target` 选项的不同，将在生成的套件中添加不同的运行时分块加载逻辑，例如：`web` 将使用 jsonp 来加载。一个分块只加载一次，并行的请求将合并成一个。运行时将检查已加载的分块是否满足多个分块。

## 分块类型

### 入口块

一个入口块包含了运行时和一串模块。如果一个入口块包含了模块 `0`，那么运行时将会执行这个模块。如果没包含，那么将会等待包含了模块 `0` 的分块，然后执行（总会有一个分块包含了模块 `0`）。

### 普通块

一个普通的分块仅包含一系列的模块，而不包含运行时。分块的结构和具体分块的加载算法有关，如，jsonp 模块使用 jsonp 回调封装。分块同时还包含了一个其满足的分块的 id 的列表。

### 初始块（非空）

一个初始块是一个普通的分块，不同的是会做特别优化，因为这关系到首次加载时间的长短。这种分块可以通过结合 `CommonChunksPlugin` 来生成。

## 分离应用和第三方库代码

把应用拆分成两个文件，如 `app.js` 和 `vendor.js`，然后再 `vendor.js` 中引用第三方代码。然后像下面这样把名称当做参数传递给 `CommonChunksPlugin`。

```js
module.exports = {
	entry: {
		app: './app.js',
		vendor: ['jquery', 'underscore', ...],
	},
	output: {
		filename: 'bundle.js'
	},
	plugins: [
		new webpack.optimize.CommonsChunkPlugin(/* chunkName = */"vendor", /* filename= */"vendor.bundle.js")
	]
};
```

这将会把 `vendor` 分块中的所有模块从 `app` 分块中移除，然后 `bundle.js` 将只包含应用代码，而把依赖的第三方代码置于 `vendor.bundle.js`。

在 html 页面中，在加载 `bundle.js` 前先加载 `vendor.bundle.js`。

```html
<script src="vendor.bundle.js"></script>
<script src="bundle.js"></script>
```

## 多入口块

可以[配置][configure]多个入口来生成多个入口块。入口块包含了运行时，而页面必需有一个运行时。

### 执行多入口块

使用了 `CommonsChunkPlugin` 后，运行时会被移到公共分块中，这样入口块将在初始块中。虽然只能加载一个入口块，但可以加载多个初始块。从而允许在同一个页面允许多个入口点。

示例：

```js
{
    entry: { a: "./a", b: "./b" },
    output: { filename: "[name].js" },
    plugins: [ new webpack.optimize.CommonsChunkPlugin("init.js") ]
}
```

```html
<script src="init.js"></script>
<script src="a.js"></script>
<script src="b.js"></script>
```

## 公共块

`CommonsChunkPlugin` 会把再多个入口块中的公共模块移到一个新的入口块（公共块）中，同时，也会把运行时移到公共块中。这意味着，原入口块变成了初始块，详见[插件列表][list-of-plugins]。

## 优化

根据具体的场景可以使用不同的优化插件来合并分块，详见[插件列表][list-of-plugins]。

- `LimitChunkCountPlugin`
- `MinChunkSizePlugin`
- `AggressiveMergingPlugin`

## 命名块

`require.ensure` 函数接受一个额外的第三参数，这个参数必需是一个字符串。如果两个分离点使用了同样的字符串，那标识使用了同一个分块。

### `require.include`

```js
require.include(request)
```

`require.include` 是一个 webpack 的函数，用来把一个模块添加到当前分块中，但不执行它（这一步会在打包后移除）。

示例：

```js
require.ensure(["./file"], function(require) {
  require("./file2");
});

// is equals to

require.ensure([], function(require) {
  require.include("./file");
  require("./file2");
});
```

当一个模块再多个子分块中都存在，那么 `require.include` 将很有用。

## 示例

- [Simple][simple]
- [with bundle-loader][with bundle-loader]
- [with context][with context]
- [with amd and context][with amd and context]
- [with deduplication][with deduplication]
- [named-chunks][named-chunks]
- [multiple entry chunks][multiple entry chunks]
- [multiple commons chunks][multiple commons chunks]

可以查看一个[样板应用][example-app]来查看可运行的 demo，注意 DevTools 中的网络请求。


[configure]: cofiguration.md
[list-of-plugins]: list-of-plugins.md
[simple]: https://github.com/webpack/webpack/tree/master/examples/code-splitting
[with bundle-loader]: https://github.com/webpack/webpack/tree/master/examples/code-splitting-bundle-loader
[with context]: https://github.com/webpack/webpack/tree/master/examples/code-splitted-require.context
[with amd and context]: https://github.com/webpack/webpack/tree/master/examples/code-splitted-require.context-amd
[with deduplication]: https://github.com/webpack/webpack/tree/master/examples/code-splitted-dedupe
[named-chunks]: https://github.com/webpack/webpack/tree/master/examples/named-chunks
[multiple entry chunks]: https://github.com/webpack/webpack/tree/master/examples/multiple-entry-points
[multiple commons chunks]: https://github.com/webpack/webpack/tree/master/examples/multiple-commons-chunks
[example-app]: http://webpack.github.io/example-app/


