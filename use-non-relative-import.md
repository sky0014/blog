# 使用非相对路径（non-relative）引用代码文件

es module 默认是使用相对路径进行文件引用，相信大家经常会看到类似下面这种`import`：

```js
import * as utils from "../../../../../../../shared/utils";
```

这种引用太长且不直观，如果可以改成非相对路径就好多了：

```js
import * as utils from "src/shared/utils";
```

本文会展示如何进行配置以达到上述效果。

## 配置`baseUrl`和`paths`

在你的`tsconfig.json`或`jsconfig.json`文件中，新增两项：

```json
{
  //...
  "baseUrl": ".",
  "paths": {
    "src/*": ["./src/*"]
  }
}
```

## 配置 webpack

除了上面的配置外，webpack 打包时也需要能识别出这种路径，有两种方法：

### 方法 1

在`webpack.config.js`中手动添加：

```js
module.exports = {
  //...
  resolve: {
    alias: {
      src: path.resolve(process.cwd(), "src/"),
    },
  },
};
```

### 方法 2

方法 1 需要保持与`tsconfig.json`中的配置一样，不太优雅，如果能自动保持同步就好了。

幸运的是，已经有人写了这种 webpack 插件：[tsconfig-paths-webpack-plugin](https://github.com/dividab/tsconfig-paths-webpack-plugin)，感谢作者！

首先安装：

```
npm install --save-dev tsconfig-paths-webpack-plugin
```

然后修改`webpack.config.js`：

```js
const TsconfigPathsPlugin = require('tsconfig-paths-webpack-plugin');

module.exports = {
  ...
  resolve: {
    plugins: [new TsconfigPathsPlugin({/* 详细配置参考项目README */})]
  }
  ...
}
```

注意：与常规 plugin 不同，这个 plugin 要放在`resolve`项下，不要弄错了。

这样就可以自动保持与`tsconfig.json`中的配置同步了。

## 配置 vscode

vscode 自动 import 时，仍然会使用相对路径，需要修改 vscode 的配置项。

在 vscode 设置中，搜索：`importModuleSpecifier`，有 js 和 ts 两项，都改为`non-relative`，如下图所示：

![image](https://user-images.githubusercontent.com/6689073/223684498-9131dad6-6674-4504-8f9c-13bfeab51c3d.png)

当然，我们还有更简单的办法，可以直接修改项目中的 vscode 配置文件`.vscode/settings.json`，让配置可以跟着项目走：

```json
{
  //...
  "javascript.preferences.importModuleSpecifier": "non-relative",
  "typescript.preferences.importModuleSpecifier": "non-relative"
}
```

通过上述配置之后，import 就会自动使用非相对路径了，从此告别`../`

## 参考

- [https://johnnyreilly.com/typescript-webpack-alias-goodbye-relative-paths](https://johnnyreilly.com/typescript-webpack-alias-goodbye-relative-paths)
- [https://stackoverflow.com/questions/52432191/auto-import-in-visual-studio-code-only-offering-absolute-path-with-lerna-subpack](https://stackoverflow.com/questions/52432191/auto-import-in-visual-studio-code-only-offering-absolute-path-with-lerna-subpack)
