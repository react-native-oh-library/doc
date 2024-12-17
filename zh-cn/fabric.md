# Fabric 组件

Fabric 组件是一种使用 Fabric 渲染器渲染并展示在屏幕上的 UI 组件。

![fabric](../img/fabric.png)

在开发 Fabric 组件前，需要先创建一个 JavaScript 接口描述文件。之后 Codegen 会根据这个文件创建一些 C++ 脚手架代码，用于将部分组件逻辑（比如调用原生平台接口能力）与 React Native 结合起来。C++ 代码在各个平台都是一样的，只要组件能够与生成的 C++ 代码连接起来，就可以导入到 App 并运行。

## 如何创建 Fabric 组件

若要创建一个 Fabric 组件，需要遵循以下步骤：

1. 声明 JavaScript 接口；
2. 配置组件以用于 Codegen 生成统一代码，生成的代码可添加为 App 的依赖；
3. 编写所需的原生代码。

接下来会创建一个简单的名为 `RTNCenteredText` 的 Fabric 组件作为示例。

**该示例仅提供 HarmonyOS 版本，Android/iOS 版本请阅读 React-Native 官方文档的 [Fabric 组件章节](https://reactnative.cn/docs/the-new-architecture/pillars-fabric-components)**

### 1. 目录配置

首先，需要有一个已配置好支持 HarmonyOS 的 RN 工程，以下简称 `exampleApp`。

在 `exampleApp` 同级目录下新建三方库目录 `RTNCenteredText`。

并在 `RTNCenteredText` 下新建 `harmony` 和 `src` 文件夹。

```
.
├── exampleApp
└── RTNCenteredText
    ├── harmony（HarmonyOS 的原生实现代码）
    └── src （js/ts代码）
```

### 2. 声明 JavaScript 接口

[React-Native 新架构](https://reactnative.cn/docs/the-new-architecture/landing-page) 要求必须使用强类型风格语言声明 JavaScript 接口（Flow 和 TypeScript 皆可）。Codegen 会根据这些接口声明来生成强类型的语言，其中包括 C++ 和 ArkTS。

对于声明类型的代码文件必须满足以下两点要求：

1. 文件必须使用 `<MODULE_NAME>NativeComponent` 命名，在使用 Flow 时，以 `.js` 或 `.jsx` 为后缀名；在使用 Typescript 时，以 `.ts` 或 `.tsx` 为后缀名。Codegen 只会找到匹配这些命名规则的文件；

2. 代码中必须要输出 `HostComponent` 对象。

以下是使用 Flow 和 TypeScript 声明的 RTNCenteredText 组件。在 `src` 目录中，创建一个命名为 `RTNCenteredText` 并带有相应后缀名的文件。（Flow 和 TypeScript 选择其一即可）

#### **typescript**

RTNCenteredTextNativeComponent.ts

```ts
import type * as React from 'react';
import type {ViewProps} from 'ViewPropTypes';
import type {ColorValue, HostComponent} from 'react-native';
import codegenNativeComponent from 'react-native/Libraries/Utilities/codegenNativeComponent';
import type {
  DirectEventHandler,
  Int32,
} from 'react-native/Libraries/Types/CodegenTypes';

export type OnTouchEventData = Readonly<{
  type: Int32;
}>;

export interface NativeProps extends ViewProps {
  text?: string;
  color?: ColorValue,
  onTextTouch?: DirectEventHandler<OnTouchEventData>;
  // 在这里添加其他 props
}

export default codegenNativeComponent<NativeProps>(
  'RTNCenteredText',
) as HostComponent<NativeProps>;
```

<!-- tabs:end -->

<!-- tabs:start -->

#### **flow**

RTNCenteredTextNativeComponent.js

```js
// @flow strict-local

import type {ViewProps} from 'react-native/Libraries/Components/View/ViewPropTypes';
import type {HostComponent} from 'react-native';
import codegenNativeComponent from 'react-native/Libraries/Utilities/codegenNativeComponent';
import type {
  DirectEventHandler,
  Int32,
} from 'react-native/Libraries/Types/CodegenTypes';

export type OnTouchEventData = $ReadOnly<{|
  type: Int32,
|}>;

export type NativeProps = $ReadOnly<{|
  ...ViewProps,
  text?: string,
  color?: ColorValue,
  onTextTouch?: DirectEventHandler<OnTouchEventData>,
  // add other props here
|}>;

export default (codegenNativeComponent<NativeProps>(
   'RTNCenteredText',
): HostComponent<NativeProps>);
```

在声明文件的顶部导入了一些内容。以下是开发 Fabric 组件必须要导入的内容：

- `HostComponent` 类型: 导出的组件需要与这个类型保持一致；
- `codegenNativeComponent` 函数：负责将组件注册到 JavaScript 运行时。
  声明文件的中间部分包含了组件的 **props**。Props（"properties" 的缩写）是用于自定义 React 组件的参数信息。在本例中，需要控制组件的 `text` 属性。

在声明文件的最后部分，导出了泛型函数 `codegenNativeComponent` 的返回值，此函数需要传递组件的名称。

将编写好的声明类型代码文件文件放入 `RTNCenteredText/src`

### 3. 选择 Fabric 的原生实现方式（重要），然后开发原生代码

RNOH 有特殊的架构限制，需要开发者在开发前根据需求选择好使用 ArkTS API 还是 C-API 实现 Fabric。

关于如何选择，请阅读这篇[说明文档](https://gitee.com/react-native-oh-library/usage-docs/blob/master/zh-cn/capi-architecture.md)。

`注意：开发容器组件目前只支持使用 CAPI 实现。`

请根据实际情况选择：

- [使用 ArkTS API 开发 Fabric](/zh-cn/fabric-arkts.md)
- [使用 C-API 开发 Fabric](/zh-cn/fabric-capi.md)


