# 样式

## 内嵌样式

通过 `style-loader` 和 `css-loader` 可以把样式内嵌到 javascript 套件中，这样可以把 css 模块化，使用起来很简单，直接 `require('./stylesheet.css')` 就可以了。

### 安装

通过 npm 来安装加载器

```bash
npm install style-loader css-loader --save-dev
```

### 配置

支持 `require()` css 的配置文件可参考下面这样：

```js
{
    // ...
    module: {
        loaders: [
            { test: /\.css$/, loader: "style-loader!css-loader" }
        ]
    }
}
```

> 对于 css 预处理语言，其配置文件参照对应加载器的文档即可，可以串联使用

需要注意模块的执行顺序很难完全把握，所以需要设计好样式让其和顺序无关，但同一个样式内的顺序还是可用的。

### 使用 css

```js
// in your modules just require the stylesheet
// This has the side effect that a <style>-tag is added to the DOM.
require("./stylesheet.css");
```

## 分离的 css 套件

结合使用 [extract-text-webpack-plugin][extract-text-webpack-plugin]，可以生成一个原生的 css 文件。

结合代码分离可以有两种方式：

- 每个初始块（参考[代码分离][code splitting]）创建一个 css 文件，然后在别的依赖块中引入样式（推荐）。
- 每个初始块创建一个 css 文件，其中同时包含了依赖块中的样式

之所以推荐第一种方式是因为可以优化初始化加载时间。第二种方式更适合于多入口点的小型 app，因为可以利用缓存来减少 http 消耗。

### 安装插件

```bash
npm install extract-text-webpack-plugin --save
```

### 通用

使用该插件需要把每个需要用特定加载器提取到 css 文件中的模块标记出来。在编译的优化阶段，插件会检查每个待提取的模块（在第一种方式中，只有这些会被提取到初始块中）。这些模块会被编译成 node.js 用法然后执行来获取内容。另外，这些模块在原套件中会被重新编译然后被一个空白的模块替换。

被提取的模块将生成一个新的资源文件。

### 把 css 从初始块中分离到独立的文件

以下是一个多入口的示例，但对但入口同样使用。

```js
// webpack.config.js
var ExtractTextPlugin = require("extract-text-webpack-plugin");
module.exports = {
    // The standard entry point and output config
    entry: {
        posts: "./posts",
        post: "./post",
        about: "./about"
    },
    output: {
        filename: "[name].js",
        chunkFilename: "[id].js"
    },
    module: {
        loaders: [
            // Extract css files
            {
                test: /\.css$/,
                loader: ExtractTextPlugin.extract("style-loader", "css-loader")
            },
            // Optionally extract less files
            // or any other compile-to-css language
            {
                test: /\.less$/,
                loader: ExtractTextPlugin.extract("style-loader", "css-loader!less-loader")
            }
            // You could also use other loaders the same way. I. e. the autoprefixer-loader
        ]
    },
    // Use the plugin to specify the resulting filename (and add needed behavior to the compiler)
    plugins: [
        new ExtractTextPlugin("[name].css")
    ]
}
```

将获得一下输出文件：

- `posts.js`、`posts.css`
- `post.js`、`post.css`
- `about.js`、`abcout.css`
- `1.js`、`2.js`（包含了内嵌样式）

### 把所有的样式分离到文件

把 `allChunks` 选项设置为 `true` 来开启第二种模式

```js
// ...
module.exports = {
    // ...
    plugins: [
        new ExtractTextPlugin("style.css", {
            allChunks: true
        })
    ]
}
```

将获得以下输出文件：

- `posts.js`、`posts.css`
- `post.js`、`post.css`
- `about.js`、`abcout.css`
- `1.js`、`2.js`（不包含内嵌样式）

### 公共块的样式

可以结合 CommonsChunkPlugin 插件来使用独立的 css 文件，在这种情况下，公共块也会生成一个独立的 css 文件。

```js
// ...
module.exports = {
    // ...
    plugins: [
        new webpack.optimize.CommonsChunkPlugin("commons", "commons.js"),
        new ExtractTextPlugin("[name].css")
    ]
}
```

将获得以下输出文件：

- `common.js`、`common.css`
- `posts.js`、`posts.css`
- `post.js`、`post.css`
- `about.js`、`abcout.css`
- `1.js`、`2.js`（包含了内嵌样式）

或开启了 `allChunks: true` 后

- `1.js`、`2.js`（不包含内嵌样式）

[extract-text-webpack-plugin]: https://github.com/webpack/extract-text-webpack-plugin
[code splitting]: code-splitting.md

