# 使用加载器

## 什么是加载器

加载器是把一个资源文件作为入参转换为另一个资源文件的 node.js 函数。

例如，可以通过加载器来让 webpack 来加载 CoffeeScript 或 JSX 文件。

### 加载器特性

- 加载器可以串联。可以通过管道来处理资源，最后一个加载器需要输出为 JavaScript，中间别的加载器可以输出任意的格式到下一个加载器。
- 加载器可以是同步的也可以是异步的
- 加载器可以通过 node.js 来处理所有的事情
- 加载器可以传查询参数，这可以通过配置文件来配置
- 加载器可以通过在配置文件中的扩展或正则来绑定
- 加载器可以通过 `npm` 来发布及安装
- 模块除了在 `package.json` 中的 `main` 字段外，也可以通过 `loader` 字段来导出加载器
- 可以通过配置文件来访问加载器
- 插件可以提供加载器更多特性
- 加载器可以产出任何格式的文件
- [更多……][loaders]

如果对加载器使用示例有兴趣，可以查看[加载器列表][list-of-loaders]。

## 解析加载器

[解析加载器][resolving]和解析模块类似。一个加载器模块即一个导出函数的 node.js 模块。一般通过 npm 来发布加载器，但也可以直接把加载器存在项目中。

### 引用加载器

虽然不是强制要求，但惯例把加载器命名为 `XXX-loader`，其中 `XXX` 就是加载器的名字，如：`json-loader`。

可以通过全称（实际的名称）来引用加载器（如 `json-loader`），也可以通过简略名来引用（如 `json`）。

加载器的命名惯例和搜索优先级是通过 webpack 的配置 api `resolveLoader.moduleTemplate` 来指定的。

参考下面的用法，可以看到加载器命名惯例很有用，尤其是通过 `require()` 语句来引用加载器时。

### 安装加载器

如果加载器发布在 npm 上，那么可以通过下面的两个命令来安装

`$ npm install xxx-loader --save`

`$ npm install xxx-loader --save-dev`

## 用法

在项目中，有多种方式来使用加载器。

- 通过 `require` 语句显示指定
- 通过配置文件来指定
- 通过命令行来指定

### 通过 `require`

> 注：尽量不要使用这种方式，特别是希望脚本可以环境无关地同时跑在 node.js 和浏览器环境中。优先通过配置文件中的命名惯例来指定加载器，参考下一节。

可以通过 `require` 语句（或 `define`、`require.ensure` 等）来指定加载器，只需要用 `!` 来分隔资源即可，每部分都是相对当前目录。

```js
require("./loader!./dir/file.txt");
// => 使用当前目录下的 "loader.js" 来转换 "dir" 目录下的 "file.txt" 文件

require("jade!./template.jade");
// => 使用 "jade-loader" （通过 npm 安装在 "node_modules" 目录）来转换 "template.jade" 文件

require("style!css!less!bootstrap/less/bootstrap.less");
// => 通过 github 来安装在 "node_modules" 目录下的 "bootstrap" 模块里的 "less" 目录里的 "bootstrap.less"
//    文件会先被 "less-loader" 转换，然后再被 "css-loader" 转换，最后被 "style-loader" 转换
```

### 通过配置文件

可以通过正则来在配置文件中绑定加载器

```js
{
    module: {
        loaders: [
            { test: /\.jade$/, loader: "jade" },
            // => "jade" loader is used for ".jade" files

            { test: /\.css$/, loader: "style!css" },
            // => "style" and "css" loader is used for ".css" files
            // Alternative syntax:
            { test: /\.css$/, loaders: ["style", "css"] },
        ]
    }
}
```

### 通过命令行

通过 CLI 扩展参数来绑定加载器

```bash
$ webpack --module-bind jade --module-bind 'css=style!css'
```

上述命令给 `.jade` 文件绑定了 `jade` 加载器，给 `.css` 文件班定了 `style` 和 `css` 加载器。

### 查询参数

加载器可以通过查询 web 风格的字符串来指定查询参数，如：`url-loader?mimetype=image/png`。

> 注：查询字符串的格式要看加载器，具体格式参考加载器文档。大多数的加载器支持普通的查询格式（`?key=value&key2=value2`）和 JSON 对象（`?{"key":"value","key2":"value2"}`）。

#### `require`

```js
require("url-loader?mimetype=image/png!./file.png");
```

#### 配置文件

```js
{ test: /\.png$/, loader: "url-loader?mimetype=image/png" }
```

或

```js
{
    test: /\.png$/,
    loader: "url-loader"
    query: { mimetype: "image/png" }
}
```

#### 命令行

```bash
webpack --module-bind "png=url-loader?mimetype=image/png"
```

[loaders]: loader.md
[list-of-loaders]: list-of-orders.md
[resolving]: resolving.md
[configuration]: configuration.md
[cli]: cli.md
