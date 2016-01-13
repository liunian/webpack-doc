# webpack 入门教程

## 欢迎

这个简单的教程会通个一个简单的例子来作为指南。

从中，你可以学到：

- 怎么安装 webpack
- 怎么使用 webpack
- 怎么使用加载器
- 怎么使用开发服务器

-----

## 安装 webpack

首先确保安装了 [node.js][node.js]

```bash
$ npm install webpack -g
```

> 上面命令确保可以在命令行中使用 webpack 命令

-----

## 设定编译

从一个空白文件开始，创建以下文件：

增加 `entry.js`

```js
document.write("It works.");
```

增加 `index.html`

```html
<html>
    <head>
        <meta charset="utf-8">
    </head>
    <body>
        <script type="text/javascript" src="bundle.js" charset="utf-8"></script>
    </body>
</html>
```

然后执行以下命令：

```bash
$ webpack ./entry.js bundle.js
```

这将编译 entry.js 并输出为 bundle.js。

如果一切正常，将输入类似下面的信息：

```
Hash: e97678c23acf8ee01956
Version: webpack 1.9.10
Time: 135ms
    Asset     Size  Chunks             Chunk Names
bundle.js  1.44 kB       0  [emitted]  main
   [0] ./entry.js 29 bytes {0} [built]
```

在浏览器中打开 `index.html`，将展示 `It works.`。

```html
It works.
```

-----

## 第二个文件

下一步，我们将把部分代码移到另一个文件中。

增加 `content.js`

```js
module.exports = "It works from content.js.";
```

更新 `entry.js`

```diff
- document.write("It works.");
+ document.write(require("./content.js"));
```

然后编译

```bash
$ webpack ./entry.js bundle.js
```

刷新浏览器，可以看到内容变为 `It works from content.js.`。

```html
It works from content.js.
```

> webpack 会分析入口文件（entry.js）来找寻依赖文件。这些文件（又称之为模块）会包括到 `bundle.js` 中。webpack 会给每个模块一个唯一的 id，然后在 `bundle.js` 中通过该 id 来访问对应的模块。启动时，只会执行入口模块。一个短小的运行时提供了 `require` 函数，在引用模块时会执行依赖模块。

-----

## 首个加载器

我们希望在应用中增加一个 css 文件。

因为 webpack 本身只能支持 js，所以需要 `css-loader` 来处理 css 文件，同时还需要一个 `style-loader` 来在页面中使用 css 文件里定义的样式。

创建一个空白的 `node_modules` 目录。

执行 `npm install css-loader style-loader` 来安装加载器。（因为需要安装在当前应用，所以不需要 `-g` 参数）

然后具体使用

增加 `style.css`

```css
body {
	background: yellow;
}
```

更新 `entry.js`

```diff
+ require("!style!css!./style.css");
document.write(require("./content.js"));
```

> 通过在模块请求前添加加载器，模块将会被加载器管道处理。这些加载器特定的方式来处理文件内容，最终转换后的内容将是一个 javascript 模块。

> 注：串联起来的加载器将自右向左执行。

-----

## 绑定加载器

我们并不希望写类似这样的代码 `require("!style!css!./style.css");`。我们可以给指定的文件后缀绑定加载器，这样只需要写 `require("./style.css")`。

更新 `entry.js`

```diff
- require("!style!css!./style.css");
+ require("./style.css");
  document.write(require("./content.js"));
```

然后运行下面的命令来编译

```bash
$ webpack ./entry.js bundle.js --module-bind 'css=style!css'
```

刷新页面，将看到同样的效果。

```html
It works from content.js.
```

-----

## 配置文件

把配置选项移到配置文件中

增加 `webpack.config.js`

```js
module.exports = {
    entry: "./entry.js",
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [
            { test: /\.css$/, loader: "style!css" }
        ]
    }
};
```

然后只需执行 `webpak` 来编译

```bash
Hash: 88d0f4710a9d3a391d69
Version: webpack 1.9.10
Time: 263ms
    Asset     Size  Chunks             Chunk Names
bundle.js  10.7 kB       0  [emitted]  main
   [0] ./entry.js 76 bytes {0} [built]
   [5] ./content.js 44 bytes {0} [built]
    + 4 hidden modules
```

> webpack 命令会尝试加载当前目录下的 `webpack.config.js` 文件。

-----

## 更友好的输出

如果项目变得越来越大，编译耗时将会越来越长。所以，我们希望能够展示一些进度条，同时增加颜色……

这些需求可以通过以下命令来实现

```bash
$ webpack --progress --colors
```

-----

## 监控模式

我们并不希望每一个变更都需要去手动编译，则可以通过以下命令来改善。

```bash
$ webpack --progress --colors --watch
```

这样，webpack 会缓存未变更的模块而输出变更的模块。

> 开启 webpack 监控模式后，webpack 会给所有文件添加用于编译的文件监控。如果有任何变更，将会触发编译。当缓存开启时，webpack 会在内存中保存所有模块内容并在没变更时直接重用。

-----

## 开发服务器

开发服务器是一个更好的选择

```bash
$ npm install webpack-dev-server -g
```

然后

```bash
$ webpack-dev-server --progress --colors
```

这会在 localhost:8080 提供一个小的 express 服务器来提供静态资源以及自动编译的套件。当有套件重新编译后，会通过 socket.io 来自动更新浏览器页面。在浏览器中打开  http://localhost:8080/webpack-dev-server/bundle 。

> webpack-dev-server 使用了 webpack 监控模式，同时也阻止了 webpack 生成结果文件到硬盘，而是直接通过内存来提供服务。


[node.js]: http://nodejs.org/
