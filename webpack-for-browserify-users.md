# 从 browserify 切换到 webpack

## 用法

和 browserify 类似，webpack 分析应用中 node 风格的 `require()` 调用来提取依赖，然后打包层一个套件，这样可以在浏览器中仅使用一次 `<script>` 标签来引入脚本。

这是 browserify 的用法

```bash
$ browserify main.js > bundle.js
```

这是 webpack 的用法

```bash
$ webpack main.js bundle.js
```

> 因为 webpack 不使用标准输出，所以必须指定输出文件名。之所以不使用标准输出，是因为它和 browserify 不同，可能会生成多个文件。

-----

[配置][configuration]webpack的最好方式是使用 `webpack.config.js` 文件。当使用[命令行][cli]时，将会从当前目录加载该配置文件。

所以，以下 browserify 命令将对应下面的 `webpack` 配置。

```bash
$ browserify --entry main.js --outfile bundle.js
```

```js
module.exports = {
    entry: "./main.js",
    output: {
        filename: "bundle.js"
    }
}
```

> 配置文件 `webpack.config.js` 需要导出配置，如上面例子中的 `module.exports = {...}`。

-----

### 输出文件

如果需要把文件生成到另一个目录，在 browserify 是：

```bash
$ browserify --outfile js/bundle.js
```

而webpack 配置是：

```js
{
    output: {
        path: path.join(__dirname, "js"),
        filename: "bundle.js"
    }
}
```

### 入口

```bash
$ browserify --entry a.js --entry b.js
```

```js
{
    entry: [
        "./a.js",
        "./b.js"
    ]
}
```

### 转换

browserify 使用转换器来处理文件，webpack 使用加载器。加载器是把源码当做参数而返回修改后的代码的函数。和转换器类似，加载器基于 node.js，可以被串联，可以是异步的。加载器可以通过查询字符串来添加额外的参数。加载器可以再 `require()` 调用中使用。转换器可以在 `package.json` 中指定，然后 `browserify` 对每一个模块都做转换。在 webpack 的配置文件中，可以通过正则来选择模块。一般地，在 webpack 配置文件（`webpack.config.js`）中指定加载器。

```bash
$ browserify --transform coffeeify
```

```js
{
    module: {
        loaders: [
            { test: /\.coffee$/, loader: "coffee-loader" }
        ]
    }
}
```

> 可以通过配合使用 webpack 和[转换加载器][transform-loader]来使用 browserify 的转换器。

### debug

```bash
$ browserify -d
# Add inlined sourcemap
```

```bash
$ webpack --devtool inline-source-map
# Add inlined sourcemap

$ webpack --devtool source-map
# 生成独立的 sourcemap 文件

$ webpack --devtool eval
# 通过 eval 生成 sourceUrls（更快）

$ webpack --devtool eval-source-map
# 通过 eval 生成 inline sourcemap

$ webpack --debug
# 给生成的代码增加更多的调试信息

$ webpack --output-pathinfo
# 在生成的源码注释中增加路径

$ webpack -d
# = webpack --devtool source-map --debug --output-pathinfo
```

### 扩展

```bash
$ browserify --extension coffee
```

```js
{
    resolve: {
        extensions: ["", ".js", ".coffee"]
    }
}
```

### 独立完备的代码

```bash
$ browserify --standalone MyLibrary
```

```js
{
    output: {
        library: "MyLibrary",
        libraryTarget: "umd"
    }
}
// webpack --output-library MyLibrary --output-library-target umd
```

### 忽略

```bash
$ browserify --ignore file.js
```

```js
{
    plugins: [
        new webpack.IgnorePlugin(/file\.js$/)
    ]
}
```

### node 全局变量

```bash
$ browserify --insert-globals
$ browserify --detect-globals
```

下列

```js
{
    node: {
        filename: true,
        dirname: "mock",
        process: false,
        global: true
    }
}
```

### 无视模式丢失警告

```bash
$ browserify --ignore-missing
```

webpack 会打印每个缺失依赖的错误信息，但不会终止生成套件，完全可以无视这些错误。但 `require` 会在运行时抛出错误。

### 跳过解析

```bash
$ browserify --noparse=file.js
```

```js
module.exports = {
    module: {
        noParse: [
            /file\.js$/
        ]
    }
}
```

### 构建信息

```bash
$ browserify --deps
$ browserify --list
```

```bash
$ webpack --json
```

### 引用外部资源

webpack 不支持在外部资源中使用 `require`，要么全部资源使用 webpack，要么像下面这样做成库的方式向外提供 `require` 函数。

```js
{
    output: {
        library: "require",
        libraryTarget: "this"
    }
}
```

```js
// entry point
module.exports = function(parentRequire) {
    return function(module) {
        switch(module) {
        case "through": return require("through");
        case "duplexer": return require("duplexer");
        }
        return parentRequire(module);
    };
}(typeof __non_webpack_require__ === "function" ? __non_webpack_require__ : function() {
    throw new Error("Module '" + module + "' not found")
});
```

> 译注：入口文件这样定义 export 后，构建后生成的文件后会在全局暴露 `require` 函数，但上面只能 `require` 有限的模块（如上面的的 `through` 和 `duplexer`）。

### 多套件

可以使用 browserify 来创建共用的套件来供不同页面组合使用，使用方式是使用 `--exclude` 或 `-x` 参数。下面是 browserify 提供的例子：

```bash
$ browserify -r ./robot > static/common.js
$ browserify -x ./robot.js beep.js > static/beep.js
$ browserify -x ./robot.js boop.js > static/boop.js
```

webpack 也支持多页面编制，并且有一个插件来自动提取公共模块。

```js
{
    entry: {
        beep: "./beep.js",
        boop: "./boop.js",
    },
    output: {
        path: "static",
        filename: "[name].js"
    },
    plugins: [
        // ./robot is automatically detected as common module and extracted
        new webpack.optimize.CommonsChunkPlugin("common.js")
    ]
}
```

```html
<script src="common.js"></script>
<script src="beep.js"></script>
```

## API

没有太多的负担，直接把配置对象传入 `webpack` api 就可以了。

```js
var webpack = require("webpack");

webpack({
    entry: "./main.js",
    output: {
        filename: "bundle.js"
    }
}, function(err, stats) {
    err // => fatal compiler error (rar)
    var json = stats.toJson() // => webpack --json
    json.errors // => array of errors
    json.warnings // => array of warnings
});
```

## 第三方工具对应表

| browserify                | webpack                                                       |
| ---                       | ---                                                           |
| watchify                  | `webpack --watch`                                             |
| browserify-middleware     | [webpack-dev-middleware][webpack-dev-middleware]              |
| beefy                     | [webpack-dev-server][webpack-dev-server]                      |
| deAMDify                  | `webpack`                                                     |
| decomponentify            | [component-webpack-plugin][component-webpack-plugin]          |
| list of source transforms | [加载器列表][list-of-loaders], [转换加载器][transform-loader] |



[configuration]: configuration.md
[cli]: cli.md
[transform-loader]: transform-loader.md
[webpack-dev-middleware]: webpack-dev-middleware.md
[webpack-dev-server]: webpack-dev-server.md
[component-webpack-plugin]: component-webpack-plugin.md
[list-of-loaders]: list-of-loaders.md


