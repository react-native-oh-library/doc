# TurboModules

Turbo Modules是升级版的Native Modules，是基于JSI开发的一套JS与Native交互的轻量级框架。TurboModules 本质上的作用是导出一系列的Native方法供JS使用。

详细的原理分析可以看：[React Native之新架构中的Turbo Module实现原理分析](https://cloud.tencent.com/developer/article/1889895)

![turbomodules](../img/turbomodule2.png)

## 目录配置

我们按照一般的三方库目录结构来配置:

```
.
└── MyFabric
    ├── android（Android 的原生实现代码）
    ├── ios（iOS 的原生实现代码）
    ├── harmony（Harmony 的原生实现代码）
    └── src （js/ts代码）
```

## 如何创建 TurboModule

创建一个 Turbo Native Module 分为以下步骤：

1. 声明 JavaScript 接口类型；
2. 配置模块以支持 Codegen 自动生成脚手架代码；
3. 编写原生代码完成模块实现。

### 1. 声明 JavaScript 接口

### 2. Codegen 配置

#### Android

#### Harmony

### 3. 原生代码

#### Android

#### Harmony