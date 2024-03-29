# 为ReactNative编写原生模块

整体参考：[https://reactnative.dev/docs/native-modules-intro](https://reactnative.dev/docs/native-modules-intro)

这里将内容精炼，并记录一些自己编写时遇到的问题。

## Android

### 编写原生模块

[https://reactnative.dev/docs/native-modules-android](https://reactnative.dev/docs/native-modules-android)

参考官方文档，已经写得很详细了。

### Android Studio编译乱码问题

如果使用kotlin编写原生模块，运行`npm run android`时，编译会失败，控制台错误信息为乱码，使用Android Studio编译也一样。

原因是react native默认生成的代码没有配置对kotlin的支持，如何配置支持kotlin可看下一章，这里写一下乱码的解决办法：

进入Android Studio安装目录，修改以下两个文件

![image](https://user-images.githubusercontent.com/6689073/182283956-b6706d8c-1886-443b-86dc-29e47bdc548d.png)

在最后增加一行：

```
-Dfile.encoding=UTF-8
```

重启Android Studio，再编译时即可正常显示中文。

### 支持Kotlin

[https://developer.android.com/kotlin/add-kotlin](https://developer.android.com/kotlin/add-kotlin)

react-native生成的android代码中，默认是没有配置kotlin支持的，这样如果你编写kotlin代码，编译时会报错：“找不到符号”

下面介绍一下如何给react-native android代码手动添加kotlin支持：

在`Project build.gradle`中添加：

```groovy
buildscript {
    ext {
        ...
        kotlinVersion = "1.7.20-Beta"
    }
    
    dependencies {
        ...
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlinVersion")
    }    
}
```

在`Module build.gradle`中添加：

```groovy
apply plugin: "kotlin-android"

dependencies {
    implementation "androidx.core:core-ktx:1.8.0"
    implementation "org.jetbrains.kotlin:kotlin-stdlib:$kotlinVersion"
}
```

之后kotlin代码就可以被正确的编译了。

## iOS

TODO...
