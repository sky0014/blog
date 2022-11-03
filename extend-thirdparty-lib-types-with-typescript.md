# 使用Typescript扩展第三方库的类型

有时，我们使用的第三方库类型可能有部分属性缺失，或者我们想要添加一些自定义属性，这时我们可以通过下面的方法对其类型进行扩展。

假如第三方库是：`sample-lib`

1. 创建一个文件夹`types`，其下创建一个类型文件`global.d.ts`，内容如下：（命名都可以自定义）

```ts
// 引入原类型
import "sample-lib";

// 扩展类型字段
declare module "sample-lib" {
  interface SampleType {
    myProp?: string;
  }
}
```

2. 修改`tsconfig.json`，增加或修改`baseUrl`和`path`属性，告诉typescript类型文件所在的地址

```json
{
  "compilerOptions": {
    ...
    "baseUrl": ".",
    "paths": {
      "*": ["types/*"]
    }
  }  
}
```

通过以上简单两步，就成功扩展了第三方库的类型，写代码时会有智能提示，编译时也不会报错了。

当然，如果是第三方库的类型确实有问题，最好是能提PR去修复，一起共建良好的开源社区~