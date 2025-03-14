> [!WARNING] 本文档为非公开文档，仅用于三方库使用和开发指导，不涉及任何 React Native OpenHarmony 框架的信息，且会随着 React Native OpenHarmony 框架持续迭代更新，当前版本不代表最终展示版本。

# TurboModules

如果您使用过 React Native，您可能了解过 [Native Modules](https://reactnative.cn/docs/0.75/native-modules-intro) 这个概念。它可以通过 React Native 的「Bridge」帮助 JavaScript 和原生代码进行交互，并使用跨平台的数据格式 JSON 进行通讯

Turbo Modules 可以理解为升级版的 Native Modules，是基于 JSI 开发的一套 JS 与 Native 交互的轻量级框架。TurboModules 本质上的作用是导出一系列的 Native 方法供 JS 使用。

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
├── exampleApp
└── RTNCalculator
    ├── harmony（HarmonyOS 的原生实现代码）
    └── src （js/ts代码）
```

### 2. 声明 JavaScript 接口

新架构要求必须使用强类型风格语言声明 JavaScript 接口（Flow 和 TypeScript 皆可）。Codegen 会根据这些接口声明来生成强类型的语言，其中包括 C++、Objective-C 和 Java。

对于声明类型的代码文件必须满足以下两点要求：

1. 文件必须使用 `Native<MODULE_NAME>`命名，在使用 Flow 时，以 .js 或 .jsx 为后缀名；在使用 **Typescript** 时，以 `.ts` 或 `.tsx` 为后缀名。**Codegen** 只会找到匹配这些命名规则的文件；
2. 代码中必须要输出 `TurboModuleRegistrySpec` 对象

<!-- tabs:start -->

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
    }
  }
```

> [!TIP] 可以通过添加 harmony/alias 字段为第三方库指定别名，通常在移植某个第三方库时使用，以确保开发者在使用时仍能保持原库名不变。

#### 3.2 使用 codegen-lib-harmony 工具

此步操作是要使用 Codegen 来为组件生成脚手架代码。在 `react-native-harmony-cli >= 0.0.27` 的版本中提供了 codegen-lib-harmony 工具，开发者可以使用该脚本随时执行 Codegen，可以选择运行时生成，也可以选择将这部分生成的脚手架代码内置到模块内。以下将介绍内置到模块内的用法。

在 `RTNCenteredText/package.json` 中新增 `react-native-harmony-cli` 开发依赖：

```json
"devDependencies": {
  "react-native-harmony-cli": "npm:@react-native-oh/react-native-harmony-cli@^0.0.27",
  "@react-native-community/cli": "11.4.1"
},
```

在 `scripts` 字段配置 codegen-lib 脚本：

```json
"scripts": {
    "codegen-lib": "react-native codegen-lib-harmony --no-safety-check --npm-package-name rtn-calculator --cpp-output-path ./harmony/rtn_calculator/src/main/cpp/generated --ets-output-path ./harmony/rtn_calculator/src/main/ets/generated --turbo-modules-spec-paths ./src"
  }
```

该脚本根据指定的 spec 文件，将脚手架代码生成到指定目录。

1. --npm-package-name: npm 包名（package.json 的 name 字段）
2. --cpp-output-path: 指定用于存储生成的 C++ 文件的输出目录的相对路径
3. --ets-output-path: 指定用于存储生成的 ArkTS 文件的输出目录的相对路径
4. --turbo-modules-spec-paths: TurboModule 声明文件路径或文件夹路径

执行 codegen-lib 脚本后，可以在 `RTNCalculator/harmony/rtn_calculator/src/main` 文件夹中找到 Codegen 代码

```md
.
├── cpp
│   └── generated
│       ├── RNOH
│       │   └── generated
│       │       ├── BaseRtnCalculatorPackage.h
│       │       └── turbo_modules
│       │           ├── RTNCalculator.cpp
│       │           └── RTNCalculator.h
│       └── react
│           └── renderer
│               └── components
│                   └── rtn_calculator
│                       ├── ComponentDescriptors.h
│                       ├── EventEmitters.cpp
│                       ├── EventEmitters.h
│                       ├── Props.cpp
│                       ├── Props.h
│                       ├── ShadowNodes.cpp
│                       ├── ShadowNodes.h
│                       ├── States.cpp
│                       └── States.h
└── ets
    └── generated
        ├── components
        │   └── ts.ts
        ├── index.ets
        ├── ts.ts
        └── turboModules
            ├── RTNCalculator.ts
            └── ts.ts
```

生成的代码分为两个文件夹：

- `cpp` 包含 C++ 代码，以让 JS 和 C++/ArkTS 正确交互 
- `ets` 包含平台特定的代码

在 `cpp/generated` 文件夹中，是所有连接 JS 和 HarmonyOS 的样板代码。

- `cpp/generated/RNOH/generated` 下生成的是自定义TurboModule对接 RNOH 框架所需的文件；
- `cpp/generated/react/renderer/components/rtn_centered_text` 下包含自定义组件所需的粘合代码，因为该示例只有 TurboModule 没有 Fabric 组件，所以可以发现该路径下生成的文件均是空实现。

在 `ets/generated` 文件夹中，`turboModules` 下会生成类型定义文件，用于定义 RTNCalculator 这个 TurboModule 的接口规范。

### 4. 原生实现

HarmonyOS 平台上 Turbo Native Module 的原生代码需执行如下步骤：

1. 创建用于实现模块的 CalculatorModule.ts
2. 创建 CalculatorPackage.ts
3. 创建用于导出模块的 Index.ets 和 ts.ts

> [!TIP] 目前 RNOH 已支持使用 ets 实现 TurboModule，所以可以将 `CalculatorModule.ts` 和 `CalculatorPackage.ts` 的后缀改为 `.ets`，同时需要把 `ts.ts` 文件删除并把内容复制到 `Index.ets` 中导出。

> [!TIP] 可以在 DevEco Studio 中通过 File -> New -> Module.. -> Static Lirbrary 创建空壳模块，以此为基础修改文件内容

HarmonyOS 第三方库原生代码文件结构应为如下：

```md
harmony
└── rtn_calculator
    ├── Index.ets
    ├── build-profile.json5
    ├── hvigorfile.ts
    ├── oh-package.json5
    ├── src
    │   └── main
    │       ├── cpp
    │       │   ├── CMakeLists.txt
    │       │   ├── RTNCalculatorPackage.h
    │       │   └── generated
    │       ├── ets
    │       │   ├── CalculatorModule.ts
    │       │   ├── CalculatorPackage.ts
    │       │   └── generated
    │       └── module.json5
    └── ts.ts
```

创建 `CalculatorModule.ts`

<!-- tabs:start -->

#### **CalculatorModule.ts**

```ts
import { TurboModule } from '@rnoh/react-native-openharmony/ts';
import { TM } from "./generated/ts"

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
import { RNPackage, TurboModulesFactory, } from "@rnoh/react-native-openharmony/ts";
import type { TurboModule, TurboModuleContext, } from "@rnoh/react-native-openharmony/ts";
import { TM } from "./generated/ts";
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

#### **Index.ets**

```ts
export * from "./ts";
```

<!-- tabs:end -->

创建 `cpp/RTNCalculatorPackage.h`

<!-- tabs:start -->

#### **RTNCalculatorPackage.h**

```cpp
#pragma once

#include "RNOH/generated/BaseRtnCalculatorPackage.h"

namespace rnoh {

class RTNCalculatorPackage : public BaseRtnCalculatorPackage {
    using Super = BaseRtnCalculatorPackage;

public:
    RTNCalculatorPackage(Package::Context ctx) : Super(ctx) {}
};

} // namespace rnoh
```

<!-- tabs:end -->


创建 `cpp/CMakeLists.txt`

<!-- tabs:start -->

#### **CMakeLists.txt**

```txt
# 设置 Codegen 生成目录，指定 generated 目录路径
set(rtn_centered_text_generated_dir "${CMAKE_CURRENT_SOURCE_DIR}/generated")

# 使用 GLOB_RECURSE 递归地查找所有在 generated 目录下的 .cpp 文件，并将其存储到变量 rtn_calculator_generated_SRC 中
file(GLOB_RECURSE rtn_calculator_generated_SRC "${rtn_calculator_generated_dir}/**/*.cpp")

# 查找当前目录下的所有 .cpp 文件，并将其存储到变量 rtn_calculator_SRC 中
# CONFIGURE_DEPENDS 表示如果这些文件被修改，CMake 会重新配置
file(GLOB rtn_calculator_SRC CONFIGURE_DEPENDS *.cpp)

# 创建一个共享库 rtn_centered_text，包含两部分：rtn_centered_text_SRC 和 rtn_centered_text_generated_SRC
add_library(rtn_calculator SHARED ${rtn_calculator_SRC} ${rtn_calculator_generated_SRC})

# 为目标库 rtn_centered_text 设置包含路径，这些路径会包含当前源目录和 Codegen 生成文件所在的目录
target_include_directories(rtn_calculator PUBLIC ${CMAKE_CURRENT_SOURCE_DIR} ${rtn_calculator_generated_dir})

# 将库 rtn_calculator 链接到 rnoh sdk，意味着 rtn_calculator 使用 rnoh sdk 中的功能
target_link_libraries(rtn_calculator PUBLIC rnoh)
```

<!-- tabs:end -->

创建 `oh-package.json5`，`hvigorfile.ts`，`module.json5`，`build-profile.json5`

<!-- tabs:start -->

#### **oh-package.json5**

```json
{
  license: 'MIT',
  types: '',
  name: 'rtn-calculator',
  description: '',
  main: 'Index.ets',
  type: 'module',
  version: '0.0.1',
  dependencies: {
    "@rnoh/react-native-openharmony": '0.72.38'
  },
}
```

<!-- tabs:end -->

<!-- tabs:start -->

#### **hvigorfile.ts**

```ts
// Script for compiling build behavior. It is built in the build plug-in and cannot be modified currently.
export { harTasks } from '@ohos/hvigor-ohos-plugin';
```

<!-- tabs:end -->

<!-- tabs:start -->

#### **src/main/module.json5**

```json
{
  module: {
    name: 'rtn_calculator',
    type: 'har',
    deviceTypes: ['default'],
  },
}
```

<!-- tabs:end -->

<!-- tabs:start -->

#### **build-profile.json5**

```json
{
  "apiType": "stageMode",
  "targets": [
    {
      "name": "default",
    }
  ]
}
```

<!-- tabs:end -->

### 5. 将 Turbo Native Module 添加到 App

#### 5.1 配置 RN 工程，进行打包安装

执行以下操作，假设 exampleApp 为您的 App 工程

```bash
// 进入三方库工程
cd RTNCalculator

// 打包三方库，rtn-calculator-0.0.1.tgz
npm pack

// 进入 App 工程
cd ../exampleApp

// 本地路径安装三方库，若需要更新版本，请先执行 npm uninstall rtn-calculator
npm i file:../RTNCalculator/rtn-calculator-0.0.1.tgz

```

此命令会将 RTNCalculator 模块安装到 exampleApp 工程， 您可以在 `node_modules/rtn-calculator` 目录找到相关文件。

#### 5.2 原生工程配置项

首先使用 DevEco Studio 打开 exampleApp 项目里的鸿蒙工程 `harmony`

- 修改 `exampleApp/harmony/build-profile.json5`，在 modules 字段添加：

```json
{
...
  modules: [
    ...
    {
      name: 'rtn_calculator',
      srcPath: '../../RTNCalculator/harmony/rtn_calculator'
    },
  ]
}
```
指向 rtn_calculator 模块所在路径，使得 rtn_calculator 能够被识别为工程 module。

- 在工程根目录的 `exampleApp/harmony/oh-package.json5` 添加 RN SDK 依赖和 overrides 字段：

> [!TIP] overrides 字段是为了让工程依赖同一个版本的 RN SDK，最终的依赖版本以 overrides 字段指定的为准。

```json
{
  ...
  "dependencies": {
    "@rnoh/react-native-openharmony": "0.72.38"
  },
  "overrides": {
    "@rnoh/react-native-openharmony": "0.72.38"  // 请按需选择版本
  }
}
```

- 打开 `exampleApp/harmony/entry/oh-package.json5`，添加以下依赖，引入 RTNCalculator 的原生代码：

```json
"dependencies": {
    "rtn-calculator": "../../../RTNCalculator/harmony/rtn_calculator",  // rtn_calculator 模块所在路径
  }
```

- 点击右上角的 `sync` 按钮同步工程，或在终端运行以下命令

```bash
cd entry
ohpm install
```

#### 5.3 链接 CPP

打开 `exampleApp/harmony/entry/src/main/cpp/CMakeLists.txt`，添加：

```diff
# ...
+ set(OH_MODULES_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../oh_modules")

# RNOH_BEGIN: manual_package_linking_1
+ add_subdirectory("${OH_MODULES_DIR}/rtn-calculator/src/main/cpp" ./rtn-calculator)
# RNOH_END: manual_package_linking_1

# RNOH_BEGIN: manual_package_linking_2
+ target_link_libraries(rnoh_app PUBLIC rtn_calculator)
# RNOH_END: manual_package_linking_2
```

打开 `exampleApp/harmony/entry/src/main/cpp/PackageProvider.cpp`，添加：

```diff
+ #include "RTNCalculatorPackage.h"

using namespace rnoh;

std::vector<std::shared_ptr<Package>> PackageProvider::getPackages(Package::Context ctx) {
    return {
+     std::make_shared<RTNCalculatorPackage>(ctx),
    };
}
```

#### 5.4 在 ArkTs 侧引入 Calculator TurboModule

打开 `MyApp/harmony/entry/src/main/ets/RNPackageFactory.ets`，添加：

```diff
import type { RNPackageContext, RNPackage } from '@rnoh/react-native-openharmony';
+ import { CalculatorPackage } from "rtn-calculator/ts";

export function createRNPackages(ctx: RNPackageContext): RNPackage[] {
  return [
+   new CalculatorPackage(ctx),
  ];
}
```

编译、运行即可。

### 6. JavaScript

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

> [!TIP] 可通过 `hdc rport tcp:8081 tcp:8081 && npm run start` 使用热更新

### 7. 其他

#### 7.1 如何将 HarmonyOS 原生部分的代码打成 *.har 包

当完成上述操作，调试验证完毕您所需要的功能开发后，即可进行模块 har 打包操作

1. 打开 DevEco Studio，选中 `rtn_calculator` 模块，点击 Build -> Make Module 'rtn_calculator'；

2. .har 会生成到目录 rtn_calculator/build/default/outputs/default (模块 har 包生成的默认路径)
