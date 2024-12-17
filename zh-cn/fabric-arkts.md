> [!WARNING] 本文档仅用于三方库使用和开发指导，不涉及任何 React Native OpenHarmony 框架的信息，且会随着 React Native OpenHarmony 框架持续迭代更新，当前版本不代表最终展示版本。

# 使用 ArkTS API 开发 Fabric 组件

此文档主要介绍在原生侧使用 ArkTS API 开发 Fabric 组件，React 侧请阅读入口文档 [Fabric 组件](/zh-cn/fabric.md)

### 1. Codegen 配置

#### 1.1 配置 `package.json` 文件

请在 `RTNCenteredText` 的根目录创建 `package.json` 文件。

```json
{
  "name": "rtn-centered-text",
  "version": "0.0.1",
  "description": "Showcase a Fabric component with a centered text",
  "react-native": "src/index",
  "source": "src/index",
  "files": [
    "src",
    "harmony",
    "!**/__tests__",
    "!**/__fixtures__",
    "!**/__mocks__"
  ],
  "keywords": ["react-native", "harmony"],
  "repository": "https://github.com/<your_github_handle>/rtn-centered-text",
  "author": "<Your Name> <your_email@your_provider.com> (https://github.com/<your_github_handle>)",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/<your_github_handle>/rtn-centered-text/issues"
  },
  "homepage": "https://github.com/<your_github_handle>/rtn-centered-text#readme",
  "devDependencies": {},
  "peerDependencies": {
    "react": "*",
    "react-native": "*"
  }
}
```

> [!TIP] 可以通过添加 harmony/alias 字段为第三方库指定别名，通常在移植某个第三方库时使用，以确保开发者在使用时仍能保持原库名不变。

#### 1.2 使用 codegen-lib-harmony 工具

此步操作是要使用 Codegen 来为组件生成脚手架代码。在 `react-native-harmony-cli >= 0.0.27` 的版本中提供了 codegen-lib-harmony 工具，开发者可以使用该脚本随时执行 Codegen，可以选择运行时生成，也可以选择将这部分生成的脚手架代码内置到模块内。以下将介绍内置到模块内的用法。

在 `RTNCenteredText/package.json` 中新增 `react-native-harmony-cli` 开发依赖：

```json
"devDependencies": {
  "react-native-harmony-cli": "npm:@react-native-oh/react-native-harmony-cli@^0.0.27",
  "@react-native-community/cli": "11.3.6"
},
```

在 `scripts` 字段配置 codegen-lib 脚本：

```json
"scripts": {
    "codegen-lib": "react-native codegen-lib-harmony --no-safety-check --npm-package-name rtn-centered-text --cpp-output-path ./harmony/rtn_centered_text/src/main/cpp/generated --ets-output-path ./harmony/rtn_centered_text/src/main/ets/generated --arkts-components-spec-paths ./src"
  }
```

该脚本根据指定的 spec 文件，将脚手架代码生成到指定目录。

> codegen-lib-harmony 参数介绍：

1. --npm-package-name: npm 包名（package.json 的 name 字段）
2. --cpp-output-path: 指定用于存储生成的 C++ 文件的输出目录的相对路径
3. --ets-output-path: 指定用于存储生成的 ArkTS 文件的输出目录的相对路径
4. --arkts-components-spec-paths: Fabric 声明文件路径或文件夹路径（ArkTS API 开发）

执行 codegen-lib 脚本后，可以在 `RTNCenteredText/harmony/rtn_centered_text/src/main` 文件夹中找到 Codegen 代码

```md
.
├── cpp
│   └── generated
│       ├── RNOH
│       │   └── generated
│       │       ├── BaseRtnCenteredTextPackage.h
│       │       └── components
│       │           └── RTNCenteredTextJSIBinder.h
│       └── react
│           └── renderer
│               └── components
│                   └── rtn_centered_text
│                       ├── ComponentDescriptors.h
│                       ├── EventEmitters.cpp
│                       ├── EventEmitters.h
│                       ├── Props.cpp
│                       ├── Props.h
│                       ├── ShadowNodes.cpp
│                       ├── ShadowNodes.h
│                       ├── States.cpp
│                       └── States.h
├── ets
│   └── generated
│       ├── components
│       │   ├── RTNCenteredText.ts
│       │   └── ts.ts
│       ├── index.ets
│       ├── ts.ts
│       └── turboModules
│           └── ts.ts
```

生成的代码分为两个文件夹：

