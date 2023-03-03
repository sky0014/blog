# 使用 Jest 测试你的 React、React Native 代码

单元测试对于保障代码质量非常重要，特别是对于底层基础代码，非常有必要。

同时，写单元测试的过程中，也可以发现你代码中以前未考虑到的场景，提升代码的健壮性。

本文会帮助你一步一步为你的代码搭建单测环境，放心，有了优秀开源库的帮助，这并不复杂。

## 测试工具的选择

参考：[https://reactjs.org/docs/testing-environments.html](https://reactjs.org/docs/testing-environments.html)

- 单元测试：使用 [Jest](https://jestjs.io/)

  > Jest 是被广泛使用的优秀的测试工具，其测试主要运行于 node 环境中，通过`jsdom`模拟 dom 环境，因此无法去测试真实浏览器的某些特性，主要用于单元测试

- 浏览器测试：使用 [Mocha](https://mochajs.org/)

  > Mocha 可以运行在真实浏览器环境中，因此如果有这种需要，可以使用 Mocha

- 复杂自动化测试：使用 [Puppeteer](https://pptr.dev/)
  > 对于更复杂的操作，可以编写脚本，使用 Puppeteer 自动化控制浏览器进行测试

一般情况下 Jest 已经足够满足测试需求，下面也主要以 Jest 为例进行介绍。

## 配置 Jest

安装 Jest 相关包：

```
npm i jest @types/jest jest-environment-jsdom @testing-library/jest-dom --save-dev
```

在项目目录下新建文件`jest.config.js`，此即 Jest 的配置文件，主要内容如下：

```js
module.exports = {
  testEnvironment: "jsdom", // 指定测试环境jsdom，默认是node
  setupFilesAfterEnv: ["@testing-library/jest-dom/"], // 自动加载jest-dom
};
```

package.json 中新增 script：

```json
  "scripts": {
    ...
    "test": "jest",
  },
```

新建 test 目录，编写测试用例`sample.test.js`：

```js
it("sample", () => {
  expect(0).toBe(0);
});
```

Jest 默认会自动寻找 test 目录以及\*.test.js 文件执行测试，当然也可以通过配置指定目录及匹配文件。

详细配置项及测试用例编写教程可参考 Jest 官网：[https://jestjs.io/](https://jestjs.io/)

最后，执行`npm test`，即可触发 Jest 执行测试用例。

## 支持 React 和 Typescript

如果使用了 React 和 Typescript，需要将代码进行转换，以让其可以在 node 环境中运行。

Jest 默认即支持 babel 转换，你只需要配置 babel 转换规则即可。

> 你也可以使用[ts-jest](https://www.npmjs.com/package/ts-jest)进行转换，然而我个人在实际使用中发现一些覆盖率相关的代码定位问题，以及调试相关问题，因此后面还是使用 babel 进行配置，参考此文：[https://blog.oluwasetemi.dev/migrating-a-react-code-base-test-from-using-ts-jest-to-babel-jest](https://blog.oluwasetemi.dev/migrating-a-react-code-base-test-from-using-ts-jest-to-babel-jest)

首先安装 babel 相关包：

```
npm i @babel/preset-env @babel/preset-react @babel/preset-typescript --save-dev
```

项目目录下创建 babel 配置文件`babel.config.js`：

```js
module.exports = {
  presets: [
    ["@babel/preset-env", { loose: true }],
    "@babel/preset-typescript",
    "@babel/preset-react",
  ],
};
```

无需其他配置，Jest 会自动读取 babel 配置并将其应用。

## 支持 React Native

Jest 同样支持测试 React Native。

首先安装相关包：

```
npm i @testing-library/react-native @testing-library/jest-native metro-react-native-babel-preset --save-dev
```

注意，RN 的 babel 转换规则与 web 不同，因此这里我们安装了包含 RN 规则的包：`metro-react-native-babel-preset`，下面会将其设置到 babel 配置中。

修改`jest.config.js`：

```js
module.exports = {
  displayName: "React Native", // 运行测试时展示的名字
  testEnvironment: "node", // RN需要设置为node
  preset: "react-native",
  setupFilesAfterEnv: ["@testing-library/jest-native/extend-expect"],
  testMatch: ["<rootDir>/test/native/**/*.[jt]s?(x)"], // 指定native测试用例位置
  transformIgnorePatterns: [
    // 无需Jest转换的代码添加到这里，不如部分esm包（注意要增加时在后面增加|条件，而不是新增一项）
    "/node_modules/(?!(@react-native|react-native)/).*",
  ],
  // RN转换规则与web不同，在这里强制指定，覆盖babel.config.js中的规则
  transform: {
    // must overwrite react-native jest-preset transform key (^.+\\.(js|ts|tsx)$)
    "^.+\\.(js|ts|tsx)$": [
      "babel-jest",
      {
        presets: ["module:metro-react-native-babel-preset"],
      },
    ],
  },
  // 原测试用例中使用的包可以采用这种方式重定向
  moduleNameMapper: {
    "^@testing-library/react$": "@testing-library/react-native",
  },
};
```

这样配置后，使用 Jest 就可以正常测试你的 RN 代码了。

## 同时支持 React17、React18 以及 React Native

有时，我们的测试用例想要运行在不同环境下，比如 React17、React18 以及 React Native，Jest 也提供了这种功能。

在 Jest 配置中，通过`moduleNameMapper`项可以将包进行重定向，我们项目中本来使用的是 React18，如果想将其切换到 React17 下运行，可以做如下配置：

```js
module.exports = {
  testEnvironment: "jsdom", // 指定测试环境jsdom，默认是node
  setupFilesAfterEnv: ["@testing-library/jest-dom/"], // 自动加载jest-dom
  displayName: "React 17",
  moduleNameMapper: {
    "^react$": "react-17",
    "^react-dom$": "react-dom-17",
    "^react-dom/test-utils$": "react-dom-17/test-utils", // `act` is here
    "^react-test-renderer$": "react-test-renderer-17",
    "^@testing-library/react$": "@testing-library/react-12",
  },
};
```

在`package.json`中：

```json
    "react": "^18.2.0",
    "react-17": "npm:react@^17",
    "react-dom": "^18.2.0",
    "react-dom-17": "npm:react-dom@^17",
    ...
```

要将多种环境统一到一起进行测试，可以在`jest.config.js`中配置`projects`项：

```js
const react17 = ...
const react18 = ...
const rn = ...

module.exports = {
  projects: [react17, react18, rn],
};
```

此时，运行`npm test`，Jest 就会自动切换相关环境，一次性测试完毕。

## 代码覆盖率

代码覆盖率是单元测试一个很重要的指标，我们最好将代码尽量做到全覆盖，Jest 也集成了检测代码覆盖率的功能，底层使用的是[Istanbul](https://github.com/gotwarlost/istanbul)

使用方法：

```
jest --coverage
```

可以修改`package.json`中的 script：

```json
  "scripts": {
    ...
    "test": "rimraf coverage && jest --coverage",
  },
```

此时，运行`npm test`即会自动在项目目录下生成`coverage`文件夹，其中包含代码覆盖率的相关信息。通过下图标出的 `index.html` 文件，可以在浏览器中很方便的查看相关信息：

![image](https://user-images.githubusercontent.com/6689073/222718396-9c5e43f2-1b41-4a97-b4c7-e10e7403b420.png)

![image](https://user-images.githubusercontent.com/6689073/222718699-ead27633-462a-4330-92a1-68380be6ad1c.png)

![image](https://user-images.githubusercontent.com/6689073/222718841-4f92c9a2-d423-439a-b76e-73b837364e5c.png)

同时，如果你有部分代码无需检测，为避免拉低整体覆盖率，可使用如下指令进行忽略：

- `/* istanbul ignore file */` 忽略整个文件
- `/* istanbul ignore next */` 忽略下一句
- `/* istanbul ignore else */` 忽略 else 语句

## 技巧

### 只运行某个用例

假如我们在项目中，已经写了很多的测试用例，在写一个新用例时，可能需要反复调整，如果每次调整都执行`npm test`观察，会将全部用例都运行一遍，太浪费时间。

Jest 提供了简单的办法可以让我们只运行某一个用例，只需要在用例后面加上`.only`即可，其他用例都会被自动跳过，这样我们调整用例的效率可以大大提升，示例如下：

```js
it('test1', ()=> { ... })
it('test1', ()=> { ... })
it('test1', ()=> { ... })
it('test1', ()=> { ... })
// only run this test
it.only('new test', ()=> { ... })
```

### 调试

运行 Jest 时，我们同样可以进行调试，帮助我们分析问题。

首先在需要观察的地方打上断点，然后在`package.json`的`script`选项上选择`调试`，再选择`test`即可：

![image](https://user-images.githubusercontent.com/6689073/222723094-10c36bef-d73f-46d9-83ca-9dcf5ca3dcb1.png)

![image](https://user-images.githubusercontent.com/6689073/222727713-842ee597-e871-4605-8112-a836fdfae0a9.png)

## 完整示例

以上所述内容的完整实践示例可参考此项目（PS：欢迎使用及 star ，相信一定会让你的开发更简单）：

[https://github.com/sky0014/store](https://github.com/sky0014/store)
