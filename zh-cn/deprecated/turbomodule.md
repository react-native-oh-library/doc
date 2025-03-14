> [!WARNING] 本文档为非公开文档，仅用于三方库使用和开发指导，不涉及任何 React Native OpenHarmony 框架的信息，且会随着 React Native OpenHarmony 框架持续迭代更新，当前版本不代表最终展示版本。

# TurboModules

Turbo Modules 是升级版的 Native Modules，是基于 JSI 开发的一套 JS 与 Native 交互的轻量级框架。TurboModules 本质上的作用是导出一系列的 Native 方法供 JS 使用。

详细的原理分析可以看：[React Native 之新架构中的 Turbo Module 实现原理分析](https://cloud.tencent.com/developer/article/1889895)

![turbomodules](../img/turbomodule2.png)

## 如何创建 TurboModule

创建一个 Turbo Native Module 分为以下步骤：

1. 声明 JavaScript 接口类型；
2. 配置模块以支持 Codegen 自动生成脚手架代码；
3. 编写原生代码完成模块实现。

接下来会创建一个简单的名为 `RTNCalculator` 的 TurboModule 作为示例。

**该示例仅提供 HarmonyOS 版本，Android/iOS 版本请阅读 React-Native 官方文档的 [TurboModules 章节](https://reactnative.cn/docs/0.73/the-new-architecture/pillars-turbomodules)**

### 1. 目录配置

我们按照一般的三方库目录结构来配置:

```md
TurboModulesGuide
├── MyApp
└── RTNCalculator
    ├── android（Android 的原生实现代码）
    ├── ios（iOS 的原生实现代码）
    ├── harmony（HarmonyOS 的原生实现代码）
    └── src （js/ts代码）
```

### 2. 声明 JavaScript 接口

新架构要求必须使用强类型风格语言声明 JavaScript 接口（Flow 和 TypeScript 皆可）。Codegen 会根据这些接口声明来生成强类型的语言，其中包括 C++、Objective-C 和 Java。

对于声明类型的代码文件必须满足以下两点要求：

1. 文件必须使用 `Native<MODULE_NAME>`命名，在使用 Flow 时，以 .js 或 .jsx 为后缀名；在使用 **Typescript** 时，以 `.ts` 或 `.tsx` 为后缀名。**Codegen** 只会找到匹配这些命名规则的文件；
2. 代码中必须要输出 `TurboModuleRegistrySpec` 对象

<!-- tabs:start -->

#### **flow**

NativeCalculator.js

```js
// @flow
import type { TurboModule } from "react-native/Libraries/TurboModule/RCTExport";
import { TurboModuleRegistry } from "react-native";

export interface Spec extends TurboModule {
  add(a: number, b: number): Promise<number>;
}
export default (TurboModuleRegistry.get<Spec>("RTNCalculator"): ?Spec);
```

#### **typescript**

NativeCalculator.ts

```ts
import type { TurboModule } from "react-native/Libraries/TurboModule/RCTExport";
import { TurboModuleRegistry } from "react-native";

export interface Spec extends TurboModule {
  add(a: number, b: number): Promise<number>;
}

export default TurboModuleRegistry.get<Spec>("RTNCalculator") as Spec | null;
```

<!-- tabs:end -->

在代码顶部需导入以下两个声明文件：

- 类型 TurboModule ：定义 Turbo Native Module 的基础接口
- JS 模块 TurboModuleRegistry：包含了用于加载 Turbo Native Module 的函数

代码的第二个部分就是针对 Turbo Native Module 的接口声明。在本例中，接口声明了 `add` 函数，它将用于接受两个数字并返回一个包装数字的 Promise。为声明 Turbo Native Module，此接口**必须**命名为 Spec。

最后，调用 `TurboModuleRegistry.get` 并传入模块名，它将在 Turbo Native Module 可用的时候进行加载。

将 js 声明文件放入 `src` 文件夹下。

> [!TIP] 当我们在编写 JavaScript 代码时，如果没有配置好对应的模块或依赖安装，就从第三方库导入类型，可能会使的您的 IDE 不能正确载入导入声明，从而显示错误或警告。这种情况是正常的，它不会在您添加模块到 App 的时候出现问题。

### 3. Codegen 配置

接下来，需要为 Codegen 和自动链接添加一些配置。Codegen 的作用是生成 C++ 脚手架代码，负责串联 JS 和原生侧。

有一些配置文件在 Android/iOS/HarmonyOS 平台是通用的，而有的仅能在某一平台使用。

#### 3.1 配置 `package.json` 文件

请在 `RTNCalculator` 的根目录创建 `package.json` 文件。

```json
{
    "name": "rtn-calculator",
    "version": "0.0.1",
    "description": "Add numbers with TurboModules",
    "react-native": "src/index",
    "source": "src/index",
    "files": [
      "src",
      "harmony",
      "rtn-calculator.podspec",
      "!android/build",
      "!ios/build",
      "!**/__tests__",
      "!**/__fixtures__",
      "!**/__mocks__"
    ],
    "keywords": ["react-native", "harmony"],
    "repository": "https://github.com/<your_github_handle>/rtn-calculator",
    "author": "<Your Name> <your_email@your_provider.com> (https://github.com/<your_github_handle>)",
    "license": "MIT",
    "bugs": {
      "url": "https://github.com/<your_github_handle>/rtn-calculator/issues"
    },
    "homepage": "https://github.com/<your_github_handle>/rtn-calculator#readme",
    "devDependencies": {},
    "peerDependencies": {
      "react": "*",
      "react-native": "*"
    },
    "harmony": {
      "codegenConfig": {
        "specPaths": [
          "./src"
        ]
      }
    }
  }
  
```

将 HarmonyOS 配置到 harmony.codegenConfig 字段。

- specPaths：用于找到 js 接口声明文件的相对路径，它将被 Codegen 解析

#### 3.2 codegen通用配置项

HarmonyOS 需要在 RN 工程中通过运行脚本运行 Codegen。

打开 RN 工程下的 package.json，如 `MyApp/package.json`，添加：

```json
{
  ...
  "scripts": {
    ...
    "codegen": "react-native codegen-harmony --rnoh-module-path ./harmony/react_native_openharmony"
  },
  ...
}
```

> codegen-harmony 参数介绍：

1. --rnoh-module-path: 指定 rnoh OHOS 模块的相对路径，用于存储生成的 ts 文件；如果使用 har 包引入 rnoh 模块，则需要指向：./harmony/entry/oh_modules/@rnoh/react-native-openharmony"

2. --cpp-output-path: 指定用于存储生成的 C++ 文件的输出目录的相对路径，默认 ./harmony/entry/src/main/cpp/generated；

3. --project-root-path: 包根目录的相对路径。

### 4. 原生代码

HarmonyOS 平台上 Turbo Native Module 的原生代码需执行如下步骤：

1. 创建用于实现模块的 CalculatorModule.ts
2. 创建 CalculatorPackage.ts
3. 创建用于导出模块的 index.ets 和 ts.ts
4. 创建 oh-package.json5，hvigorfile.ts，module.json5

> [!TIP] 可以在 DevEco Studio 中通过 File -> New -> Module.. -> Static Lirbrary 创建空壳模块，以此为基础修改文件内容

HarmonyOS 第三方库原生代码文件结构应为如下：

```md
harmony
└── calculator
    ├── src
    │   └── main
    │       ├──ets
    │       │   ├── CalculatorModule.ts
    │       │   └── CalculatorPackage.ts
    │       └── modules.json5         
    ├── build-profile.json5
    ├── hvigorfile.ts
    ├── index.ets
    ├── oh-package.json5
    └── ts.ts
```

创建 `CalculatorModule.ts`

<!-- tabs:start -->

#### **CalculatorModule.ts**

```ts
import { TurboModule } from '@rnoh/react-native-openharmony/ts';
import { TM } from "@rnoh/react-native-openharmony/generated/ts"

export class CalculatorModule extends TurboModule implements TM.RTNCalculator.Spec {
    add(a: number, b: number): Promise<number> {
        return new Promise((resolve) => resolve(a + b));
      }
}
```

<!-- tabs:end -->

这个类实现了模块的功能，它继承了 TurboModule 类，对应 Android 里的 NativeCalculatorSpec。

创建用于实现模块的 `CalculatorModule.ts`

 <!-- tabs:start -->

#### **CalculatorPackage.ts**

```ts
import {
  RNPackage,
  TurboModulesFactory,
} from "@rnoh/react-native-openharmony/ts";
import type {
  TurboModule,
  TurboModuleContext,
} from "@rnoh/react-native-openharmony/ts";
import { TM } from "@rnoh/react-native-openharmony/generated/ts";
import { CalculatorModule } from './CalculatorModule';

class CalculatorModulesFactory extends TurboModulesFactory {
  createTurboModule(name: string): TurboModule | null {
    if (name === TM.RTNCalculator.NAME) {
      return new CalculatorModule(this.ctx);
    }
    return null;
  }

  hasTurboModule(name: string): boolean {
    return name === TM.RTNCalculator.NAME;
  }
}

export class CalculatorPackage extends RNPackage {
  createTurboModulesFactory(ctx: TurboModuleContext): TurboModulesFactory {
    return new CalculatorModulesFactory(ctx);
  }
}

```

<!-- tabs:end -->

这就是 HarmonyOS 平台原生代码的最后一部分，它定义了 RNPackage 对象，这个对象将用于 App 的模块加载。

创建 `ts.ts` 和 `index.ets`

 <!-- tabs:start -->

#### **ts.ts**

```ts
export * from "./src/main/ets/CalculatorPackage";
export * from "./src/main/ets/CalculatorModule";
```

<!-- tabs:end -->

 <!-- tabs:start -->

#### **index.ets**

```ts
export * from "./ts";
```

<!-- tabs:end -->

修改 `oh-package.json5`，`hvigorfile.ts`，`module.json5`，或自行创建

 <!-- tabs:start -->

#### **oh-package.json5**

```json
{
  "license": "ISC",
  "types": "",
  "devDependencies": {
  },
  "name": "rtn-calculator",
  "description": "",
  "main": "index.ets",
  "version": "0.0.1",
  "dependencies": {
    "@rnoh/react-native-openharmony": "file:../react_native_openharmony"
  }
}

```

<!-- tabs:end -->

 <!-- tabs:start -->

#### **hvigorfile.ts**

```ts
export { harTasks } from "@ohos/hvigor-ohos-plugin";
```

<!-- tabs:end -->

 <!-- tabs:start -->

#### **module.json5**

```json
{
  module: {
    name: 'calculator',
    type: 'har',
    deviceTypes: ['default'],
  },
}
```

<!-- tabs:end -->

### 5. 将 Turbo Native Module 添加到 App

#### 5.1 配置 RN 工程，执行 codegen

首先，需要将包含模块的 NPM 包添加到 App。请确保 package.json 已经配置安装好以下依赖：

```json
{
  ...
  "dependencies": {
    "react-native-harmony": "x.x.x",
    ...
  },
  "overrides": {
    "@react-native/codegen": "0.74.0"
  },
  ...
}
```

执行以下操作，假设 MyApp 为您的 App 工程路径

```bash
// 进入模块工程
cd RTNCalculator

// 打包模块
npm pack

// 进入 App 工程
cd ../MyApp

// 本地路径安装模块
npm i file:../RTNCalculator/rtn-calculator-0.0.1.tgz

// 执行以下命令执行 codegen (HarmonyOS only)

npm run codegen

```

此命令会将 RTNCalculator 模块添加到 App 内的 node_modules 目录。

#### 5.2 原生工程配置项

> [!tip] 待完善能力：HarmonyOS 平台目前暂时不支持 AutoLink，所以需要自行配置。

首先使用 DevEco Studio 打开 React-Native 项目里的 HarmonyOS 工程 `harmony`

目前 HarmonyOS 工程暂不支持引入工程外的模块，所以需要手动将模块的 HarmonyOS 源码复制到工程内。

- 复制 `RTNCalculator/harmony/calculator` 到 `harmony` 工程根目录下。

- 修改 `MyApp/harmony/build-profile.json5`，在 modules 字段添加：

```json
{
...
  modules: [
    ...
    {
      name: 'calculator',
      srcPath: './calculator',
    }
  ]
}
```

- 在工程根目录的 `MyApp/harmony/oh-package.json5` 添加 overrides 字段

```json
{
  ...
  "overrides": {
    "@rnoh/react-native-openharmony" : "./react_native_openharmony.har" // RNOH SDK har包路径或源码路径
  }
}
```

- 打开 `MyApp/harmony/entry/oh-package.json5`，添加以下依赖，引入鸿蒙原生端的代码

```json
"dependencies": {
  "@rnoh/react-native-openharmony": "file:../react_native_openharmony.har",  // RNOH SDK har包路径或源码路径
  "rtn-calculator": "file:../calculator"
  }
```

- 点击右上角的 `sync` 按钮同步工程，或在终端运行以下命令

```bash
cd entry
ohpm install
```

#### 5.3 在 ArkTs 侧引入 Calculator TurboModule

打开 `MyApp/harmony/entry/src/main/ets/RNPackageFactory.ts`，添加：

```diff
import type {RNPackageContext, RNPackage} from '@rnoh/react-native-openharmony/ts';
import {SamplePackage} from 'rnoh-sample-package/ts';
+ import { CalculatorPackage } from "rtn-calculator/ts";

export function createRNPackages(ctx: RNPackageContext): RNPackage[] {
  return [
    new SamplePackage(ctx),
+   new CalculatorPackage(ctx),
    ];
}
```

编译、运行即可。

#### 5.4 JavaScript

以下是一个在 App.js 中调用 add 方法的例子：

<!-- tabs:start -->

#### **App.js**

```js
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 *
 * @format
 * @flow strict-local
 */
import React from "react";
import { useState } from "react";
import type { Node } from "react";
import { SafeAreaView, StatusBar, Text, Button } from "react-native";
import RTNCalculator from "rtn-calculator/src/NativeCalculator.js";

const App: () => Node = () => {
  const [result, setResult] = useState<number | null>(null);
  return (
    <SafeAreaView>
      <StatusBar barStyle={"dark-content"} />
      <Text style={{ marginLeft: 20, marginTop: 20 }}>
        3+7={result ?? "??"}
      </Text>
      <Button
        title="Compute"
        onPress={async () => {
          const value = await RTNCalculator.add(3, 7);
          setResult(value);
        }}
      />
    </SafeAreaView>
  );
};
export default App;
```

<!-- tabs:end -->

现在，您可以运行 App 并查看在屏幕上显示的组件。

> [!TIP] 可通过 npm run start 使用热更新