- `cpp` 包含 C++ 代码，以让 JS 和 C++/ArkTS 正确交互 
- `ets` 包含平台特定的代码

在 `cpp/generated` 文件夹中，是所有连接 JS 和 HarmonyOS 的样板代码。

- `cpp/generated/RNOH/generated` 下生成的是自定义组件对接 RNOH 框架所需的文件；
- `cpp/generated/react/renderer/components/rtn_centered_text` 下包含自定义组件所需的粘合代码。

在 `ets/generated` 文件夹中，是该自定义组件的 ArkTS 类型声明，包括了改组件所拥有的属性、事件、命令等接口的定义。

### 2. 使用 ArkTS API 实现原生组件

HarmonyOS 平台中 ArkTS 版本的 Fabric 组件的原生代码必须包含以下三个部分：

1. 创建用于实现组件的 `RTNCenteredText.ets`
2. 创建 `RTNCenteredTextPackage.ts` 和 `RTNCenteredTextPackage.h`
3. 创建用于导出模块的 `Index.ets` 和 `ts.ts`

HarmonyOS 第三方库目录文件结构应为如下：

```md
harmony
└── rtn_centered_text
    ├── Index.ets
    ├── build-profile.json5
    ├── hvigorfile.ts
    ├── oh-package.json5
    ├── src
    │   └── main
    │       ├── cpp
    │       │   ├── CMakeLists.txt
    │       │   ├── RTNCenteredTextPackage.h
    │       │   └── generated
    │       ├── ets
    │       │   ├── RTNCenteredText.ets
    │       │   ├── RTNCenteredTextPackage.ts
    │       │   └── generated
    │       └── module.json5
    └── ts.ts
```

创建 `ets/RTNCenteredText.ets`，`ets/RTNCenteredPackage.ts`

<!-- tabs:start -->

#### **RTNCenteredText.ets**

```ts
import { RNComponentContext, RNViewBase } from '@rnoh/react-native-openharmony';
import { RNC } from "./generated"

@Component
export struct RTNCenteredText {
  public static readonly NAME = RNC.RTNCenteredText.NAME
  public ctx!: RNComponentContext
  public tag: number = 0
  @State private descriptorWrapper: RNC.RTNCenteredText.DescriptorWrapper = {} as RNC.RTNCenteredText.DescriptorWrapper
  private eventEmitter: RNC.RTNCenteredText.EventEmitter | undefined = undefined
  private cleanUpCallbacks: (() => void)[] = []

  aboutToAppear() {
    this.eventEmitter = new RNC.RTNCenteredText.EventEmitter(this.ctx.rnInstance, this.tag)
    this.onDescriptorWrapperChange(this.ctx.descriptorRegistry.findDescriptorWrapperByTag<RNC.RTNCenteredText.DescriptorWrapper>(this.tag)!)
    this.cleanUpCallbacks.push(this.ctx.descriptorRegistry.subscribeToDescriptorChanges(this.tag,
      (_descriptor, newDescriptorWrapper) => {
        this.onDescriptorWrapperChange(newDescriptorWrapper! as RNC.RTNCenteredText.DescriptorWrapper)
      }
    ))
  }

  private onDescriptorWrapperChange(descriptorWrapper: RNC.RTNCenteredText.DescriptorWrapper) {
    this.descriptorWrapper = descriptorWrapper
  }

  aboutToDisappear() {
    this.cleanUpCallbacks.forEach(cb => cb())
  }

  build() {
    RNViewBase({ ctx: this.ctx, tag: this.tag }) {
      Text(this.descriptorWrapper.props.text)
        .height("100%")
        .width("100%")
        .fontSize(30)
        .textAlign(TextAlign.Center)
        .fontColor(this.descriptorWrapper.props.color?.toRGBAString())
        .onTouch((e) => {
          this.eventEmitter?.emit("textTouch", { type: e.type })
        })
    }
  }
}
```

<!-- tabs:end -->

这里定义了原生视图 RTNCenteredText，主要功能是显示一个居中的文本，并响应触摸事件。组件通过 @State 和 descriptorWrapper 管理其状态，并通过 eventEmitter 发射事件。在生命周期方法 aboutToAppear 和 aboutToDisappear 中处理组件的初始化和清理操作。UI 渲染中通过 Text 控件展示内容，并支持根据描述符动态更新属性。

 <!-- tabs:start -->

#### **RTNCenteredPackage.ts**

