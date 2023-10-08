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

### 3. 原生代码

#### Android

#### Harmony


