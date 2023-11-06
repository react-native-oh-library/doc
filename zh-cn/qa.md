# 常见问题Q&A

## 1. 代码与调试相关

### 1.1 如何在C++侧创建结点时打印日志？

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

### 1.3 如何加快打bundle的速度？

本地调试可以注释掉 `node_modules/@rnoh/react-native-harmony-cli/dist/commands/bundle-harmony.js` 文件下的如下两行代码来提高打bundle的速度：

```js
const assets = yield retrieveAssetsData(metroConfig, buildOptions);
copyAssets(assets, args.assetsDest);
```

### 1.4 bob-build或使用babel时 Platform.OS 报错：'"android" | "windows" | "macos" | "web"' and '"harmony"' have no overlap.

将 node_modules 的 `react-native/Libraries/Utilities/Platform.d.ts` 替换成 react-native-harmony 包里 `react-native-harmony/Libraries/Utilities/Platform.d.ts`。




## 2. 环境配置相关

### 2.1 ArkTs语法校验报错

在 `entry/src/main/module.json` 添加：
```json
"metadata": [
    {
        "name": "ExecuteArkTSLinter",
        "value": "false"
    }
]
```

### 2.2 蓝区RNOH打包报错：third-party/boost/libs/context/src/asm/jump_arm/aapcs_elf_gas.S 等几个汇编源文件，编译.so包失败

在 entry 模块的 `build-profile.json5` 文件添加配置信息：

```json
"buildOption": {
    "externalNativeOptions": {
        "arguments": "-DBUILD_TEST=OFF",
    }
}

```

