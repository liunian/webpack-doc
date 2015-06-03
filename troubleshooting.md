# 疑难解答

## 解析

### 常见解析问题

可以通过 `--display-error-details` 来暂时详细信息。

阅读[配置][configuration]中`resolve`和加载器的解析（`resolveLoader`）章节。

### `npm link` 模块找不到依赖

node.js 的模块解析算法很简单：从模块所在目录递归向上找寻 `node_modules` 目录，然后再这些目录里查找带查找的模块是否存在。如果 `npm link` 了对等依赖（peer dependencies）不在当前根目录的模块时，将无法查找到该模块。（可以认为通过 `npm link` 对等依赖（`peerDependencies`是不被 node.js 支持的。）。即使不在 `package.json` 中列出来，但一个模块依赖于当前应用同样也是一种对等依赖，虽然这不是一个最好的设计）。

但在 webpack 中可以很简单地绕过这个问题：把 `node_modules` 目录添加到应用的解析路径中。而这个实现有两种方案：`resolve.fallback` 和 `resolveLoadr.fallback`。

下面是一个配置示例：

```js
module.exports = {
  resolve: { fallback: path.join(__dirname, "node_modules") },
  resolveLoader: { fallback: path.join(__dirname, "node_modules") }
};
```

## 监控

### webpack 不能监控变化然后重新编译

#### 监控数量达到最大值

确认系统是否有足够的监控数，如果不够，那么 webpack 的监控将无法检测到超出数量外的变更。

```bash
cat /proc/sys/fs/inotify/max_user_watches
```

Arch users, add  to /etc/sysctl.d/99-sysctl.conf and then execute sysctl --system. Ubuntu users (and possibly others): echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p.

Arch 用户，可以在 `/etc/sysctl.d/99-sys-ctl.conf` 中添加 `fs.inotify.max_user_watches=524288`，然后执行 `sysctl --system`。Ubuntu 用户（和同类比的）可以执行 ` echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p`。

#### OS-X fsevents 的 bug

OS-X 的目录会被损坏，具体参考：[OS X FSEvents bug may prevent monitoring of certain folders][osx fsevents bug]。

#### Windows 路径

webpack 的配置文件需要时绝对路径，因为 windows 的路径分隔符是 `\`，所以 `__dirname + "/app/folder"` 这样的代码将会出错，从而会导致问题。

需要使用正确的分隔符，如：`path.resolve(__dirname, "app/folder")` 或者 `path.join(__dirname, "app", "folder")`。

[configuration]: ./configuration.md
[osx fsevents bug]: http://feedback.livereload.com/knowledgebase/articles/86239-os-x-fsevents-bug-may-prevent-monitoring-of-certai
