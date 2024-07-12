# RN 三方库移植

## 对标策略

由于每一个 RN 三方库原库就支持一个至多个平台（android、iOS、mac、window 等），所以三方库移植则是对标移动端在 HarmonyOS 上支持相同的属性接口规格和一致的效果。

移动端主要对标 iOS：

- 若一个库支持 iOS 和 android 则 iOS 支持的接口均需要实现，而仅支持 android 的接口若 HarmonyOS 支持则也需要移植实现；
- 若该库仅支持 android 如 [react-native-SmartRefreshLayout](https://github.com/react-native-studio/react-native-SmartRefreshLayout) ，则对标 android 做实现；
- 效果对标遵循以上接口的对标方式。

## 三方库移植

三方库移植应该基于正确的对标策略，站在质量和性能的角度去移植实现，最终需做到一样 RN 代码输入一样的功能效果输出和更好的性能效果。

### RN JS 侧移植

JS 侧代码是多端通用的，因此我们在移植时不得随意修改原库的 JS 侧的代码逻辑。针对 JS 侧代码可做的修改项有：添加 HarmonyOS 侧的平台判断和新架构适配。

**HarmonyOS 侧的平台判断**

每个库可能在自身代码添加许多平台判断，因此我们需要根据我们的对标策略，适当的添加平台判断代码：

1. Platform 接口中 HarmoyOS 判断

   ```
     if (Platform.OS === 'ios') {
       headerHeight = 32;
     } else if (Platform.OS === 'android') {
       headerHeight = 56;
     } else if (Platform.OS === 'harmony') {
       headerHeight = 32;
     } else {
       headerHeight = 64;
     }

      headerTitleAlign = Platform.select({
         ios: 'center',
         harmony: 'center',
         default: 'left',
       })
   ```

2. 添加 HarmonyOS 平台后缀的文件

   如 [react-native-tab-view](https://github.com/react-navigation/react-navigation/tree/main/packages/react-native-tab-view/src) 中包含，Pager.android.tsx、Pager.ios.tsx、Pager.tsx 代码文件，我们则需要对标 iOS 添加**Pager.harmoy.tsx**文件，代码实现则复制至 Pager.ios.tsx。若在该库中缺少这部分添加则该库会走到 window 侧实现，虽然可以运行但性能效果缺相差甚远。类似的需要注意的代码文件还有 xx.native.tsx。

以上列举了常见的平台判断，在移植时排除以上内容后必须在 js 代码中全量搜索以下关键词，以确保平台判断无疏漏：

- ios
- android
- Platform

**新架构适配**

参考新架构迁移指南

### RN native 侧移植

由于系统和语言差异，iOS 和 android 的代码实现和 HarmonyOS 必然存在差异，但是 iOS 支持的接口大部分 HarmonyOS 都有类似的接口，所以我们移植需要参考原库的设计实现思路和方案，特别是一些涉及到大量布局计算的库如 [flash-list](https://github.com/Shopify/flash-list)。

因为各个库 native 实现各不相同，以下列举了简单的移植思路:

1. 根据 [iOS 开发文档](https://developer.apple.com/documentation/) 和 [android 开发文档](https://developer.android.google.cn/guide?hl=th)，配合 [HarmonyOS 文档](https://developer.huawei.com/consumer/cn/doc/) 编写纯 HarmonyOS 的实现 demo，确保相关 OS 接口均有支持。
2. 使用 codegen 打通 RN 至原生的通道。
3. 配合 RN native 接口将实现集成进 RN 中。
4. 以上如有对标原库不支持的可提交相关需求推进。

RN native 接口查看

ArkTS：

通过 `ctx` 对象和 `@rnoh/react-native-openharmony` 的导出项中寻找对标接口。如实现返回桌面的监听。

```
this.ctx.rnInstance.subscribeToLifecycleEvents("BACKGROUND", () => {
 			// todo
    })
```

C API:

可通过`#include "RNOH/CppComponentInstance.h"`下的的 `CppComponentInstance` 、`context`和基类查找。
