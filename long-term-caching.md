# 持久缓存

为了有效地缓存文件，通常需要在URL上加一个hash值或版本号。你可以输出或手动移动文件到在一个名为V1.3文件夹。但这有几个缺点:开发人员中额外代码和未改变文件不会从缓存加载。

Webpack中的Loaders(`worker-loader`, `file-loader`)可以为文件的文件名添加hash值。你可以通过设置下面两种选项来为分块添加hash。

1. 计算并添加所有分块的hash值
2. 计算并添加每一分块的hash值

## 选项一：整体文件配置一个hash值

选项一通过添加`[hash]`到文件名称配置选项中

```bash
webpack ./entry output.[hash].bundle.js
```

```js
{
    output: {
        path: path.join(__dirname, "assets", "[hash]"),
        publicPath: "assets/[hash]/",
        filename: "output.[hash].bundle.js",
        chunkFilename: "[id].[hash].bundle.js"
    }
}
```

## 选项二：每一个分块添加hash值

选项二通过添加`[chunkhash]`到分块文件名称配置选项中

```bash
--output-chunk-file [chunkhash].js
```

```js
output: { chunkFilename: "[chunkhash].bundle.js" }
```

注意: 你需要把入口块的hash值添加到你的HTML中。这时，你可能更想要提取hash或文件名。

如果使用webpack模块热替换，则必须使用选项一的配置。

## 从stats中获取文件名

你可能想访问资源的最后文件名插入到HTML，在webpack的stats是可以拿到这些信息的。如果你正在使用CLI可以运行 `--json`，会以JSON格式数据打印到stdout。

你可以添加一些能拿到stats对象信息的插件到配置项中，例如 `assets-webpack-plugin` 插件。
下面示例展示如何把它们写入到文件中。

```js
plugins: [
  function() {
    this.plugin("done", function(stats) {
      require("fs").writeFileSync(
        path.join(__dirname, "..", "stats.json"),
        JSON.stringify(stats.toJson()));
    });
  }
]
```

stats.json数据包含一个有用的对象属性 `assetsByChunkName`, 其中分块名称作为键和文件名(s)作为值。

> 注意: 如果你的每个分块输出多个资源，例如一个JavaScript文件和一个SourceMap，它将是一个数组。第一个是你的JavaScript内容。
