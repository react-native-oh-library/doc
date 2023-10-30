# TurboModules

Turbo Modules是升级版的Native Modules，是基于JSI开发的一套JS与Native交互的轻量级框架。TurboModules 本质上的作用是导出一系列的Native方法供JS使用。

详细的原理分析可以看：[React Native之新架构中的Turbo Module实现原理分析](https://cloud.tencent.com/developer/article/1889895)

![turbomodules](../img/turbomodule2.png)

## 如何创建 TurboModule

创建一个 Turbo Native Module 分为以下步骤：

1. 声明 JavaScript 接口类型；
2. 配置模块以支持 Codegen 自动生成脚手架代码；
3. 编写原生代码完成模块实现。

接下来会创建一个简单的名为 `RTNCalculator` 的 TurboModule 作为示例。

### 1. 目录配置

我们按照一般的三方库目录结构来配置:

```md
.
└── RTNCalculator
    ├── android（Android 的原生实现代码）
    ├── ios（iOS 的原生实现代码）
    ├── harmony（Harmony 的原生实现代码）
    └── src （js/ts代码）
```

### 2. 声明 JavaScript 接口

新架构要求必须使用强类型风格语言声明 JavaScript 接口（Flow 和 TypeScript 皆可）。Codegen 会根据这些接口声明来生成强类型的语言，其中包括 C++、Objective-C 和 Java。

对于声明类型的代码文件必须满足以下两点要求：

1. 文件必须使用 `Native<MODULE_NAME>`命名，在使用 Flow 时，以 .js 或 .jsx 为后缀名；在使用 **Typescript** 时，以 `.ts` 或 `.tsx` 为后缀名。**Codegen** 只会找到匹配这些命名规则的文件；
2. 代码中必须要输出 `TurboModuleRegistrySpec` 对象

<!-- tabs:start -->

#### **flow**

NativeCalculator.js
```js
// @flow
import type { TurboModule } from 'react-native/Libraries/TurboModule/RCTExport';
import { TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  add(a: number, b: number): Promise<number>;
}
export default (TurboModuleRegistry.get<Spec>(
  'RTNCalculator'
): ?Spec);
```

#### **typescript**

NativeCalculator.ts
```ts
import type {TurboModule} from 'react-native/Libraries/TurboModule/RCTExport';
import {TurboModuleRegistry} from 'react-native';

export interface Spec extends TurboModule {
  add(a: number, b: number): Promise<number>;
}

export default TurboModuleRegistry.get<Spec>(
  'RTNCalculator',
) as Spec | null;
```
<!-- tabs:end -->

在代码顶部需导入以下两个声明文件：

- 类型 TurboModule ：定义 Turbo Native Module 的基础接口
- JS 模块 TurboModuleRegistry：包含了用于加载 Turbo Native Module 的函数

代码的第二个部分就是针对 Turbo Native Module 的接口声明。在本例中，接口声明了 `add` 函数，它将用于接受两个数字并返回一个包装数字的 Promise。为声明 Turbo Native Module，此接口**必须**命名为 Spec。

最后，调用 `TurboModuleRegistry.get` 并传入模块名，它将在 Turbo Native Module 可用的时候进行加载。

### 3. Codegen 配置
接下来，需要为 Codegen 和自动链接添加一些配置。Codegen的作用是生成 C++ 脚手架代码，负责串联 JS 和原生侧。

有一些配置文件在 Android/iOS 平台是通用的，而有的仅能在某一平台使用。

Harmony 平台暂时不支持 Codegen，TurboModule 的 C++ 代码需要自行编写。

#### Shared

shared 是 package.json 文件中的一个配置项，它将在 yarn 安装模块时被调用。请在 `RTNCalculator` 的根目录创建 `package.json` 文件。

```json
{
  "name": "rtn-calculator",
  "version": "0.0.1",
  "description": "Add numbers with TurboModules",
  "react-native": "src/index",
  "source": "src/index",
  "files": [
    "src",
    "android",
    "ios",
    "harmony",
    "rtn-calculator.podspec",
    "!android/build",
    "!ios/build",
    "!**/__tests__",
    "!**/__fixtures__",
    "!**/__mocks__"
  ],
  "keywords": ["react-native", "ios", "android", "harmony"],
  "repository": "https://github.com/<your_github_handle>/rtn-calculator",
  "author": "<Your Name> <your_email@your_provider.com> (https://github.com/<your_github_handle>)",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/<your_github_handle>/rtn-calculator/issues"
  },
  "homepage": "https://github.com/<your_github_handle>/rtn-calculator#readme",
  "devDependencies": {},
  "peerDependencies": {
    "react": "*",
    "react-native": "*"
  },
  "codegenConfig": {
    "name": "RTNCalculatorSpec",
    "type": "modules",
    "jsSrcsDir": "src",
    "android": {
      "javaPackageName": "com.rtncalculator"
    }
  }
}
```

将 Codegen 的配置声明到 codegenConfig 字段。codegenConfig 是一个用于存放要生成的第三方库的对象数组，每个对象又包含其它三个字段：

- name：第三方库的名称。按照惯例，名称应以 Spec 为结尾
- type：在这个 npm 包里的模块类型。在本例中，我们开发的是 Turbo Native Module，所以值为 modules
- jsSrcsDir：用于找到 js 接口声明文件的相对路径，它将被 Codegen 解析
- android.javaPackageName：由 Codegen 生成的 Java 包名 (需与 AndroidManifest.xml 中包名一致)

#### Android

若要在 Android 平台运行 Codegen，需要创建三个文件：

1. 带有 Codegen 配置信息的 build.gradle 文件
2. AndroidManifest.xml
3. 一个实现 ReactPackage 接口的 Java 类
在文件创建完成后，android 目录文件结构应该是这样的：

```md
android
├── build.gradle
└── src
    └── main
        ├── AndroidManifest.xml
        └── java
            └── com
                └── rtncalculator
                    └── CalculatorPackage.java
```

首先，在 `android` 目录创建 `build.gradle` 文件，并配置以下内容：

<!-- tabs:start -->
#### **build.gradle**
```gradle
buildscript {
  ext.safeExtGet = {prop, fallback ->
    rootProject.ext.has(prop) ? rootProject.ext.get(prop) : fallback
  }
  repositories {
    google()
    gradlePluginPortal()
  }
  dependencies {
    classpath("com.android.tools.build:gradle:7.1.1")
  }
}

apply plugin: 'com.android.library'
apply plugin: 'com.facebook.react'

android {
  compileSdkVersion safeExtGet('compileSdkVersion', 31)
}

repositories {
  maven {
    // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
    url "$projectDir/../node_modules/react-native/android"
  }
  mavenCentral()
  google()
}

dependencies {
  implementation 'com.facebook.react:react-native:+'
}
```
<!-- tabs:end -->

其次，创建 `android/src/main` 目录，然后在这个目录内创建 `AndroidManifest.xml` 文件，并编写以下代码：

<!-- tabs:start -->
#### **AndroidManifest.xml**
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.rtncalculator">
</manifest>
```
<!-- tabs:end -->

这个 manifest 文件的用途是声明您开发的模块的 Java 包

最后，您需要一个继承 TurboReactPackage 接口的类。在运行 Codegen 前，您不用完整实现这个类。对于 App 而言，一个没有实现接口的空类就已经能当做一个 React Native 依赖，Codegen 会尝试生成其脚手架代码。

创建 `android/src/main/java/com/rtncalculator` 目录，在这个目录内创建 `CalculatorPackage.java` 文件

<!-- tabs:start -->
#### **CalculatorPackage.java**
```java
package com.rtncalculator;

import androidx.annotation.Nullable;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.module.model.ReactModuleInfoProvider;
import com.facebook.react.TurboReactPackage;

import java.util.Collections;
import java.util.List;

public class CalculatorPackage extends TurboReactPackage {

  @Nullable
  @Override
  public NativeModule getModule(String name, ReactApplicationContext reactContext) {
          return null;
  }

  @Override
  public ReactModuleInfoProvider getReactModuleInfoProvider() {
      return null;
  }
}
```
<!-- tabs:end -->

ReactPackage 接口的用途是让 React Native 为使用 App 中的 ViewManager 和 Native Modules，识别出哪些原生类需要在第三方库里导出。

Codegen 会在 App 编译的时候自动运行。

#### Harmony

!> 待完善能力：因为 Harmony 平台暂时不支持 Codegen，也不能复用安卓的 C++ 代码，所以这部分需要自行编写和添加。

 在 `harmony/rtn-calculator/src/main/cpp` 目录下创建： `CMakeLists.txt`，`CalculatorPacakge.h`，`RTNCalculatorTurboModule.h`，`RTNCalculatorTurboModule.cpp`。

```md
harmony
└── rtn-calculator
    ├── src
    │   └── main
    │       ├── cpp
    │       │   ├── CalculatorPacakge.h
    │       │   ├── CMakeLists.txt
    │       │   ├── RTNCalculatorTurboModule.cpp
    │       │   └── RTNCalculatorTurboModule.h
    │       ├──ets
    │       └── modules.json5         
    ├── build-profile.json5
    ├── hvigorfile.ts
    ├── index.ets
    ├── oh-package.json5
    └── ts.ts
```
<!-- tabs:start -->
#### **CMakeLists.txt**
```cmake
# the minimum version of CMake
cmake_minimum_required(VERSION 3.13)
set(CMAKE_VERBOSE_MAKEFILE on)

file(GLOB rnoh_calculator_SRC CONFIGURE_DEPENDS *.cpp)
add_library(rnoh_calculator SHARED ${rnoh_calculator_SRC})
target_include_directories(rnoh_calculator PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_link_libraries(rnoh_calculator PUBLIC rnoh)
```
<!-- tabs:end -->

<!-- tabs:start -->
#### **RTNCalculatorTurboModule.h**
```cpp
# pragma once
# include "RNOH/ArkTSTurboModule.h"

namespace rnoh {
  class JSI_EXPORT RTNCalculatorTurboModule : public ArkTSTurboModule {
    public:
      RTNCalculatorTurboModule(const ArkTSTurboModule::Context ctx, const std::string name);
  };
} // namespace rnoh
```
<!-- tabs:end -->

<!-- tabs:start -->
#### **RTNCalculatorTurboModule.cpp**
```cpp
#include "RTNCalculatorTurboModule.h"
#include "RNOH/ArkTSTurboModule.h"

using namespace rnoh;
using namespace facebook;

static jsi::Value __hostFunction_RTNCalculatorTurboModule_add(jsi::Runtime &rt, react::TurboModule, const jsi::Value *args, size_t count) {
  return static_cast<ArkTSTurboModule &>(turboModule).callAsync(rt, "add", args, count);
}

RTNCalculatorTurboModule::RTNCalculatorTurboModule(const ArkTSTurboModule::Context ctx, const std::string name) : ArkTSTurboModule(ctx, name) {
  methodMap_["add"] = MethodMetadata{2, __hostFunction_RTNCalculatorTurboModule_add};
}
```
<!-- tabs:end -->


通过 `RNOH/Package.h` 来导出 CalculatorPackage
<!-- tabs:start -->
#### **CalculatorPacakge.h**
```cpp
#include "RNOH/Package.h"
#include "RTNCalculatorTurboModule.h"

using namespace rnoh;
using namespace facebook;
class NativeRTNCalculatorFactoryDelegate : public TurboModuleFactoryDelegate {
  public:
    SharedTurboModule createTurboModule(Context ctx, const std::string &name) const override {
      if (name == "RTNCalculator") {
        return std::make_shared<RTNCalculatorTurboModule>(ctx, name);
      }
      return nullptr;
    }
}

namespace rnoh {
  class CalculatorPackage : public Package {
    public:
      CalculatorPackage(Package::Context ctx) : Package(ctx) {}
      std::unique_ptr<TurboModuleFactoryDelegate> createTurboModuleFactoryDelegate() override {
        return std::make_unique<NativeRTNCalculatorFactoryDelegate>();
      }
  };
} // namespace rnoh
```
<!-- tabs:end -->




### 4. 原生代码

#### Android

Android 平台上 Turbo Native Module 的原生代码需执行如下步骤：

1. 创建用于实现模块的 CalculatorModule.java
2. 修改之前生成的 CalculatorPackage.java

 Android 第三方库目录文件结构应为如下：
 ```md
 android
├── build.gradle
└── src
    └── main
        ├── AndroidManifest.xml
        └── java
            └── com
                └── rtncalculator
                    ├── CalculatorModule.java
                    └── CalculatorPackage.java
 ```

创建 CalculatorModule.java

<!-- tabs:start -->
#### **CalculatorModule.java**
```java
package com.rtncalculator;

import androidx.annotation.NonNull;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.Promise;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import java.util.Map;
import java.util.HashMap;

public class CalculatorModule extends NativeCalculatorSpec {

    public static String NAME = "RTNCalculator";

    CalculatorModule(ReactApplicationContext context) {
        super(context);
    }

    @Override
    @NonNull
    public String getName() {
        return NAME;
    }

    @Override
    public void add(double a, double b, Promise promise) {
        promise.resolve(a + b);
    }
}
```
<!-- tabs:end -->

这个类实现了模块的功能，它继承了 NativeCalculatorSpec 类，而这个类是之前从 JavaScript 接口声明文件 NativeCalculator 自动生成的。

修改 CalculatorPackage.java

<!-- tabs:start -->
#### **CalculatorPackage.java**
```diff
package com.rtncalculator;

import androidx.annotation.Nullable;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
+ import com.facebook.react.module.model.ReactModuleInfo;
import com.facebook.react.module.model.ReactModuleInfoProvider;
import com.facebook.react.TurboReactPackage;

import java.util.Collections;
import java.util.List;
+ import java.util.HashMap;
+ import java.util.Map;

public class CalculatorPackage extends TurboReactPackage {

  @Nullable
  @Override
  public NativeModule getModule(String name, ReactApplicationContext reactContext) {
+      if (name.equals(CalculatorModule.NAME)) {
+          return new CalculatorModule(reactContext);
+      } else {
          return null;
+      }
  }


  @Override
  public ReactModuleInfoProvider getReactModuleInfoProvider() {
-      return null;
+      return () -> {
+          final Map<String, ReactModuleInfo> moduleInfos = new HashMap<>();
+          moduleInfos.put(
+                  CalculatorModule.NAME,
+                  new ReactModuleInfo(
+                          CalculatorModule.NAME,
+                          CalculatorModule.NAME,
+                          false, // canOverrideExistingModule
+                          false, // needsEagerInit
+                          true, // hasConstants
+                          false, // isCxxModule
+                          true // isTurboModule
+          ));
+          return moduleInfos;
+      };
  }
}
```
<!-- tabs:end -->

这就是 Android 平台原生代码的最后一部分，它定义了 TurboReactPackage 对象，这个对象将用于 App 的模块加载。

#### Harmony

Harmony 平台上 Turbo Native Module 的原生代码需执行如下步骤：

1. 创建用于实现模块的 CalculatorModule.ts
2. 创建 CalculatorPackage.ts
3. 创建 index.ets 和 ts.ts
4. 修改 oh-package.json5，hvigorfile.ts，module.json5

 Harmony 第三方库目录文件结构应为如下：
 ```md
harmony
└── rtn-calculator
    ├── src
    │   └── main
    │       ├── cpp
    │       │   ├── CalculatorPacakge.h
    │       │   ├── CMakeLists.txt
    │       │   ├── RTNCalculatorTurboModule.cpp
    │       │   └── RTNCalculatorTurboModule.h
    │       ├──ets
    │       │   ├── CalculatorModule.ts
    │       │   └── CalculatorPackage.ts
    │       └── modules.json5         
    ├── build-profile.json5
    ├── hvigorfile.ts
    ├── index.ets
    ├── oh-package.json5
    └── ts.ts
 ```

创建 `CalculatorModule.ts`

 <!-- tabs:start -->
#### **CalculatorModule.ts**
```ts
import { TurboModule } from 'rnoh/ts';

export class CalculatorModule extends TurboModule {
  add(a: number, b: number): Promise<number> {
    return new Promise(resolve => resolve(a + b));
  }
}
```
<!-- tabs:end -->

这个类实现了模块的功能，它继承了 TurboModule 类，对应 Android 里的 NativeCalculatorSpec。

创建用于实现模块的 `CalculatorModule.ts`

 <!-- tabs:start -->
#### **CalculatorPackage.ts**
```ts
import { RNPackage, TurboModulesFactory } from 'rnoh/ts;
import type { TurboModule, TurboModuleContext } from 'rnoh/ts';
import { CalculatorModule } from './CalculatorModule';

class CalculatorModulesFactory extends TurboModulesFactory {
  createTurboModule(name: string): TurboModule | null {
    if (name === 'RTNCalculator') {
      return new CalculatorModule(this.ctx)
    }
    return null;
  }

  hasTurboModule(name: string): boolean {
    return name === 'RTNCalculator';
  }
}

export class CalculatorPackage extends RNPackage {
  createTurboModulesFactory(ctx: TurboModuleContext): TurboModulesFactory {
    return new CalculatorModulesFactory(ctx);
  }
}
```
<!-- tabs:end -->

这就是 Harmony 平台原生代码的最后一部分，它定义了 RNPackage 对象，这个对象将用于 App 的模块加载。

创建 `ts.ts` 和 `index.ets`

 <!-- tabs:start -->
#### **ts.ts**
```ts
export * from "./src/main/ets/CalculatorPackage"
export * from "./src/main/ets/CalculatorModule"
```
<!-- tabs:end -->

 <!-- tabs:start -->
#### **index.ets**
```ts
export * from './ts'
```
<!-- tabs:end -->

修改 `oh-package.json5`，`hvigorfile.ts`，`module.json5`

 <!-- tabs:start -->
#### **oh-package.json5**
```json
{
  "devDependencies": {
    "rnoh": "file:../rnoh"
  },
  "name": "rnoh-calculator",
  "main": "index.ets",
  "type": "module"
}
```
<!-- tabs:end -->

 <!-- tabs:start -->
#### **hvigorfile.ts**
```ts
export { harTasks } from '@ohos/hvigor-ohos-plugin';
```
<!-- tabs:end -->

 <!-- tabs:start -->
#### **module.json5**
```json
{
  module: {
    name: 'calculator',
    type: 'har',
    deviceType: ['default']
  }
}
```
<!-- tabs:end -->

### 5. 将 Turbo Native Module 添加到 App

#### Shared

首先，需要将包含模块的 NPM 包添加到 App。可以使用以下命令执行此操作：

```bash
cd tester
yarn add ../RTNCalculator
```

此命令会将 RTNCalculator 模块添加到 App 内的 node_modules 目录。

#### Android

在配置 Android 之前，您需要先开启新架构：

1. 打开 android/gradle.properties；
2. 滑到文件底部，将 newArchEnabled 的值从 false 修改为 true。

#### Harmony

!> 待完善能力：Harmony 平台目前暂时不支持 AutoLink，所以需要自行配置。

首先使用 DevEco Studio 打开 React-Native 项目里的鸿蒙工程 `harmony`

##### 引入原生端代码

打开 `entry/oh-package.json5`，添加以下依赖，引入鸿蒙原生端的代码

```json
"dependencies": {
    "rnoh": "file:../rnoh",
    "rnoh-calculator": "file:../../node_modules/RTNCalculator/harmony/rtn-calculator"
  }
```

在终端运行以下命令

```bash
cd entry
ohpm install --no-link
```

##### 配置 CMakeLists 和引入 CalculatorPackge

打开 `entry/src/main/cpp/CMakeLists.txt`，添加：

```diff
project(rnapp)
cmake_minimum_required(VERSION 3.4.1)
set(RNOH_APP_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
set(OH_MODULE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../oh_modules")
set(RNOH_CPP_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../../../../react-native-harmony/harmony/cpp")

add_subdirectory("${RNOH_CPP_DIR}" ./rn)

# RNOH_BEGIN: add_package_subdirectories
add_subdirectory("../../../../sample_package/src/main/cpp" ./sample-package)
+ add_subdirectory("${OH_MODULE_DIR}/rnoh-calculator/src/main/cpp" ./calculator)
# RNOH_END: add_package_subdirectories

add_library(rnoh_app SHARED
    "./PackageProvider.cpp"
    "${RNOH_CPP_DIR}/RNOHAppNapiBridge.cpp"
)

target_link_libraries(rnoh_app PUBLIC rnoh)

# RNOH_BEGIN: link_packages
target_link_libraries(rnoh_app PUBLIC rnoh_sample_package)
+ target_link_libraries(rnoh_app PUBLIC rnoh_calculator)
# RNOH_END: link_packages
```

打开 `entry/src/main/cpp/PackageProvider.cpp`，添加：

```diff
#include "RNOH/PackageProvider.h"
#include "SamplePackage.h"
+ #include "CalculatorPackage.h"

using namespace rnoh;

std::vector<std::shared_ptr<Package>> PackageProvider::getPackages(Package::Context ctx) {
    return {
      std::make_shared<SamplePackage>(ctx),
+     std::make_shared<CalculatorPackage>(ctx)
    };
}
```


##### 在ArkTs侧引入 Calculator TurboModule

打开 `entry/src/main/ets/RNPackageFactory.ts`，添加：

```diff
import {RNPackageContext, RNPackage} from 'rnoh/ts';
import {SamplePackage} from 'rnoh-sample-package/ts';
+ import {CalculatorPackage} from 'rnoh-calculator/ts';

export function createRNPackages(ctx: RNPackageContext): RNPackage[] {
  return [
    new SamplePackage(ctx),
+   new CalculatorPackage(ctx)
    ];
}
```

#### JavaScript

以下是一个在 App.js 中调用 add 方法的例子：

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
import React from 'react';
import {useState} from 'react';
import type {Node} from 'react';
import {
  SafeAreaView,
  StatusBar,
  Text,
  Button,
} from 'react-native';
import RTNCalculator from 'rtn-calculator/js/NativeCalculator.js';

const App: () => Node = () => {
  const [result, setResult] = useState<number | null>(null);
  return (
    <SafeAreaView>
      <StatusBar barStyle={'dark-content'} />
      <Text style={{marginLeft: 20, marginTop: 20}}>
        3+7={result ?? '??'}
      </Text>
      <Button
        title="Compute"
        onPress={async () => {
          const value = await RTNCalculator.add(3, 7);
          setResult(value);
        }}
      />
    </SafeAreaView>
  );
};
export default App;
```
<!-- tabs:end -->

