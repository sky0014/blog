# 在Nodejs中使用ESM模块

最近开发一个nodejs程序，发现运行时一直报错：

```
export function createLogger() {
^^^^^^

SyntaxError: Unexpected token 'export'
```

很明显是esm的问题。

但是我自己写的ts代码已经使用tsc转换为commonjs了，为何仍然报这个问题？

排查发现，我使用的一个第三方库，使用了esm的代码形式，tsc不会转换node_modules库里的代码，所以运行时就报错了。

查了下相关资料，虽然已经快2022年了，但是nodejs仍然没有提供比较完美的使用esm模块的方案，无论是`type: "module"`，还是`.mjs`，都有一定侵入性和副作用，而且要求`import`时还需要带文件后缀，自己的代码还好，可以配合改改，如果是第三方库中使用了esm，就很头疼了。

幸而最终发现了一个非常magical的库，可以简单而优雅的解决这个问题：[https://github.com/standard-things/esm](https://github.com/standard-things/esm)

使用起来非常简单，首先安装该模块：

```
npm i esm --save
```

然后，运行node程序时加个参数：

```
node -r esm your_esm.js
```

All done！不用改任何其他东西，兼容标准的esm写法，无论是自己的代码还是库里的代码，都没问题，感谢作者，从此以后你就可以在node程序里愉快的使用标准的esm了，用作者的话说就是：

```
Tomorrow's ECMAScript modules today!
```

更多用法请参考项目主页。