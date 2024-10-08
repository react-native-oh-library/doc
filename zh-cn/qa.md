> [!WARNING] 本文档为非公开文档，仅用于三方库使用和开发指导，不涉及任何 React Native OpenHarmony 框架的信息，且会随着 React Native OpenHarmony 框架持续迭代更新，当前版本不代表最终展示版本。

# 常见问题 Q&A

## 1. 代码与调试相关

### 1.1 如何在 C++侧创建结点时打印日志？

A：

在 `UIManager.cpp` 下的 UIManager::creatNode() 里添加：

```cpp
std::string json = folly::toJson((folly::dynamic)rawProps);
LOG(INFO) << "UIManger::createNode rawProps:" << json.c_str() << "name:" << name << "Tag:" << tag;
```

需要引入：

```cpp
#include "folly/json.h"
#include <glog/logging.h>
```

### 1.2 为什么有些日志不打印？

在每次插入手机时可以输入以下命令来避免日志打印被流控：

```bash
hdc shell
hilog -Q domainoff
hilog -Q pidoff
```

### 1.3 如何加快打 bundle 的速度？

本地调试可以注释掉 `node_modules/@rnoh/react-native-harmony-cli/dist/commands/bundle-harmony.js` 文件下的如下两行代码来提高打 bundle 的速度：

```js
const assets = yield retrieveAssetsData(metroConfig, buildOptions);
copyAssets(assets, args.assetsDest);
```

### 1.4 bob-build 或使用 babel 时 Platform.OS 报错：'"android" | "windows" | "macos" | "web"' and '"harmony"' have no overlap.

将 node_modules 的 `react-native/Libraries/Utilities/Platform.d.ts` 替换成 react-native-harmony 包里 `react-native-harmony/Libraries/Utilities/Platform.d.ts`。

### 1.5 bob-build 时报错："harmony/\*\*\*/hvigorfile.ts:2:26 - error TS2307: Cannot find module '@ohos/hvigor-ohos-plugin' or its corresponding type declarations."

把 hvigorfile.ts 所在目录(如："harmony" )添加到添加到 tsconfig.build.json 文件的 exclude 中，bob-build 时就不会对 harmony 下面文件编译成 js。

**tsconfig.build.json**

```json
{
  "extends": "./tsconfig",
  "exclude": ["example", "harmony"]
}
```

## 2. 环境配置相关

### 2.1 ArkTs 语法校验报错

在 `entry/src/main/module.json` 添加：

```json
"metadata": [
    {
        "name": "ExecuteArkTSLinter",
        "value": "false"
    }
]
```

### 2.2 蓝区 RNOH 打包报错：third-party/boost/libs/context/src/asm/jump_arm/aapcs_elf_gas.S 等几个汇编源文件，编译.so 包失败

在 entry 模块的 `build-profile.json5` 文件添加配置信息：

```json
"buildOption": {
    "externalNativeOptions": {
        "arguments": "-DBUILD_TEST=OFF",
    }
}

```

### 2.3 mac 鸿蒙模拟器已启动，依然无法发现设备

先把 android 模拟器关闭即可。

### 2.4 mac 鸿蒙模拟器，重新打开后之前安装的 app 没了

模拟器暂时不支持保存数据
