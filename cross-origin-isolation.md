# Cross Origin Isolation：如何进行跨域隔离

Chrome92开始，如想使用`SharedArrayBuffer`对象，需对网站进行跨域隔离（Cross Origin Isolation），本文将描述如何进行跨域隔离及对其的一些思考。

1. HTML Document请求返回中，增加两个header：

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

这就完了吗？事情当然没有那么简单。一般情况下，此时你的页面上会报出大量的跨域相关错误，js/css/png等各种都有，只要是第三方域名链接的地址，都可能报错，这里才是真正麻烦的地方，也是最容易出错导致故障的地方。

![image](https://user-images.githubusercontent.com/6689073/131970377-17d2b27f-c415-4279-9ab6-98828e8c4c35.png)

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
Access-Control-Allow-Origin:* 
```
这个大家应该都很熟悉了，只不过以前js/css/png这些资源都是不检测跨域的，但是如果开启了跨域隔离，这些资源都会变得跟其他资源一样，会进行跨域检测。
  
  
