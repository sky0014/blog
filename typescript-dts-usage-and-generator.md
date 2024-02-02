# 如何使用及生成 Typescript 的 dts 文件

## 编写DTS

创建`d.ts`文件，如`local.d.ts`

在其中编写你需要的类型代码，语法类似ts，只是多了一些关键字如`declare`等，详细可参考官方文档：
[https://www.typescriptlang.org/docs/handbook/declaration-files/by-example.html](https://www.typescriptlang.org/docs/handbook/declaration-files/by-example.html)

下面列举几个常用的场景：

### 案例1：添加全局属性

在window上添加全局属性`myprop`

```ts
declare interface Window {
  myprop: string;
}
```

### 案例2：给第三方库添加类型

假设第三方库名称是`third_lib`

如果该库没有自带类型定义，可以尝试在`@types`包寻找是否有其他人贡献的：

```bash
npm i @types/third_lib --save-dev
```

如果也没有的话，那么就可以自己在项目中添加他的类型定义：

```ts
declare module "third_lib" {
  const add: (a: number, b: number) => number;

  export default add;
}
```

### 案例3：给png、jpg等特定后缀文件添加类型

假设有如下代码：

```ts
import logo from './logo.png'

// logo.url 应该为该图片文件的地址
console.log(logo.url);
```

指定png图片默认导出对象中包含url字符串类型：

```ts
// 支持通配符 *
declare module "*.png" {
  const image: {
    url: string;
  };
  export default image;
}
```


## 使用DTS

要让dts生效，有以下两种办法，可任选其一：

### 方案1：include

假如你的`tsconfig`是这样配置的：

```json
{
  ...
  "include": ["./src/**/*"],
}
```

那么你只需要把你的dts文件，放到`src`任意目录下即可，这种方案是最简单的，不用修改`tsconfig`

如果你想把它放到src外面，也只需要在`include`后面把对应的目录加上即可。

### 方案2：typeRoots

将类型目录（假设目录名：types）加入到`typeRoots`配置项中

```json
// tsconfig.json
"compilerOptions" : {
    ...
    "typeRoots": [
        "node_modules/@types",
        "types"
    ]
}
```

- typeRoots：默认情况下包含`node_modules`下的所有`@types`包，如果这里手动指定的话，则只会使用你指定的路径，因此一般需要把默认的`node_modules/@types`也加上去。

## 在TS库中自动生成DTS

假如你用ts编写了一个库，给其他人使用的时候最好也能提供一份dts类型文件，以便于他人获得更好的类型提示。

如果自己手写，可能比较繁琐，本文将提供几种方法自动生成高质量的dts文件。

### rollup: 使用`rollup-plugin-dts`插件

如果你的库是使用`rollup`打包的，那很简单，只需要增加一个插件就可以了，具体配置如下：

```diff
// rollup.config.js

import typescript from "@rollup/plugin-typescript";
+ import dts from "rollup-plugin-dts";

export default [
  {
    input: "src/index.ts",
    output: {
      dir: "lib",
    },
    plugins: [typescript()],
  },
+  {
+    input: "src/index.ts",
+    output: [{ file: "lib/index.d.ts", format: "es" }],
+    plugins: [dts()],
+  },
];
```

详细使用说明可参考官方文档：[https://github.com/Swatinem/rollup-plugin-dts](https://github.com/Swatinem/rollup-plugin-dts)

### 通用：使用`dts-bundle-generator`工具

如果是使用其他工具打包的，可以使用下面的通用方案：

1. 打开typescript的`declaration`选项，编译时会同时生成d.ts类型文件，还可以通过`declarationDir`选项指定d.ts存放的目录

    ```json
    // tsconfig.json
    {
        "compilerOptions": {
            ...
            "declaration": true,
            "declarationDir": "./dist/types"
        }
    }
    ```

    注意，此时生成的只是单个ts文件对应的d.ts文件，并不是我们所需要的最终打包好的单一d.ts文件。

2. 使用`dts-bundle-generator`生成单一d.ts文件

    安装该工具：

    ```bash
    npm install --save-dev dts-bundle-generator
    ```

    使用命令行打包生成单一d.ts：

    ```bash
    dts-bundle-generator dist/types/index.d.ts -o dist/yourlib.d.ts --export-referenced-types false 
    ```

    - 参数o：指定输出文件路径
    - 参数export-referenced-types：指定是否将所有引用类型都export

    更多参数定制可参考官方文档：[https://github.com/timocov/dts-bundle-generator](https://github.com/timocov/dts-bundle-generator)

3. 清理types目录

    ```bash
    rm -rf ./dist/types
    ```

此方案不依赖于打包工具，适用范围更广，可参考使用之。