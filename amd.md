# AMD

AMD（异步模块定义）是对 天生同步的 CommonJS 模块系统不适合浏览器环境的回应。

AMD 定义了模块可以异步加载其依赖的模块标准，解决了同步加载的问题。

## 规范

模块通过 `define` 函数来定义。

### `define`

下面的 `define` 函数就是 AMD 中模块的定义形式，它只是一个有参数的函数。

```js
define(id?: String, dependencies?: String[], factory: Function|Object);
```

#### `id`

模块标识，可选。

#### `dependencies`

当前模块所依赖的模块，具体是一个元素是模块标识的数组。可选，不填时，默认是 `["require", "exports", "module"]`。

#### `factory`

最后一个参数是模块实体，可以是一个函数（只调用一次），也可以是一个对象。当 `factory` 是函数时，该函数返回的值讲师模块对外提供的值。

## 示例

来看看几个例子：

### 命名模块

定义一个依赖于 `jQuery` 的名为 `myModule` 的模块。

```js
define('myModule', ['jquery'], function($) {
    // $ is the export of the jquery module.
    $('body').text('hello world');
});
// and use it
define(['myModule'], function(myModule) {});
```

> 注：在 webpack 中，一个命名模块仅局部可用。在 Require.js 中，则是全局可用。

### 匿名模块

定义一个不指定标识的模块

```js
define(['jquery'], function($) {
    $('body').text('hello world');
});
```

### 多个依赖

定义一个依赖于多个模块的模块。注意，每个依赖模块的导出值都会被传入到工厂函数中。


```js
define(['jquery', './math.js'], function($, math) {
    // $ and math are the exports of the jquery module.
    $('body').text('hello world');
});
```

### 导出值

定义一个导出其本身的模块

```js
define(['jquery'], function($) {

    var HelloWorldize = function(selector){
        $(selector).text('hello world');
    };

    return HelloWorldize;
});
```

### 使用 require 加载依赖

```js
define(function(require) {
    var $ = require('jquery');
    $('body').text('hello world');
});
```