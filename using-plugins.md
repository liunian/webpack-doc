# 使用插件

## 内置插件

通过在 webpack 配置文件中添加 `plugins` 字段来给模块引入插件。

```js
// webpack should be in the node_modules directory, install if not.
var webpack = require("webpack");

module.exports = {
    plugins: [
        new webpack.ResolverPlugin([
            new webpack.ResolverPlugin.DirectoryDescriptionFilePlugin("bower.json", ["main"])
        ], ["normal", "loader"])
    ]
};
```

## 其它插件

其它非内置插件可以通过 `npm` 或别的可用方式来安装。

```bash
npm install component-webpack-plugin
```

然后可以这样使用：

```js
var ComponentPlugin = require("component-webpack-plugin");
module.exports = {
    plugins: [
        new ComponentPlugin();
    ]
}
```

## 更多

[插件列表][list-of-plugins]

[list-of-plugins]: ./list-of-plugins.md
