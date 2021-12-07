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