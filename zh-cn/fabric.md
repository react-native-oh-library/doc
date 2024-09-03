> [!WARNING] 本文档为非公开文档，仅用于三方库使用和开发指导，不涉及任何 React Native OpenHarmony 框架的信息，且会随着 React Native OpenHarmony 框架持续迭代更新，当前版本不代表最终展示版本。

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

同样的，我们按照一般的三方库目录结构来配置:

```
.
├── MyApp
└── RTNCenteredText
    ├── android（Android 的原生实现代码，若有）
    ├── ios（iOS 的原生实现代码，若有）
    ├── harmony（HarmonyOS 的原生实现代码）
    └── src （js/ts代码）
```

### 2. 声明 JavaScript 接口

新架构要求必须使用强类型风格语言声明 JavaScript 接口（Flow 和 TypeScript 皆可）。Codegen 会根据这些接口声明来生成强类型的语言，其中包括 C++、Objective-C 和 Java。

对于声明类型的代码文件必须满足以下两点要求：

1. 文件必须使用 `<MODULE_NAME>NativeComponent` 命名，在使用 Flow 时，以 `.js` 或 `.jsx` 为后缀名；在使用 Typescript 时，以 `.ts` 或 `.tsx` 为后缀名。Codegen 只会找到匹配这些命名规则的文件；

2. 代码中必须要输出 HostComponent 对象。

以下是使用 Flow 和 TypeScript 声明的 RTNCenteredText 组件。在 `js` 目录中，创建一个命名为 `RTNCenteredText` 并带有相应后缀名的文件。

<!-- tabs:start -->

#### **flow**

RTNCenteredTextNativeComponent.js

```js
// @flow strict-local

import type { ViewProps } from "react-native/Libraries/Components/View/ViewPropTypes";
import type { HostComponent } from "react-native";
import codegenNativeComponent from "react-native/Libraries/Utilities/codegenNativeComponent";

type NativeProps = $ReadOnly<{|
  ...ViewProps,
  text: ?string,
  // 在这里添加其他属性
|}>;

export default (codegenNativeComponent<NativeProps>(
  "RTNCenteredText"
): HostComponent<NativeProps>);
```

#### **typescript**

RTNCenteredTextNativeComponent.ts

```ts
import type { ViewProps } from "ViewPropTypes";
import type { HostComponent } from "react-native";
import codegenNativeComponent from "react-native/Libraries/Utilities/codegenNativeComponent";

export interface NativeProps extends ViewProps {
  text?: string;
  // 添加其它 props
}

export default codegenNativeComponent<NativeProps>(
  "RTNCenteredText"
) as HostComponent<NativeProps>;
```

<!-- tabs:end -->

在声明文件的顶部导入了一些内容。以下是开发 Fabric 组件必须要导入的内容：

- `HostComponent` 类型: 导出的组件需要与这个类型保持一致；
- `codegenNativeComponent` 函数：负责将组件注册到 JavaScript 运行时。
  声明文件的中间部分包含了组件的 props。Props（"properties" 的缩写）是用于自定义 React 组件的参数信息。在本例中，需要控制组件的 text 属性。

在声明文件的最后部分，导出了泛型函数 `codegenNativeComponent` 的返回值，此函数需要传递组件的名称。

### 3. Codegen 配置

#### 3.1 配置 `package.json` 文件

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

#### 3.2 选择 Fabric 的原生实现方式，配置 codegen

RNOH 有特殊的架构限制，需要开发者在开发前根据需求选择好使用 ArkTS API 还是 CAPI 实现 Fabric。

