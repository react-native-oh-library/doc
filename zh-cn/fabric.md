# Fabric 组件

Fabric 组件是一种使用 Fabric 渲染器渲染并展示在屏幕上的 UI 组件。

![fabric](../img/fabric.png)

在开发 Fabric 组件前，需要先创建一个 JavaScript 接口描述文件。之后 Codegen 会根据这个文件创建一些 C++ 脚手架代码，用于将部分组件逻辑（比如调用原生平台接口能力）与 React Native 结合起来。C++ 代码在各个平台都是一样的，只要组件能够与生成的 C++ 代码连接起来，就可以导入到 App 并运行。

因为 Harmony 平台暂时还没有 codegen 工具，所以我们需要使用 Android 平台的 codegen 来生成相关的 C++ 代码，然后复制到 Harmony 平台使用。

## 目录配置

同样的，我们按照一般的三方库目录结构来配置:

```
.
└── MyFabric
    ├── android（Android 的原生实现代码）
    ├── ios（iOS 的原生实现代码）
    ├── harmony（Harmony 的原生实现代码）
    └── src （js/ts代码）
```

## 如何创建 Fabric 组件

创建一个 Fabric 组件与创建 TurboModule 类似，需要遵循以下步骤：

1. 声明 JavaScript 接口；
2. 配置组件以用于 Codegen 生成统一代码，生成的代码可添加为 App 的依赖；
3. 编写所需的原生代码。

### 1. 声明 JavaScript 接口

### 2. Codegen 配置

#### Android

#### Harmony

Harmony 平台暂时还没有 Codegen，所以我们需要手动运行 Android 的 Codegen，然后把生成的代码复制过来使用。

!> 请务必先把 Android 的 Codegen 配置好再执行以下操作

首先我们需要一个 React-Native App来执行 Codegen，假设 App 的目录是和 当前目录平级的 `MyApp`，执行以下命令来创建一个 Gradle 任务来执行 Codegen。

!> 在运行 Codegen 之前，您需要在 Android 中的 App 启动新架构。您可以通过修改 gradle.properties 文件中的 newArchEnabled 属性，将 false 改为 true。

```bash
cd MyApp
yarn add ../RTNCalculator
cd android
./gradlew generateCodegenArtifactsFromSchema
```

生成后的代码保存在 `MyApp/node_modules/rtn-calculator/android/build/generated/source/codegen` 目录，并呈以下结构：

```md
codegen
├── java
│   └── com
│       └── RTNCalculator
│           └── NativeCalculatorSpec.java
├── jni
│   ├── Android.mk
│   ├── RTNCalculator-generated.cpp
│   ├── RTNCalculator.h
│   └── react
│       └── renderer
│           └── components
│               └── RTNCalculator
│                   ├── ComponentDescriptors.h
│                   ├── EventEmitters.cpp
│                   ├── EventEmitters.h
│                   ├── Props.cpp
│                   ├── Props.h
│                   ├── ShadowNodes.cpp
│                   └── ShadowNodes.h
└── schema.json
```

`RTNCalculator` 目录下的代码是 Harmony 需要的。将这些代码复制到 `harmony/rtn-calculator/src/main/cpp` 文件夹下，并在同级目录创建

### 3. 原生代码

#### Android

#### Harmony


