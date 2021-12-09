# 使用Electron创建桌面应用

## 创建应用

[`Electron Forge`](https://www.electronforge.io/)是一套构建现代Electron应用的完整工具，包括创建、编译、打包、发布等，使用它可以快速的开发一个Electron跨端应用。

### 设置npm

electron库在国内很容易因为网络原因安装失败，可以先设置一下使用国内镜像：

```bash
npm config set registry=https://registry.npm.taobao.org/
npm config set ELECTRON_MIRROR=https://npm.taobao.org/mirrors/electron/
```

### 从模板创建项目

```bash
npx create-electron-app@latest my-app
```
这个命令会创建最基础的`Electron Forge`模板代码，包含`Electron Forge`的相关构建命令以及最简单的js/css/html样板代码。

如果你像使用一些更高级的特性如：typescript、webpack，它也提供了相应的模板：
```bash
npx create-electron-app my-new-app --template=typescript
#npx create-electron-app my-new-app --template=typescript-webpack
#npx create-electron-app my-new-app --template=webpack
```

当然你也可以自己在最小模板的基础上进行改造，使用任何你需要的前端工具、特性等。

上述命令会自动安装相关依赖，命令执行完之后，进入目录，执行`npm start`就可以看到界面，进行开发了。

### 支持React

#### 安装React和eslint规则
```bash
npm i react react-dom --save
npm i eslint-plugin-react --save-dev
```

#### 修改tsconfig.json和.eslintrc.json
```json
// tsconfig.json
{
  "compilerOptions": {
    ...
    "jsx": "react-jsx"
  }
}
```
如果是react17之前的版本，这里要使用`"jsx": "react"`，其区别是：
- react 传统方式，使用`React.createElement`进行转换，需要在文件里显式引入React
- react-jsx react17支持的新的转换方式，使用`_jsx`转换，文件里不再需要显式引入React
    ```js
    // 源代码
    export const helloWorld = () => <h1>Hello world</h1>;

    // react-jsx转换后
    // 由编译器引入（禁止自己引入！）
    import { jsx as _jsx } from "react/jsx-runtime";    
    export const helloWorld = () => _jsx("h1", { children: "Hello world" }, void 0);
    ```

由于使用react-jsx之后，不再需要显式引入React库，可能造成eslint警告，在`.eslintrc.json`中添加`plugin:react/jsx-runtime`即可兼容，最终eslint修改如下：
```json
// .eslintrc.json
{  
  "extends": [
    ...
    "plugin:react/recommended",
    "plugin:react/jsx-runtime"
  ],
  "parserOptions": {
    "ecmaFeatures": {
      "jsx": true
    }
  }  
}
```

#### 修改文件名为`*.tsx`

这一步踩坑了，在之前的`.ts`文件里调了很久一直无法识别jsx语法，最后发现是要将文件后缀从`.ts`改成`.tsx`，这样才能识别到。