关于如何选择，请阅读这篇[说明文档](https://gitee.com/react-native-oh-library/usage-docs/blob/master/zh-cn/capi-architecture.md)。

此步操作是要将 Codegen 的配置声明到 harmony.codegenConfig 字段。

- version: 枚举值，1 代表ArkTS组件的codegen版本，2 代表CAPI组件的codegen版本；（关于ArkTS和CAPI组件请见2.2的详细说明）
- specPaths：用于找到 js 接口声明文件的相对路径，它将被 Codegen 解析

> [!WARNING] 接入 codegen 之后，同一个模块中 ArkTS 版本和 CAPI 版本的 Fabric 无法共存，请先选择好实现方式

在 `package.json` 中新增 harmony.codegenConfig 字段：

##### Option1: ArkTS API 实现 Fabric

```json
{
  // ...
  "harmony": {
    "codegenConfig": {
      "version": 1,
      "specPaths": ["./src"],
    }
  }
}
```

##### Option2: C-API 实现 Fabric

```json
{
  // ...
  "harmony": {
    "codegenConfig": {
      "version": 2,
      "specPaths": ["./src"],
    }
  }
}
```

#### 3.3 codegen通用配置项

HarmonyOS 需要在 RN 工程中通过运行脚本来执行 Codegen。

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

1. --rnoh-module-path: 指定 @rnoh/react-native-openharmony 模块的相对路径，用于存储生成的 ts 文件；如果使用 har 包引入 RNOH SDK，则需要指向安装之后的路径，比如：./harmony/entry/oh_modules/@rnoh/react-native-openharmony"

2. --cpp-output-path: 指定用于存储生成的 C++ 文件的输出目录的相对路径，默认 ./harmony/entry/src/main/cpp/generated；

3. --project-root-path: 包根目录的相对路径。

### 4. 实现原生组件

#### Option1: 使用 ArkTS API 实现原生组件

HarmonyOS 平台中 ArkTS 版本的 Fabric 组件的原生代码必须包含以下三个部分：

1. 创建用于实现组件的 RTNCenteredText.ets
2. 创建 RTNCenteredTextPackage.ts
3. 创建用于导出模块的 index.ets 和 ts.ts
4. 修改 oh-package.json5，hvigorfile.ts，module.json5

> [!TIP] 可以在 DevEco Studio 中通过 File -> New -> Module.. -> Static Lirbrary 创建空壳模块，以此为基础修改文件内容

HarmonyOS 原生代码文件结构应为如下：

```md
harmony
└── rtn-centered-text
    ├── src
    │   └── main
    │       ├──ets
    |       |   ├── RTNCenteredTextPackage.ts
    │       │   └── RTNCenteredText.ets
    │       └── modules.json5         
    ├── build-profile.json5
    ├── hvigorfile.ts
    ├── index.ets
    ├── oh-package.json5
    └── ts.ts
```

<!-- tabs:start -->

#### **RTNCenteredText.ets**

```ts
import { RNComponentContext, RNViewBase } from '@rnoh/react-native-openharmony';
// import codegen 生成的内容
import { RNC } from "@rnoh/react-native-openharmony/generated";

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
        // 组件属性更新时进入的回调
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
    }
  }
}
```

<!-- tabs:end -->

 <!-- tabs:start -->

#### **RTNCenteredPackage.ts**

```ts
import { RNPackage } from "@rnoh/react-native-openharmony/ts";
import type {
  DescriptorWrapperFactoryByDescriptorTypeCtx,
  DescriptorWrapperFactoryByDescriptorType,
} from "@rnoh/react-native-openharmony/ts";
// import codegen 生成的内容
import { RNC } from "@rnoh/react-native-openharmony/generated/ts";

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

创建 `index.ets`

<!-- tabs:start -->

#### **index.ets**

```ts
export * from "./ts";
export * from "./src/main/ets/RTNCenteredText";
```

<!-- tabs:end -->

#### Option2: 使用 C-API 实现原生组件

HarmonyOS 平台中 C-API 版本的 Fabric 组件的原生代码必须包含以下部分：

1. 创建用于实现组件的 RTNCenteredTextComponentInstance.h、RTNCenteredTextComponentInstance.cpp；
2. 创建用于对接 ArkUI 的 xxxNode.h、xxxNode.cpp；（若框架已经实现相关的 Node，此步可以跳过。本例中需要用到的 TextNode、StackNode 框架已经实现，故无需自行创建）
3. Cpp 脚手架（请看 codegen 章节）
4. 修改 oh-package.json5，hvigorfile.ts，module.json5

> [!TIP] 可以在 DevEco Studio 中通过 File -> New -> Module.. -> Static Lirbrary 创建空壳模块，以此为基础修改文件内容

HarmonyOS 原生代码文件结构应为如下：

```md
harmony
└── rtn-centered-text
    ├── src
    │   └── main
    │       ├── cpp
    │       │   ├── CMakeLists.txt
    │       │   ├── ComponentDescriptors.h
    │       │   ├── EventEmitters.cpp
    │       │   ├── EventEmitters.h
    │       │   ├── Props.cpp
    │       │   ├── Props.h
    │       │   ├── ShadowNodes.cpp
    │       │   ├── ShadowNodes.h
    │       │   ├── CenteredTextJSIBinder.h
    │       │   ├── CenteredTextPackage.h
    │       │   ├── RTNCenteredTextComponentInstance.h
    │       │   └── RTNCenteredTextComponentInstance.cpp
    │       └── modules.json5         
    ├── build-profile.json5
    ├── hvigorfile.ts
    └── oh-package.json5
```

创建 `RTNCenteredTextComponentInstance.h`

<!-- tabs:start -->

#### **RTNCenteredTextComponentInstance.h**

```cpp
#pragma once
#include "ShadowNodes.h"
#include "RNOH/CppComponentInstance.h"
#include "RNOH/arkui/StackNode.h"
#include "RNOH/arkui/TextNode.h"

namespace rnoh {
class RTNCenteredTextComponentInstance : public CppComponentInstance<facebook::react::RTNCenteredTextShadowNode> {
private:
    using FragmentTouchTargetByTag = std::unordered_map<facebook::react::Tag, std::shared_ptr<TouchTarget>>;

    TextNode m_textNode{};
    StackNode m_stackNode{};

public:
    RTNCenteredTextComponentInstance(Context context);
    StackNode &getLocalRootArkUINode() override;

protected:
    void onPropsChanged(SharedConcreteProps const &props) override;
};
} // namespace rnoh
```

<!-- tabs:end -->

创建 `RTNCenteredTextComponentInstance.cpp`

<!-- tabs:start -->

#### **RTNCenteredTextComponentInstance.cpp**

```cpp
#include "RTNCenteredTextComponentInstance.h"

namespace rnoh {

RTNCenteredTextComponentInstance::RTNCenteredTextComponentInstance(Context context)
    : CppComponentInstance(std::move(context)) {
    m_stackNode.insertChild(m_textNode, 0);
}

StackNode &RTNCenteredTextComponentInstance::getLocalRootArkUINode() { return m_stackNode; }

void RTNCenteredTextComponentInstance::onPropsChanged(SharedConcreteProps const &props) {
    CppComponentInstance::onPropsChanged(props);
    if (props == nullptr) {
        return;
    }
    m_textNode.setTextContent(props->text);
    m_textNode.setFontSize(30.0);
    m_textNode.setAlignment(ARKUI_ALIGNMENT_CENTER);
}

} // namespace rnoh
```

<!-- tabs:end -->

修改 `RTNCenteredTextPackage.h`

<!-- tabs:start -->

#### **RTNCenteredTextPackage.cpp**

```cpp
#include "RNOH/Package.h"
#include "ComponentDescriptors.h"
#include "RTNCenteredTextJSIBinder.h"
#include "RTNCenteredTextComponentInstance.h"

namespace rnoh {

class RTNCenteredTextComponentInstanceFactoryDelegate : public ComponentInstanceFactoryDelegate {
public:
    using ComponentInstanceFactoryDelegate::ComponentInstanceFactoryDelegate;

    ComponentInstance::Shared create(ComponentInstance::Context ctx) override {
        if (ctx.componentName == "RTNCenteredText") {
            return std::make_shared<RTNCenteredTextComponentInstance>(std::move(ctx));
        }
        return nullptr;
    }
};

class RTNCenteredTextPackage : public Package {
public:
    RTNCenteredTextPackage(Package::Context ctx): Package(ctx) {}

    ComponentInstanceFactoryDelegate::Shared createComponentInstanceFactoryDelegate() override {
        return std::make_shared<RTNCenteredTextComponentInstanceFactoryDelegate>();
    }

    std::vector<facebook::react::ComponentDescriptorProvider> createComponentDescriptorProviders() override {
        return {facebook::react::concreteComponentDescriptorProvider<facebook::react::RTNCenteredTextComponentDescriptor>()};
    }

    ComponentJSIBinderByString createComponentJSIBinderByName() override {
        return {{"RTNCenteredText", std::make_shared<RTNCenteredTextJSIBinder>()}};
    }
};
} // namespace rnoh
```

<!-- tabs:end -->

#### ArkTS Fabric 和 C-API Fabric 共有部分

修改 `oh-package.json5`，`hvigorfile.ts`，`module.json5`，或自行创建

 <!-- tabs:start -->

#### **oh-package.json5**

```json
{
  "license": "ISC",
  "types": "",
  "devDependencies": {},
  "name": "rtn-centered-text",
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
  "module": {
    "name": "centered_text",
    "type": "har",
    "deviceTypes": ["default"]
  }
}
```

<!-- tabs:end -->

### 5. 将 Fabric 组件添加到 App

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
cd RTNCenteredText

// 打包模块
npm pack

// 进入 App 工程
cd ../MyApp

// 本地路径安装模块
npm i file:../RTNCenteredText/rtn-centered-text-0.0.1.tgz

// 执行以下命令执行 codegen (HarmonyOS only)

npm run codegen

```

此命令会将 RTNCenteredText 模块添加到 App 内的 node_modules 目录。

#### 5.2 原生工程配置项

> [!tip] 待完善能力：HarmonyOS 平台目前暂时不支持 AutoLink，所以需要自行配置。

首先使用 DevEco Studio 打开 React-Native 项目里的鸿蒙工程 `harmony`

目前 HarmonyOS 工程暂不支持引入工程外的模块，所以需要手动将模块的 HarmonyOS 源码复制到工程内。

- 复制 `RTNCenteredText/harmony/centered_text` 到 `harmony` 工程根目录下。

- 修改 `MyApp/harmony/build-profile.json5`，在 modules 字段添加：

```json
{
...
  modules: [
    ...
    {
      name: 'centered_text',
      srcPath: './centered_text',
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
    "rtn-centered-text": "file:../../node_modules/RTNCenteredText/harmony/rtn-centered-text"
  }
```

- 点击右上角的 `sync` 按钮同步工程，或在终端运行以下命令

```bash
cd entry
ohpm install
```

##### 5.2.1 ArkTS 组件特有配置项

打开 `MyApp/harmony/entry/src/main/ets/pages/Index.ets`，添加：

```diff
...
+ import { RTNCenteredText } from "rtn-centered-text"

+ const arkTsComponentNames: Array<string> = [RTNCenteredText.NAME];
@Builder
export function buildCustomRNComponent(ctx: ComponentBuilderContext) {
  Stack() {
+   if (ctx.componentName === RTNCenteredText.NAME) {
+     RTNCenteredText({
+     tag: ctx.tag,
+     ctx: ctx.rnComponentContext,
+   })
+ }
 //...
  }
  .position({x: 0, y: 0})
}
//...
```

打开 `MyApp/harmony/entry/src/main/ets/RNPackageFactory.ts`，添加：

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

##### 5.2.2 C-API 组件特有配置项

打开 `MyApp/harmony/entry/src/main/cpp/CMakeLists.txt`，添加：

```diff
project(rnapp)
cmake_minimum_required(VERSION 3.4.1)
set(CMAKE_SKIP_BUILD_RPATH TRUE)
set(RNOH_APP_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
set(NODE_MODULES "${CMAKE_CURRENT_SOURCE_DIR}/../../../../../node_modules")
+ set(OH_MODULES_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../oh_modules")
set(RNOH_CPP_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../../../../react-native-harmony/harmony/cpp")
set(LOG_VERBOSITY_LEVEL 1)
set(CMAKE_ASM_FLAGS "-Wno-error=unused-command-line-argument -Qunused-arguments")
set(CMAKE_CXX_FLAGS "-fstack-protector-strong -Wl,-z,relro,-z,now,-z,noexecstack -s -fPIE -pie")
set(WITH_HITRACE_SYSTRACE 1) # for other CMakeLists.txt files to use
add_compile_definitions(WITH_HITRACE_SYSTRACE)

add_subdirectory("${RNOH_CPP_DIR}" ./rn)

# RNOH_BEGIN: manual_package_linking_1
add_subdirectory("../../../../sample_package/src/main/cpp" ./sample-package)
+ add_subdirectory("${OH_MODULES_DIR}/rtn-centered-text-capi/src/main/cpp" ./centered-text)
# RNOH_END: manual_package_linking_1

file(GLOB GENERATED_CPP_FILES "./generated/*.cpp")

add_library(rnoh_app SHARED
    ${GENERATED_CPP_FILES}
    "./PackageProvider.cpp"
    "${RNOH_CPP_DIR}/RNOHAppNapiBridge.cpp"
)
target_link_libraries(rnoh_app PUBLIC rnoh)

# RNOH_BEGIN: manual_package_linking_2
target_link_libraries(rnoh_app PUBLIC rnoh_sample_package)
+ target_link_libraries(rnoh_app PUBLIC rtn_centered_text)
# RNOH_END: manual_package_linking_2
```

打开 `MyApp/harmony/entry/src/main/cpp/PackageProvider.cpp`，添加：

```diff
#include "RNOH/PackageProvider.h"
#include "SamplePackage.h"
+ #include "RTNCenteredTextPackage.h"

using namespace rnoh;

std::vector<std::shared_ptr<Package>> PackageProvider::getPackages(Package::Context ctx) {
    return {
      std::make_shared<RNOHGeneratedPackage>(ctx),
      std::make_shared<SamplePackage>(ctx),
+     std::make_shared<RTNCenteredTextPackage>(ctx),
    };
}
```

编译、运行即可。

#### 5.3 JavaScript

最后，操作以下步骤，您就可以在 JavaScript 调用组件了。

1. 在 js 文件中导入组件。假设要在 App.js 进行导入，需要添加这行代码：

```js
import RTNCenteredText from "rtn-centered-text/src/RTNCenteredTextNativeComponent";
```

2. 接下来，在 React Native 组件里进行调用。调用的语法和其它组件相同：

**App.js**

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
  return (
    <SafeAreaView>
      <RTNCenteredText
        text="Hello World!"
        style={{ width: "100%", height: 50 }}
      />
    </SafeAreaView>
  );
};

export default App;
```

<!-- tabs:end -->

现在，您可以运行 App 并查看在屏幕上显示的组件。

> [!TIP] 可通过 npm run start 使用热更新
