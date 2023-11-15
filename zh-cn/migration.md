# 迁移三方库到新框架

> [!tip] 迁移到新框架分为两步：JS 代码迁移和原生代码迁移。这里只介绍 JS 代码的迁移，Android 和 iOS 原生代码的迁移与鸿蒙化无关，按需自行学习。

> tags: 可通过社区查找参考代码，在仓库的 pull request 搜索 new architecture 或者 Fabric/Turbo Module 以查看社区里迁移中的代码，进行参考。

## 如何判断三方库是否为新框架

主要从三个方面来判断：

1. 在 `package.json` 里判断是否使用了 Codegen；
2. 是否有 Native<MODULE_NAME> 命名的 js/ts 文件并且里面导出了 TurboModuleRegistrySpec 对象；
3. 是否有 <MODULE_NAME>NativeComponent 命名的 js/ts 文件并且里面导出了 codegenNativeComponent 对象。

## JS 代码迁移新架构

### TurboModules

Turbo Modules 对应旧架构的 Native Modules，所以首先要先找到旧架构里声明 Native Modules 的部分。
