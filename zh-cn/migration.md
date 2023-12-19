# 迁移三方库到新框架

> [!tip] 迁移到新框架分为两步：JS 代码迁移和原生代码迁移。这里只介绍 JS 代码的迁移，Android 和 iOS 原生代码的迁移与鸿蒙化无关，按需自行学习。

> [!tip] 可通过社区查找参考代码，在仓库的 pull request 搜索 new architecture 或者 Fabric/Turbo Module 以查看社区里迁移中的代码，进行参考。

## 如何判断三方库是否支持新框架

有三个方法来判断：

1. 在 `package.json` 里判断是否使用了 Codegen，即搜索是否存在 "codegenConfig" 字段；
2. 是否有 Native<MODULE_NAME> 命名的 js/ts 文件并且里面导出了 TurboModuleRegistrySpec 对象；
3. 是否有 <MODULE_NAME>NativeComponent 命名的 js/ts 文件并且里面导出了 codegenNativeComponent 对象。

一般用第一个方法就能快速识别。

## 在 JavaScript 中支持新架构的预备工作

### 在 JavaScript 中定义 Specs

JavaScript Specs 是每个 Native Module 提供的方法的真实来源，定义了所有 Native Module 提供的 API，以及这些常量和函数的类型。使用类型 Spec 文件允许您有意地声明 Native Module 方法的所有输入参数和输出。

要适配新架构，首先需要创建你的 Native Modules 和 Native Component 的 Specs。这些 Specs 在之后将被 Codegen 工具使用来为所有受支持的平台生产原生接口代码，作为跨平台强制统一 API 的一种方式。

新架构要求必须使用强类型风格语言声明 JavaScript 接口（Flow 和 TypeScript 皆可）。Codegen 会根据这些接口声明来生成强类型的语言，其中包括 C++、Objective-C 和 Java。

> [!tip] 目前，harmony 平台的 codegen 还在开发当中。

#### Turbo Native Modules

TurboModules 对于声明类型的代码文件必须满足以下两点要求：

1. 文件必须使用 `Native<MODULE_NAME>`命名，在使用 Flow 时，以 .js 或 .jsx 为后缀名；在使用 **Typescript** 时，以 `.ts` 或 `.tsx` 为后缀名。**Codegen** 只会找到匹配这些命名规则的文件；
2. 代码中必须要输出 `TurboModuleRegistrySpec` 对象。

以下是一个基本的 TurboModules JavaScript Specs 模板，使用 Flow 和 TypeScript 语法。

<!-- tabs:start -->

#### **Flow**

`Native<MODULE_NAME>.js`

```js
// @flow

import type { TurboModule } from "react-native/Libraries/TurboModule/RCTExport";
import { TurboModuleRegistry } from "react-native";

export interface Spec extends TurboModule {
  +getConstants: () => {||};

  // your module methods go here, for example:
  getString(id: string): Promise<string>;
}

export default (TurboModuleRegistry.get<Spec>("<MODULE_NAME>"): ?Spec);
```

#### **Typescript**

`Native<MODULE_NAME>.ts`

```ts
import type { TurboModule } from "react-native";
import { TurboModuleRegistry } from "react-native";

export interface Spec extends TurboModule {
  readonly getConstants: () => {};

  // your module methods go here, for example:
  getString(id: string): Promise<string>;
}

export default TurboModuleRegistry.get<Spec>("<MODULE_NAME>");
```

<!-- tabs:end -->

#### Turbo Native Modules 迁移例子

| 库名                                      | 新架构适配 pull request                                                                                                      |
| ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| @react-native-async-storage/async-storage | [Convert library to TurboModule #910](https://github.com/react-native-async-storage/async-storage/pull/910/files)            |
| react-native-image-picker                 | [Add new architecture support #2082](https://github.com/react-native-image-picker/react-native-image-picker/pull/2082/files) |

#### Fabric Native Components

Fabric Components 对于声明类型的代码文件必须满足以下两点要求：

1. 文件必须使用 `<FABRIC COMPONENT>NativeComponent` 命名，在使用 Flow 时，以 `.js` 或 `.jsx` 为后缀名；在使用 Typescript 时，以 `.ts` 或 `.tsx` 为后缀名。Codegen 只会找到匹配这些命名规则的文件；
2. 代码中必须要输出 HostComponent 对象。

以下是一个基本的 Fabric Components JavaScript Specs 模板，使用 Flow 和 TypeScript 语法。

<!-- tabs:start -->

#### **Flow**

`<FABRIC COMPONENT>NativeComponent.js`

```js
// @flow strict-local

import type { ViewProps } from "react-native/Libraries/Components/View/ViewPropTypes";
import type { HostComponent } from "react-native";
import codegenNativeComponent from "react-native/Libraries/Utilities/codegenNativeComponent";

type NativeProps = $ReadOnly<{|
  ...ViewProps,
  // add other props here
|}>;

export default (codegenNativeComponent<NativeProps>(
  "<FABRIC COMPONENT>"
): HostComponent<NativeProps>);
```

#### **Typescript**

`<FABRIC COMPONENT>NativeComponent.ts`

```ts
import type { ViewProps } from "ViewPropTypes";
import type { HostComponent } from "react-native";
import codegenNativeComponent from "react-native/Libraries/Utilities/codegenNativeComponent";

export interface NativeProps extends ViewProps {
  // add other props here
}

export default codegenNativeComponent<NativeProps>(
  "<FABRIC COMPONENT>"
) as HostComponent<NativeProps>;
```

<!-- tabs:end -->

#### Fabric Native Components 迁移例子

| 库名                                      | 新架构适配 pull request                                                                                                      |
| ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| @react-native-community/slider | [Add Fabric support #410](https://github.com/callstack/react-native-slider/pull/410/files)            |
| @react-native-picker/picker                 | [feat: add Fabric support #456](https://github.com/react-native-picker/picker/pull/456/files) |

### 支持的类型

当使用 Flow 或 TypeScript 时，您需要使用 [类型注释](https://flow.org/en/docs/types/) 来定义 Spec 。请记住，定义 JavaScript Spec 是为了确保生产的原生接口代码是类型安全的，支持的类型集可以一对一映射到原生平台上相应的类型。

总的来说，这意味着您可以使用基本类型（strings，numbers，booleans）、函数类型、对象类型和数组类型。另一方面，联合类型（Union Types）不支持。

所有的类型必须为只读。对于 Flow ：`+` 或 `$ReadOnly<>` 或 `{||}`。对于 TypeScript：`readonly` 用于属性，`readonly <>` 用于对象，`ReadonlyArray<>` 用于数组。

### Codegen Helper Types

您可以在 JavaScript 规范中使用预定义的类型，下面是它们的列表：

- Double
- Float
- Int32
- WithDefault<Type, Value> - Sets default value for type
- BubblingEventHandler<T> - For bubbling events (eg: onChange).
- DirectEventHandler<T> - For direct events (eg: onClick).

## 原生平台代码

### Android & iOS

请参考官方的[新架构迁移指南](https://reactnative.cn/docs/new-architecture-intro)

### harmony

harmony 平台只支持新架构，所以并没有迁移的说法。

关于鸿蒙原生代码，请参考鸿蒙原生侧代码中的 [C++ NAPI 层](zh-cn/cpp.md) 和 [原生层](zh-cn/native.md) 章节。