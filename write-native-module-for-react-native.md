# 为ReactNative编写原生模块

整体参考：[https://reactnative.dev/docs/native-modules-intro](https://reactnative.dev/docs/native-modules-intro)

这里将内容精炼，并记录一些自己编写时遇到的问题。

## Android

[https://reactnative.dev/docs/native-modules-android](https://reactnative.dev/docs/native-modules-android)

### 支持Kotlin

react-native生成的android代码中，默认是没有配置kotlin支持的，这样如果你编写kotlin代码，编译时会报错：“找不到符号”

下面介绍一下如何给react-native android代码手动添加kotlin支持

## Ios

TODO...