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

