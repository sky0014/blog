# 搭建react native开发环境

## 安装nodejs

官网下载安装包安装：[https://nodejs.org/en/](https://nodejs.org/en/)

装完后可以设置下使用国内源：

```
npm config set registry=https://registry.npm.taobao.org/
```

## 安装android studio

官网下载安装包安装：[https://developer.android.google.cn/studio/](https://developer.android.google.cn/studio/)

安装时可勾上安装模拟器，方便电脑上测试

装完后，使用android studio自带的SDK管理工具，安装android sdk

再使用模拟器管理工具，创建一个模拟器

然后设置相应的环境变量

```
ANDROID_HOME
ANDROID_SDK_ROOT

%ANDROID_HOME%\platform-tools
%ANDROID_HOME%\emulator
%ANDROID_HOME%\tools
%ANDROID_HOME%\tools\bin
```

![image](https://user-images.githubusercontent.com/6689073/150129016-7a7326a4-e1be-4453-af59-3f6244faabc1.png)

![image](https://user-images.githubusercontent.com/6689073/150142518-8c76a9d3-65da-4381-9add-79c3288028d1.png)

## 配置jdk

android studio内置了jdk，直接使用它的就可以了，以我本机为例，可以在如下地址找到：
```
D:\Program Files\Android\Android Studio\jre
```

然后设置环境变量
```
JAVA_HOME

%JAVA_HOME%\bin
```

![image](https://user-images.githubusercontent.com/6689073/150142111-363d12ae-3f29-43fb-ace4-b7871a58fc8b.png)

![image](https://user-images.githubusercontent.com/6689073/150142358-2ce778cf-8148-48de-806f-e271c27c2685.png)


## 创建react native工程

使用react native cli工具创建工程：
```
npx react-native init AwesomeProject
```

当然你也可以指定不同模板，比如使用typescript模板：
```
npx react-native init AwesomeTSProject --template react-native-template-typescript
```

安装完之后，进入项目目录执行
```
npm run android
```

此时你的app就可以在模拟器里面跑起来了，enjoy!

![image](https://user-images.githubusercontent.com/6689073/150142686-487f5942-4c50-4f71-b239-209be7322e1a.png)

## 相关链接

### 学习资料

- [ReactNative官网](https://reactnative.dev/)
- [ReactNative中文网](https://reactnative.cn/)

### UI组件库

- [React Native Elements](https://reactnativeelements.com/)（较成熟，推荐）
- [ant design for react native](https://rn.mobile.ant.design/docs/react/introduce-cn)