```ts
import { RNPackage } from "@rnoh/react-native-openharmony/ts";
import type {
  DescriptorWrapperFactoryByDescriptorTypeCtx,
  DescriptorWrapperFactoryByDescriptorType,
} from "@rnoh/react-native-openharmony/ts";
// import codegen 生成的内容
import { RNC } from "./generated/ts";

export class RTNCenteredTextPackage extends RNPackage {
  createDescriptorWrapperFactoryByDescriptorType(
    ctx: DescriptorWrapperFactoryByDescriptorTypeCtx
  ): DescriptorWrapperFactoryByDescriptorType {
    return {
      [RNC.RTNCenteredText.NAME]: (ctx) =>
      new RNC.RTNCenteredText.DescriptorWrapper(ctx.descriptor),
    };
  }
}
```

<!-- tabs:end -->

创建 `ts.ts`

<!-- tabs:start -->

#### **ts.ts**

```ts
export * from "./src/main/ets/RTNCenteredTextPackage";
```

<!-- tabs:end -->

创建 `Index.ets`

<!-- tabs:start -->

#### **Index.ets**

```ts
export * from "./ts";
export * from "./src/main/ets/RTNCenteredText";
```

<!-- tabs:end -->

创建 `cpp/RTNCenteredTextPackage.h`

<!-- tabs:start -->

#### **RTNCenteredTextPackage.h**

```cpp
#pragma once

#include "RNOH/generated/BaseRtnCenteredTextPackage.h"

namespace rnoh {

class RTNCenteredTextPackage : public BaseRtnCenteredTextPackage {
    using Super = BaseRtnCenteredTextPackage;

public:
    RTNCenteredTextPackage(Package::Context ctx) : Super(ctx) {}
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

# 使用 GLOB_RECURSE 递归地查找所有在 generated 目录下的 .cpp 文件，并将其存储到变量 rtn_centered_text_generated_SRC 中
file(GLOB_RECURSE rtn_centered_text_generated_SRC "${rtn_centered_text_generated_dir}/**/*.cpp")

# 查找当前目录下的所有 .cpp 文件，并将其存储到变量 rtn_centered_text_SRC 中
# CONFIGURE_DEPENDS 表示如果这些文件被修改，CMake 会重新配置
file(GLOB rtn_centered_text_SRC CONFIGURE_DEPENDS *.cpp)

# 创建一个共享库 rtn_centered_text，包含两部分：rtn_centered_text_SRC 和 rtn_centered_text_generated_SRC
add_library(rtn_centered_text SHARED ${rtn_centered_text_SRC} ${rtn_centered_text_generated_SRC})

# 为目标库 rtn_centered_text 设置包含路径，这些路径会包含当前源目录和 Codegen 生成文件所在的目录
target_include_directories(rtn_centered_text PUBLIC ${CMAKE_CURRENT_SOURCE_DIR} ${rtn_centered_text_generated_dir})

# 将库 rtn_centered_text 链接到 rnoh sdk，意味着 rtn_centered_text 使用 rnoh sdk 中的功能
target_link_libraries(rtn_centered_text PUBLIC rnoh)
```

<!-- tabs:end -->

创建 `oh-package.json5`，`hvigorfile.ts`，`module.json5`，`build-profile.json5`

<!-- tabs:start -->

#### **oh-package.json5**

```json
{
  license: 'MIT',
  types: '',
  name: 'rtn-centered-text',
  description: '',
  main: 'Index.ets',
  type: 'module',
  version: '1.0.0',
  dependencies: {
    "@rnoh/react-native-openharmony": '0.72.38' // 执行指定依赖的 RN SDK 版本
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
    name: 'rtn_centered_text',
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

### 3. 将 Fabric 组件添加到 App

#### 3.1 配置 RN 工程，进行打包安装

执行以下操作，假设 exampleApp 为您的 App 工程

```bash
// 进入三方库工程
cd RTNCenteredText

// 打包三方库，rtn-centered-text-0.0.1.tgz
npm pack

// 进入 App 工程
cd ../exampleApp

// 本地路径安装三方库，若需要更新版本，请先执行 npm uninstall rtn-centered-text
npm i file:../RTNCenteredText/rtn-centered-text-0.0.1.tgz

