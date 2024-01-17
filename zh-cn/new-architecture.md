> [!WARNING] 本文档为非公开文档，仅用于三方库使用和开发指导，不涉及任何 React Native OpenHarmony 框架的信息，且会随着 React Native OpenHarmony 框架持续迭代更新，当前版本不代表最终展示版本。

# 新架构介绍

从 0.68 版本开始，React Native 提供了新架构，它为开发者提供了构建高性能和响应式应用的新功能。

新架构由两大支柱组成：

1. 新的原生模块体系 - `Turbo Modules`，一个支持与本地代码高效、灵活集成的框架。
2. `Fabric` 渲染器和组件，它提供了更好的功能、跨平台的一致性和渲染性能。

另外还有一个帮助我们避免编写重复代码的工具 `Codegen`，它通过 JavaScript 的静态类型化，生成新架构所需的 C++ 模板。（目前仅支持 Android 和 iOS 平台，HarmonyOS 版本还在开发中）。

## 开始使用新架构

接下来，如果你想深入了解新架构的开发，可以进入 [TurboModules](/zh-cn/turbomodule.md) 和 [Fabric 组件](/zh-cn/fabric.md)学习如何从头构建一个 TurboModule 和 Fabric 组件。

如果你对 TurboModule 和 Fabric 组件已经有所了解，可以直接进入 [迁移三方库到新架构](/zh-cn/migration.md)来上手实操。
