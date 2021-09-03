# Cross Origin Isolation：如何进行跨域隔离

Chrome92开始，如想使用`SharedArrayBuffer`对象，需对网站进行跨域隔离（Cross Origin Isolation），本文将描述如何进行跨域隔离及对其的一些思考。

## 开启跨域隔离

在HTML Document请求返回中，增加两个header：

```
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Opener-Policy: same-origin
```

此时即对当前页面开启了跨域隔离，可以通过如下代码检测：

```js
if (self.crossOriginIsolated) {
  // 成功开启，可以使用SharedArrayBuffer等API
} else {
  // 未开启
}
```

这就完了吗？事情当然没有那么简单。一般情况下，此时你的页面上会报出大量的跨域相关错误，js/css/png等各种都有，只要是第三方域名链接的地址，都可能报错。

**处理这些跨域错误，才是真正麻烦的地方，也是最容易导致故障的地方。**

![image](https://user-images.githubusercontent.com/6689073/131970377-17d2b27f-c415-4279-9ab6-98828e8c4c35.png)

在devtool中，发生跨域错误的请求，会有如下展示：

![image](https://user-images.githubusercontent.com/6689073/131972862-96583eaa-fa3b-4483-a649-db763b9af6cf.png)

## 处理跨域报错

要解决这个问题，无论是何种链接，都可以按照下面的方法处理，以js为例：

1. 声明为跨域资源，有两种操作方式，任取其一：
    - 在html tag上增加`crossorigin`属性
      ```html
      <!--before-->
      <script src="a.js"></script>
      <!--after-->
      <script src="a.js" crossorigin></script>
      ```
    - 在资源http返回头中增加：
      ```
      cross-origin-resource-policy:cross-origin
      ```

2. 资源增加跨域头
```
// 使用*允许所有域名，或者只允许你需要的域名，只能写一个域名，不能有通配符，如有多个域名可通过脚本控制返回
Access-Control-Allow-Origin:https://www.example.com
```
这个大家应该都很熟悉了，只不过以前js/css/png这些资源都是不检测跨域的，但是如果开启了跨域隔离，这些资源都会变得跟其他资源一样需要进行跨域检测。

**这也就意味着，需要梳理所有页面上可能引用到的第三方资源地址，并在客户端或服务端都进行上述处理，如有遗漏，则会造成相应功能的故障。**

对于现代大型网站来说，这是一项非常艰巨的工作，特别是可能还大量的使用了云、CDN等资源，沟通处理起来会更麻烦。

## 个人理解

18年1月，著名的`Spectre`和`Meltdown`安全漏洞被公布，他们利用的是CPU架构上的漏洞进行攻击，不更新硬件很难解决。

在网页中，可以利用高精度计时器和`SharedArrayBuffer`发起攻击，因此，Chrome曾经禁用过相关API一段时间，避免遭受攻击。

现在，他们通过跨域隔离，尝试在提供相关API能力的同时，又能解决安全问题。

假如网站`evil.com`想要发起攻击，那么首先他需要声明自己进行跨域隔离，跨域隔离之后，访问所有的第三方资源都需要对方服务器的授权，如第三方未授权，则无法访问，因此也无法收到攻击。但是假如第三方跨域头配置的是`*号`（很常见），则仍然有可能遭受攻击。

**跨域隔离简单来说就是对当前页面进行强约束，所有第三方资源都需要授权，从而保障自身有比较高的安全性，这种安全环境下，允许开发者使用`SharedArrayBuffer`等API。**

## 参考
  
1. [https://developer.chrome.com/blog/enabling-shared-array-buffer/#cross-origin-isolation](https://developer.chrome.com/blog/enabling-shared-array-buffer/#cross-origin-isolation)
2. [Chrome 91 将启用针对 SharedArrayBuffers 的跨域限制](https://blog.p2hp.com/archives/8026)
3. [Reading privileged memory with a side-channel](https://googleprojectzero.blogspot.com/2018/01/reading-privileged-memory-with-side.html)
4. [Cross-Origin Resource Policy (CORP)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cross-Origin_Resource_Policy_(CORP))

  