```

此命令会将 RTNCenteredText 模块安装到 exampleApp 工程， 您可以在 `node_modules/rtn-centered-text` 目录找到相关文件。

#### 3.2 原生工程配置项

> [!tip] 待完善能力：HarmonyOS 平台目前暂时不支持 AutoLink，所以需要自行配置。

首先使用 DevEco Studio 打开 exampleApp 项目里的鸿蒙工程 `harmony`

- 修改 `exampleApp/harmony/build-profile.json5`，在 modules 字段添加：

```json
{
...
  modules: [
    ...
    {
      name: 'rtn_centered_text',
      srcPath: '../../RTNCenteredText/harmony/rtn_centered_text',
    },
  ]
}
```
指向 rtn_centered_text 模块所在路径，使得 rtn_centered_text 能够被识别为工程 module。

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

- 打开 `exampleApp/harmony/entry/oh-package.json5`，添加以下依赖，引入 RTNCenteredText 的原生代码：

```json
"dependencies": {
    "rtn-centered-text": "../../../RTNCenteredText/harmony/rtn_centered_text"  // rtn_centered_text 模块所在路径
  }
```

- 点击右上角的 `sync` 按钮同步工程，或在终端运行以下命令

```bash
cd entry
ohpm install
```

##### 3.2.1 链接 CPP

打开 `exampleApp/harmony/entry/src/main/cpp/CMakeLists.txt`，添加：

```diff
# ...
+ set(OH_MODULES_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../oh_modules")

# RNOH_BEGIN: manual_package_linking_1
+ add_subdirectory("${OH_MODULES_DIR}/rtn-centered-text/src/main/cpp" ./centered-text)
# RNOH_END: manual_package_linking_1

# RNOH_BEGIN: manual_package_linking_2
+ target_link_libraries(rnoh_app PUBLIC rtn_centered_text)
# RNOH_END: manual_package_linking_2
```

打开 `exampleApp/harmony/entry/src/main/cpp/PackageProvider.cpp`，添加：

```diff
+ #include "RTNCenteredTextPackage.h"

using namespace rnoh;

std::vector<std::shared_ptr<Package>> PackageProvider::getPackages(Package::Context ctx) {
    return {
+     std::make_shared<RTNCenteredTextPackage>(ctx),
    };
}
```

##### 3.2.2 ArkTS Fabric 组件特有配置项

打开 `exampleApp/harmony/entry/src/main/ets/pages/Index.ets`，添加：

```diff
+ import { RTNCenteredText } from "rtn-centered-text"

+ const arkTsComponentNames: Array<string> = [RTNCenteredText.NAME];

@Builder
export function buildCustomRNComponent(ctx: ComponentBuilderContext) {
+   if (ctx.componentName === RTNCenteredText.NAME) {
+     RTNCenteredText({
+     tag: ctx.tag,
+     ctx: ctx.rnComponentContext,
+   })
+ }
}
```

打开 `exampleApp/harmony/entry/src/main/ets/RNPackageFactory.ts`，添加：

```diff
import type {RNPackageContext, RNPackage} from '@rnoh/react-native-openharmony/ts';
+ import { RTNCenteredTextPackage } from "rtn-centered-text/ts";

export function createRNPackages(ctx: RNPackageContext): RNPackage[] {
  return [
+   new RTNCenteredTextPackage(ctx),
    ];
}
```

编译、运行即可。

#### 3.3 JavaScript

最后，操作以下步骤，您就可以在 JavaScript 调用组件了。

1. 在 js 文件中导入组件。假设要在 App.js 进行导入，需要添加这行代码：

```js
import RTNCenteredText from "rtn-centered-text/src/RTNCenteredTextNativeComponent";
```

2. 接下来，在 React Native 组件里进行调用。调用的语法和其它组件相同：

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
import type { Node } from "react";
import { SafeAreaView } from "react-native";
import RTNCenteredText from "rtn-centered-text/src/RTNCenteredTextNativeComponent";

const App: () => Node = () => {
  // ... other App code ...
  const [content, setContent] = useState('Hello World!');
  return (
    <SafeAreaView>
      <RTNCenteredText
        text={content}
        style={{ width: "100%", height: 50 }}
        onTextTouch={e => {
          if (e.type == '0') {
            setContent('Touch Down');
            return;
          }
          if (e.type == '1') {
            setContent('Touch Up');
            return;
          }
          if (e.type == '2') {
            setContent('Touch Move');
            return;
          }
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

### 4. 其他

#### 4.1 如何将 HarmonyOS 原生部分的代码打成 *.har 包

当完成上述操作，调试验证完毕您所需要的功能开发后，即可进行模块 har 打包操作

1. 打开 DevEco Studio，选中 `rtn_centered_text` 模块，点击 Build -> Make Module 'rtn_centered_text'；

2. .har 会生成到目录 rtn_centered_text/build/default/outputs/default (模块 har 包生成的默认路径)