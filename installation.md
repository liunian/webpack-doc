# 安装

## node.js

安装 [node.js][node.js]。

其中，node.js 内置了一个叫做 `npm` 的包管理工具。

## webpack

通过 `npm` 安装 webpack：

```bash
$ npm install webpack -g
```

这样，webpack 将被安装为全局工具，同时，也提供了一个 `webpack` 的命令。

## 在项目中使用 webpack

最好的方法是把 webpack 作为项目的依赖，这样，每个项目都可以使用自己所需要的版本而不必依赖于全局的 webpack。

通过 `npm` 来增加一个 `package.json` 文件：

```bash
$ npm init
```

如果不把项目发布到 npm 上，上述命令中要求回答的问题的答案则不是很重要。

使用下面的命令来安装 `webpack` 并把其添加到 `packeage.json` 中：

```bash
$ npm install webpack --save-dev
```

## 版本

webpack 有稳定版和 beta 版两个可用的版本，其中，beta 版本在版本号中添加了 `-beta` 的标识。beta 版本可能带有很多不稳定变更或实验性的功能，并且未完全测试。可查看[变更记录][changelog]来看各版本的差别。正式项目中应该使用稳定版本。

```bash
$ npm install webpack@1.2.x --save-dev
```

## 延伸阅读

更多可阅读[使用][usage]。

[node.js]: http://nodejs.org
[changelog]: changelog.md
[usage]: usage.md
