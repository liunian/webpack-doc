# COMMONJS

CommonJS 工作组通过定义一种模块格式来解决 JavaScript 作用域问题，其原理是让每个模块都运行在其自己的命名空间里。

这一目标的达成是通过强制固定每个模块都必需显式地向外导出欲暴露出去的变量以及定义了引用其它模块的方式。

为了这一目的，CommonJS 提供了两个工具：

1. `require` 函数，这允许把别的模块引入到当前作用域
2. `module` 对象，用于从当前作用域导出数据

下面是一个符合规范的 hello world 例子。

## 简单 JavaScript

下面是一个使用 CommonJS 的例子。

在 `salute.js` 中定义一个值，该脚本仅包含这个用于其它脚本的值。

```js
// salute.js
var MySalute = "Hello";
```

然后，在另一个文件 `world.js` 中使用 `salute.js` 中定义的值。

```js
// world.js
var Result = MySalute + " world!";
```

## 模块依赖

实际上，`world.js` 并不正常工作，因为 `MySalute` 未定义。因此，我们需要把每个脚本都定义为模块。

```js
// salute.js
var MySalute = "Hello";
module.exports = MuSalute;
```

```js
// world.js
var Result = MySalute + " world!";
module.exports = Result;
```

这里我们使用了 `module` 对象并把我们的值赋给了 `module.exports`，这样 CommonJS 模块系统知道我们的模块希望对外提供给这些值。`salute.js` 暴露了 `MySalute`，`world.js` 暴露了 `Result`。

## 示例

### 函数

```js
// moduleA.js
module.exports = function( value ){
    return value*2;
}
```

```js
// moduleB.js
var multiplyBy2 = require('moduleA');
var result = multiplyBy2( 4 );
```
