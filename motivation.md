# 动机

现在的网站已经演化成 web app：

- 一个页面内依赖着越来越多的 JavaScript
- 现代浏览器有着越来越多的
- 虽然单页内代码越来越多，但全页刷新越来越少
	
这意味着，客户端有着越来越多的代码。

一个大的代码库需要良好的组织，这需要模块系统来把代码库拆分成模块。

## 模块系统风格

有着以下几种不同的定义依赖和导出值的方案：

- `<script>` 标签方式（无模块系统）
- CommonJs
- AMD 及其变种
- ES6 模块
- 其它……

### `<script>` 标签方式

这是在不使用模块系统的前提下来管理模块代码库的方式。

```html
<script src="module1.js"></script>
<script src="module2.js"></script>
<script src="libaryA.js"></script>
<script src="module3.js"></script>
```

模块导出接口到全局对象中，如 `window` 对象，也通过全局对象来访问依赖模块的接口。

#### 普遍的问题

- 全局变量冲突
- 强依赖加载顺序
- 开发者需要结局模块和库的依赖关系
- 大项目中列表越来越长而难以维护

### CommonJs：同步 require

这种方式使用同步的 `require` 来加载依赖并且通过给 `exports` 添加属性或给 `module.exports` 赋值来导出接口。

```js
require("module");
require("../file.js");
exports.doStuff = function() {};
module.exports = someValue;
```

这种方式用于 [node.js][node.js]。

#### 优点

- 重用服务端模块
- npm 上已经有很多这种模块
- 简单且易于使用

#### 缺点

- 网络请求不适用于同步请求，更适合异步请求
- 不能并行请求多个模块

#### 实现

- 服务端 [node.js][node.js]
- [browserify][browserify]
- [modules-webmake][modules-webmake]，打包为一个文件
- 客户端 [wreq][wreq]


### AMD：异步 require

[异步模块定义][AMD]

因为 CommonJs 的同步 `require` 方案不适合如浏览器等环境，所以有了异步的定义和导出模块方案。

```js
require(["module", "../file"], function(module, file) { /* ... */ });
define("mymodule", ["dep1", "dep2"], function(d1, d2) {
  return someExportedValue;
});
```

#### 优点

- 适合异步网络请求方式
- 并行加载多模块

#### 缺点

- 更多的编码，更难阅读和编写
- 有不同的方案

#### 实现

- [require.js][require.js] 客户端
- [curl][curl] 客户端

更多可参考 [CommonJs][CommonJs] 和 [AMD][AMD]。


### ES6 模块

EcmaScript6 给 JavaScript 增加了模块系统。

```js
import "jquery";
export function doStuff() {}
module "localModule" {}
```

#### 优点

- 易于静态分析
- 未来的标准

#### 缺点

- 浏览器原生支持需要时间
- 这种风格的模块还很少

### 统一解决方案

自主选择模块风格，兼容已有代码，易于增加新的模块风格。

-----

## 模块加载

因为模块是在客户端执行的，所以首先需要把模块从服务端加载到浏览器。

关于模块加载，有两个极端的方案：

- 一个模块一个请求
- 所有模块一个请求

这两种方案都被广泛使用，但都不是最佳的。

- 一个模块一个请求

	- 优点：按需加载
	- 缺点：过多请求导致负载过多，延迟过大，初始化慢

- 所有模块一个请求

	- 优点：更少的负载，更少的延迟
	- 缺点：不需要或还不需要的模块也被加载了

### 分块加载

极端方案的相互妥协在大多数情况下会更好，一个更具有弹性的加载方案会更好。

→ 编译模块时，把模块拆成更小的块

这样，将得到多个更小的请求。模块的每个块将是按需加载，初始请求中不包含完整的代码库，从而可以更小。

「拆分点」是可选的，并且可以由开发者自定义。

→ 可以组成一个大的代码库

注：此思路来自 [GWT][GWT]。

更多可参考[代码拆分][Code Spliting]。

-----

## 为何仅 JavaScript？

为什么仅给开发者提供 JavaScript 的模块系统？实际上还有很多别的静态资源需要处理，如：

- 样式
- 图片
- web 字体
- html 模板
- 等等

另外还有一些预编译资源：

- coffeescript -> javascript
- less -> css
- jade -> 生成模板的 javascript
- i18n 文件 -> 别的资源
- 等等

这些应该同样的使用才对，如：

```js
require("./style.css");
```

```js
require("./style.less");
require("./template.jade");
require("./image.png");
```

更多可参考[使用加载器][Using loaders]和[加载器][loaders]。

-----

## 静态分析

在编译模块时，会尝试用静态分析来获取依赖关系。

一般来说，这只能找到没有表达式的简单形式，但像 `require("./template/" + templateName + ".jade")` 确实一种常见的用法。

不同的代码库往往使用不同的风格，其中不少的风格还是比较怪异的……

### 策略

一个好的解析器应当支持绝大多数的现存代码。如果开发者用了一些不寻常的方法，那么解析器应该提供一个兼容性最好的解决方案。


[node.js]: http://nodejs.org
[browserify]: https://github.com/substack/node-browserify
[modules-webmake]: https://github.com/medikoo/modules-webmake
[wreq]: https://github.com/substack/wreq
[AMD]: https://github.com/amdjs/amdjs-api/wiki/AMD
[require.js]: http://requirejs.org/
[curl]: https://github.com/cujojs/curl
[CommonJs]: http://webpack.github.io/docs/commonjs.html
[GWT]: https://developers.google.com/web-toolkit/doc/latest/DevGuideCodeSplitting
[Code Spliting]: ./code-splitting.html
[Loaders]: ./loaders.html
[Using loaders]: ./using-loaders.html