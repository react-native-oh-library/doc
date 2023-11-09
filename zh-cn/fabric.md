# Fabric 组件

Fabric 组件是一种使用 Fabric 渲染器渲染并展示在屏幕上的 UI 组件。

![fabric](../img/fabric.png)

在开发 Fabric 组件前，需要先创建一个 JavaScript 接口描述文件。之后 Codegen 会根据这个文件创建一些 C++ 脚手架代码，用于将部分组件逻辑（比如调用原生平台接口能力）与 React Native 结合起来。C++ 代码在各个平台都是一样的，只要组件能够与生成的 C++ 代码连接起来，就可以导入到 App 并运行。

因为 Harmony 平台暂时还没有 codegen 工具，所以我们需要使用 Android 平台的 codegen 来生成相关的 C++ 代码，然后复制到 Harmony 平台使用。

## 如何创建 Fabric 组件

若要创建一个 Fabric 组件，需要遵循以下步骤：

1. 声明 JavaScript 接口；
2. 配置组件以用于 Codegen 生成统一代码，生成的代码可添加为 App 的依赖；
3. 编写所需的原生代码。

接下来会创建一个简单的名为 `RTNCenteredText` 的 Fabric 组件作为示例。

## 目录配置

同样的，我们按照一般的三方库目录结构来配置:

```
.
└── RTNCenteredText
    ├── android（Android 的原生实现代码）
    ├── ios（iOS 的原生实现代码）
    ├── harmony（Harmony 的原生实现代码）
    └── src （js/ts代码）
```

### 1. 声明 JavaScript 接口

新架构要求必须使用强类型风格语言声明 JavaScript 接口（Flow 和 TypeScript 皆可）。Codegen 会根据这些接口声明来生成强类型的语言，其中包括 C++、Objective-C 和 Java。

对于声明类型的代码文件必须满足以下两点要求：

文件必须使用 `<MODULE_NAME>NativeComponent` 命名，在使用 Flow 时，以 `.js` 或 `.jsx` 为后缀名；在使用 Typescript 时，以 `.ts` 或 `.tsx` 为后缀名。Codegen 只会找到匹配这些命名规则的文件；
代码中必须要输出 HostComponent 对象。
以下是使用 Flow 和 TypeScript 声明的 RTNCenteredText 组件。在 `js` 目录中，创建一个命名为 `RTNCenteredText` 并带有相应后缀名的文件。

<!-- tabs:start -->

#### **flow**

RTNCenteredTextNativeComponent.js
```js
// @flow strict-local

import type {ViewProps} from 'react-native/Libraries/Components/View/ViewPropTypes';
import type {HostComponent} from 'react-native';
import codegenNativeComponent from 'react-native/Libraries/Utilities/codegenNativeComponent';

type NativeProps = $ReadOnly<{|
  ...ViewProps,
  text: ?string,
  // add other props here
|}>;

export default (codegenNativeComponent<NativeProps>(
   'RTNCenteredText',
): HostComponent<NativeProps>);
```

#### **typescript**

RTNCenteredTextNativeComponent.ts
```ts
import type {ViewProps} from 'ViewPropTypes';
import type {HostComponent} from 'react-native';
import codegenNativeComponent from 'react-native/Libraries/Utilities/codegenNativeComponent';

export interface NativeProps extends ViewProps {
  text?: string;
  // 添加其它 props
}

export default codegenNativeComponent<NativeProps>(
  'RTNCenteredText',
) as HostComponent<NativeProps>;
```
<!-- tabs:end -->

在声明文件的顶部导入了一些内容。以下是开发 Fabric 组件必须要导入的内容：

- `HostComponent` 类型: 导出的组件需要与这个类型保持一致；
- `codegenNativeComponent` 函数：负责将组件注册到 JavaScript 运行时。
声明文件的中间部分包含了组件的 props。Props（"properties" 的缩写）是用于自定义 React 组件的参数信息。在本例中，需要控制组件的 text 属性。

在声明文件的最后部分，导出了泛型函数 `codegenNativeComponent` 的返回值，此函数需要传递组件的名称。

### 2. Codegen 配置

#### Shared

shared 是 package.json 文件中的一个配置项，它将在 yarn 安装模块时被调用。请在 `RTNCenteredText` 的根目录创建 `package.json` 文件。

```json
{
  "name": "rtn-centered-text",
  "version": "0.0.1",
  "description": "Showcase a Fabric component with a centered text",
  "react-native": "js/index",
  "source": "js/index",
  "files": [
    "js",
    "android",
    "ios",
    "harmony",
    "rtn-centered-text.podspec",
    "!android/build",
    "!ios/build",
    "!**/__tests__",
    "!**/__fixtures__",
    "!**/__mocks__"
  ],
  "keywords": ["react-native", "ios", "android", "harmony"],
  "repository": "https://github.com/<your_github_handle>/rtn-centered-text",
  "author": "<Your Name> <your_email@your_provider.com> (https://github.com/<your_github_handle>)",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/<your_github_handle>/rtn-centered-text/issues"
  },
  "homepage": "https://github.com/<your_github_handle>/rtn-centered-text#readme",
  "devDependencies": {},
  "peerDependencies": {
    "react": "*",
    "react-native": "*"
  },
  "codegenConfig": {
    "name": "RTNCenteredTextSpecs",
    "type": "components",
    "jsSrcsDir": "js"
  }
}
```

将 Codegen 的配置声明到 codegenConfig 字段。codegenConfig 是一个用于存放要生成的第三方库的对象数组，每个对象又包含其它三个字段：

- name：第三方库的名称。按照惯例，名称应以 Spec 为结尾
- type：在这个 npm 包里的模块类型。在本例中，我们开发的是 Turbo Native Module，所以值为 modules
- jsSrcsDir：用于找到 js 接口声明文件的相对路径，它将被 Codegen 解析

#### Android

若要在 Android 平台运行 Codegen，需要创建三个文件：

1. 带有 Codegen 配置信息的 build.gradle 文件
2. AndroidManifest.xml
3. 一个实现 ReactPackage 接口的 Java 类

在文件创建完成后，`android` 目录文件结构应该是这样的：

```md
android
├── build.gradle
└── src
    └── main
        ├── AndroidManifest.xml
        └── java
            └── com
                └── rtncenteredtext
                    └── RTNCenteredTextPackage.java
```

首先，在 `android` 目录创建 `build.gradle` 文件，并配置以下内容：

**build.gradle**

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

  defaultConfig {
    minSdkVersion safeExtGet('minSdkVersion', 21)
    targetSdkVersion safeExtGet('targetSdkVersion', 31)
    buildConfigField("boolean", "IS_NEW_ARCHITECTURE_ENABLED", "true")
  }
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
          package="com.rtncenteredtext">
</manifest>
```
<!-- tabs:end -->

这个 manifest 文件的用途是声明您开发的模块的 Java 包

最后，您需要一个继承 TurboReactPackage 接口的类。在运行 Codegen 前，您不用完整实现这个类。对于 App 而言，一个没有实现接口的空类就已经能当做一个 React Native 依赖，Codegen 会尝试生成其脚手架代码。

创建 `android/src/main/java/com/rtncenteredtext` 目录，在这个目录内创建 `RTNCenteredTextPackage.java` 文件

**RTNCenteredTextPackage.java**

<!-- tabs:start -->
#### **RTNCenteredTextPackage.java**
```java
package com.rtncenteredtext;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.Collections;
import java.util.List;

public class RTNCenteredTextPackage implements ReactPackage {

    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }

    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }

}
```
<!-- tabs:end -->

ReactPackage 接口的用途是让 React Native 为使用 App 中的 ViewManager 和 Native Modules，识别出哪些原生类需要在第三方库里导出。

Codegen 会在 App 编译的时候自动运行。


#### Harmony

Harmony 平台暂时还没有 Codegen，所以我们需要手动运行 Android 的 Codegen，然后把生成的代码复制过来使用。

!> 请务必先把 Android 的 Codegen 配置好再执行以下操作

首先我们需要一个 React-Native App来执行 Codegen，假设 App 的目录是和 当前目录平级的 `MyApp`，执行以下命令来创建一个 Gradle 任务来执行 Codegen。

!> 在运行 Codegen 之前，您需要在 Android 中的 App 启动新架构。您可以通过修改 gradle.properties 文件中的 newArchEnabled 属性，将 false 改为 true。

```bash
cd MyApp
yarn add ../RTNCenteredText
cd android
./gradlew generateCodegenArtifactsFromSchema
```

生成后的代码保存在 `MyApp/node_modules/rtn-centered-text/android/build/generated/source/codegen` 目录，并呈以下结构：

```md
codegen
├── java
│   └── com
│       └── facebook
│           └── react
│               └── viewmanagers
│                   ├── RTNCenteredTextManagerDelegate.java
│                   └── RTNCenteredTextManagerInterface.java
├── jni
│   ├── Android.mk
│   ├── CMakeLists.txt
│   ├── RTNCenteredText-generated.cpp
│   ├── RTNCenteredText.h
│   └── react
│       └── renderer
│           └── components
│               └── RTNCenteredText
│                   ├── ComponentDescriptors.h
│                   ├── EventEmitters.cpp
│                   ├── EventEmitters.h
│                   ├── Props.cpp
│                   ├── Props.h
│                   ├── ShadowNodes.cpp
│                   └── ShadowNodes.h
└── schema.json
```

`codegen/jni/react/renderer/components/RTNCenteredText` 目录下的代码是 Harmony 需要的。将这些代码复制到 `harmony/rtn-centered-text/src/main/cpp` 文件夹下，并修改一下各文件 "include" 的路径。

如 `ComponentDescriptor.h`

```diff
...
+ #include "ShadowNodes.h"
#include <react/renderer/core/ConcreteComponentDescriptor.h>
...
```

然后在同级目录创建 `CenteredTextJSIBinder.h`, `CenteredTextNapiBinder.h`, `CenteredTextPackage.h` 等三个文件，目录结构如下

```md
harmony
└── rtn-centered-text
    ├── src
    │   └── main
    │       ├── cpp
    │       │   ├── ComponentDescriptors.h
    │       │   ├── EventEmitters.cpp
    │       │   ├── EventEmitters.h
    │       │   ├── Props.cpp
    │       │   ├── Props.h
    │       │   ├── ShadowNodes.cpp
    │       │   ├── ShadowNodes.h
    │       │   ├── CenteredTextJSIBinder.h
    │       │   ├── CenteredTextNapiBinder.h
    │       │   └── CenteredTextPackage.h
    │       ├──ets
    │       └── modules.json5         
    ├── build-profile.json5
    ├── hvigorfile.ts
    ├── index.ets
    ├── oh-package.json5
    └── ts.ts
```

**CenteredTextJSIBinder.h**

<!-- tabs:start -->
#### **CenteredTextJSIBinder.h**
```cpp
#include "RNOHCorePackage/ComponentBinders/ViewComponentJSIBinder.h"

namespace rnoh {

class CenteredTextJSIBinder : public ViewComponentJSIBinder {
    facebook::jsi::Object createNativeProps(facebook::jsi::Runtime &rt) override {
        auto object = ViewComponentJSIBinder::createNativeProps(rt);
        object.setProperty(rt, "text", "string");
        return object;
    }
};
} // namespace rnoh
```
<!-- tabs:end -->

JSI Binder 的作用是桥接 JS 和 C++，将属性从 JS 端传递到 C++ 端。

**CenteredTextNapiBinder.h**

<!-- tabs:start -->
#### **CenteredTextNapiBinder.h**
```cpp
#include "RNOHCorePackage/ComponentBinders/ViewComponentNapiBinder.h"
#include "Props.h"

namespace rnoh {

class CenteredTextNapiBinder : public ViewComponentNapiBinder {
public:
    napi_value createProps(napi_env env, facebook::react::ShadowView const shadowView) override {
        napi_value napiViewProps = ViewComponentNapiBinder::createProps(env, shadowView);
        if (auto props = std::dynamic_pointer_cast<const facebook::react::RNCSliderProps>(shadowView.props)) {
            return ArkJS(env)
                .getObjectBuilder(napiViewProps)
                .addProperty("text", props->text)
                .build();
        }
        return napiViewProps;
    };
};
} //namespace rnoh
```
<!-- tabs:end -->

Napi Binder的作用是桥接 C++ 和 ArkTs ，将属性从 C++ 端传递到 ArkTs 端。

**CenteredTextPackage.h**

<!-- tabs:start -->
#### **CenteredTextPackage.h**
```cpp
#include "RNOH/Package.h"
#include "ComponentDescriptor.h"
#include "CenteredTextJSIBinder.h"
#include "CenteredTextNapiBinder.h"

namespace rnoh {

class CenteredTextPackage : public Package {
public:
    CenteredTextPackage(Package::Context ctx): Package(ctx) {}

    std::vector<facebook::react::ComponentDescriptorProvider> createComponentDescriptorProviders() override {
        return {facebook::react::concreteComponentDescriptorProvider<facebook::react::RTNCenteredTextComponentDescriptor>()};
    }

    ComponentNapiBinderByString createComponentNapiBinderByName() override {
        return {{"RTNCenteredText", std::make_shared<CenteredTextNapiBinder>()}};
    }

    ComponentJSIBinderByString createComponentJSIBinderByName() override {
        return {{"RTNCenteredText", std::make_shared<CenteredTextJSIBinder>()}};
    }
};
} // namespace rnoh
```
<!-- tabs:end -->

Package 接口的用途是让 React-Native 识别出三方库需要导出哪些 C++ 接口。

### 3. 原生代码

#### Android

Android 平台中 Fabric 组件的原生代码必须包含以下三个部分：

1. RTNCenteredText.java 用于渲染原生视图
2. RTNCenteredTextManager.java 用于实例化原生视图
3. 在 RTNCenteredTextPackage.java 实现具体的逻辑代码

Android 第三方库目录文件结构应为如下：

```md
android
├── build.gradle
└── src
    └── main
        ├── AndroidManifest.xml
        └── java
            └── com
                └── rtncenteredtext
                    ├── RTNCenteredText.java
                    ├── RTNCenteredTextManager.java
                    └── RTNCenteredTextPackage.java
```

**RTNCenteredText.java**

<!-- tabs:start -->
#### **RTNCenteredText.java**
```java
package com.rtncenteredtext;

import androidx.annotation.Nullable;
import android.content.Context;
import android.util.AttributeSet;
import android.graphics.Color;

import android.widget.TextView;
import android.view.Gravity;

public class RTNCenteredText extends TextView {

    public RTNCenteredText(Context context) {
        super(context);
        this.configureComponent();
    }

    public RTNCenteredText(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        this.configureComponent();
    }

    public RTNCenteredText(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        this.configureComponent();
    }

    private void configureComponent() {
        this.setBackgroundColor(Color.RED);
        this.setGravity(Gravity.CENTER_HORIZONTAL);
    }
}
```
<!-- tabs:end -->

这个类表示的是原生视图，将由 Android 渲染到屏幕上。它继承了 TextView 并且调用私有方法 configureComponent() 来配置自身的基本参数。

**RTNCenteredTextManager.java**

<!-- tabs:start -->
#### **RTNCenteredTextManager.java**
```java
package com.rtncenteredtext;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;

import com.facebook.react.bridge.ReadableArray;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.module.annotations.ReactModule;
import com.facebook.react.uimanager.SimpleViewManager;
import com.facebook.react.uimanager.ThemedReactContext;
import com.facebook.react.uimanager.ViewManagerDelegate;
import com.facebook.react.uimanager.annotations.ReactProp;
import com.facebook.react.viewmanagers.RTNCenteredTextManagerInterface;
import com.facebook.react.viewmanagers.RTNCenteredTextManagerDelegate;


@ReactModule(name = RTNCenteredTextManager.NAME)
public class RTNCenteredTextManager extends SimpleViewManager<RTNCenteredText>
        implements RTNCenteredTextManagerInterface<RTNCenteredText> {

    private final ViewManagerDelegate<RTNCenteredText> mDelegate;

    static final String NAME = "RTNCenteredText";

    public RTNCenteredTextManager(ReactApplicationContext context) {
        mDelegate = new RTNCenteredTextManagerDelegate<>(this);
    }

    @Nullable
    @Override
    protected ViewManagerDelegate<RTNCenteredText> getDelegate() {
        return mDelegate;
    }

    @NonNull
    @Override
    public String getName() {
        return RTNCenteredTextManager.NAME;
    }

    @NonNull
    @Override
    protected RTNCenteredText createViewInstance(@NonNull ThemedReactContext context) {
        return new RTNCenteredText(context);
    }

    @Override
    @ReactProp(name = "text")
    public void setText(RTNCenteredText view, @Nullable String text) {
        view.setText(text);
    }
}
```
<!-- tabs:end -->

RTNCenteredTextManager 类用于让 React Native 实例化原生组件，它实现了由 Codegen 生成的接口（见 implements 语句的 RTNCenteredTextManagerInterface 接口）并使用了 RTNCenteredTextManagerDelegate 类。

它同时负责导出所有 React Native 所需的内容，例如使用 @ReactModule 注解的 RTNCenteredTextManager 类，及使用 @ReactProp 注解的 setText 方法。

**RTNCenteredText.java**

最后，打开 `android/src/main/java/com/rtncenteredtext` 目录的 `RTNCenteredTextPackage.java`，并进行以下修改：

<!-- tabs:start -->
#### **RTNCenteredTextPackage update**
```diff
package com.rtncenteredtext;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.Collections;
import java.util.List;

public class RTNCenteredTextPackage implements ReactPackage {

    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
+        return Collections.singletonList(new RTNCenteredTextManager(reactContext));;
    }

    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }

}
```
<!-- tabs:end -->

新增的代码实例化了一个 RTNCenteredTextManager 对象，用于让 React Natve 运行时渲染 Fabric 组件。

#### Harmony

Harmony 平台中 Fabric 组件的原生代码必须包含以下三个部分：

1. 创建用于实现组件的 RTNCenteredText.ets
3. 创建 index.ets
4. 修改 oh-package.json5，hvigorfile.ts，module.json5

 Harmony 第三方库目录文件结构应为如下：

 ```md
harmony
└── rtn-centered-text
    ├── src
    │   └── main
    │       ├── cpp
    │       │   ├── ComponentDescriptors.h
    │       │   ├── EventEmitters.cpp
    │       │   ├── EventEmitters.h
    │       │   ├── Props.cpp
    │       │   ├── Props.h
    │       │   ├── ShadowNodes.cpp
    │       │   ├── ShadowNodes.h
    │       │   ├── CenteredTextJSIBinder.h
    │       │   ├── CenteredTextNapiBinder.h
    │       │   └── CenteredTextPackage.h
    │       ├──ets
    │       │   └── RTNCenteredText.ets
    │       └── modules.json5         
    ├── build-profile.json5
    ├── hvigorfile.ts
    ├── oh-package.json5
    └── index.ets
 ```

 **RTNCenteredText.ets**

<!-- tabs:start -->
####  **RTNCenteredText.ets**
```ts
import { Descriptor, ComponentBuilderContext, ViewBaseProps, Tag } from 'rnoh';
import { RNComponentFactory, RNOHContext, RNViewBase } from 'rnoh'

export const CENTERED_TEXT_TYPE: string = "RTNCenteredText"

export type CenteredTextProps = ViewBaseProps & {
  text: string
}

export type CenteredTextDescriptor = Descriptor<"RTNCenteredText", ViewBaseProps>


@Component
export struct RTNCenteredText {
  ctx!: RNOHContext
  tag: number = 0
  @BuilderParam buildCustomComponent: (componentBuilderContext: ComponentBuilderContext) => void
  @State descriptor: CenteredTextDescriptor = {} as CenteredTextDescriptor
  private unregisterDescriptorChangesListener?: () => void = undefined

  aboutToAppear() {
    this.descriptor = this.ctx.descriptorRegistry.getDescriptor<CenteredTextDescriptor>(this.tag)
    this.unregisterDescriptorChangesListener = this.ctx.descriptorRegistry.subscribeToDescriptorChanges(this.tag,
      (newDescriptor) => {
        this.descriptor = (newDescriptor as CenteredTextDescriptor)
      }
    )
  }

  aboutToDisappear() {
    this.unregisterDescriptorChangesListener?.()
  }

  build() {
    RNViewBase({ ctx: this.ctx, tag: this.tag }) {
      Text(this.descriptor.props.text)
      .fontColor("red")
      .fontSize(12)
      .textAlign(TextAlign.Center)
      .width("100%")
      .height("100%")
    }
  }
}
```
<!-- tabs:end -->

该部分是 RTNCenteredText 的 Harmony 原生实现。

创建 `index.ets`

 <!-- tabs:start -->
#### **index.ets**
```ts
export * from './src/main/ets/RTNCenteredText'
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
  "name": "rnoh-centered-text",
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
    name: 'centered_text',
    type: 'har',
    deviceType: ['default']
  }
}
```
<!-- tabs:end -->

### 5. 将 Fabric 组件添加到 App

#### Shared

首先，需要将包含模块的 NPM 包添加到 App。可以使用以下命令执行此操作：

```bash
cd tester
yarn add ../RTNCenteredText
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
    "rnoh-centered-text": "file:../../node_modules/RTNCenteredText/harmony/rtn-centered-text"
  }
```

在终端运行以下命令

```bash
cd entry
ohpm install --no-link
```

##### 配置 CMakeLists 和引入 CenteredTextPackge

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
+ add_subdirectory("${OH_MODULE_DIR}/rnoh-centered-text/src/main/cpp" ./centered-text)
# RNOH_END: add_package_subdirectories

add_library(rnoh_app SHARED
    "./PackageProvider.cpp"
    "${RNOH_CPP_DIR}/RNOHAppNapiBridge.cpp"
)

target_link_libraries(rnoh_app PUBLIC rnoh)

# RNOH_BEGIN: link_packages
target_link_libraries(rnoh_app PUBLIC rnoh_sample_package)
+ target_link_libraries(rnoh_app PUBLIC rnoh_centered_text)
# RNOH_END: link_packages
```

打开 `entry/src/main/cpp/PackageProvider.cpp`，添加：

```diff
#include "RNOH/PackageProvider.h"
#include "SamplePackage.h"
+ #include "CenteredTextPackage.h"

using namespace rnoh;

std::vector<std::shared_ptr<Package>> PackageProvider::getPackages(Package::Context ctx) {
    return {
      std::make_shared<SamplePackage>(ctx),
+     std::make_shared<CenteredTextPackage>(ctx)
    };
}
```


##### 在ArkTs侧引入 CenteredText

打开 `entry/src/main/ets/pages/Index.ets`，添加：

```diff
import { ComponentBuilderContext } from 'rnoh';
import { RNApp, RNAbility, AnyJSBundleProvider, MetroJSBundleProvider, ResourceJSBundleProvider } from 'rnoh'
import { createRNPackages } from '../RNPackagesFactory'
import { SampleView, SAMPLE_VIEW_TYPE, PropsDisplayer } from "rnoh-sample-package"
import { RTNCenteredText, CENTERED_TEXT_TYPE } from "rnoh-centered-text"


@Builder
function CustomComponentBuilder(ctx: ComponentBuilderContext) {
  if (ctx.descriptor.type === SAMPLE_VIEW_TYPE) {
    SampleView({
      ctx: ctx.rnohContext,
      tag: ctx.descriptor.tag,
      buildCustomComponent: CustomComponentBuilder
    })
  } else if (ctx.descriptor.type === PropsDisplayer.NAME) {
    PropsDisplayer({
      ctx: ctx.rnohContext,
      tag: ctx.descriptor.tag
    })
  } else if (ctx.descriptor.type === CENTERED_TEXT_TYPE) {
    RTNCenteredText({
      ctx: ctx.rnohContext,
      tag: ctx.descriptor.tag
    })
  }
}

@Entry
@Component
struct Index {
  @StorageLink('RNAbility') rnAbility: RNAbility | undefined = undefined
  @State shouldShow: boolean = false

  aboutToAppear() {
    setTimeout(() => {
      // Debugger don't work from the get-go, hence this artificial delay.
      this.shouldShow = true
    }, 1000)
  }

  onBackPress(): boolean | undefined {
    // NOTE: this is required since `Ability`'s `onBackPressed` function always
    // terminates or puts the app in the background, but we want Ark to ignore it completely
    // when handled by RN
    return this.rnAbility?.onBackPress();
  }

  build() {
    Column() {
      if (this.rnAbility && this.shouldShow) {
        RNApp({
          rnInstanceConfig: { createRNPackages },
          initialProps: { "foo": "bar" } as Record<string, string>,
          appKey: "app_name",
          buildCustomComponent: CustomComponentBuilder,
          jsBundleProvider: new AnyJSBundleProvider([
            new MetroJSBundleProvider(),
            new ResourceJSBundleProvider(this.rnAbility.context.resourceManager, 'hermes_bundle.hbc'),
            new ResourceJSBundleProvider(this.rnAbility.context.resourceManager, 'bundle.harmony.js')]),
        })
      }
    }
    .height('100%')
    .width('100%')
  }
}
```

#### JavaScript

最后，操作以下步骤，您就可以在 JavaScript 调用组件了。

1. 在 js 文件中导入组件。假设要在 App.js 进行导入，需要添加这行代码：

```js
import RTNCenteredText from 'rtn-centered-text/js/RTNCenteredTextNativeComponent';
```

2. 接下来，在 React Native 组件里进行调用。调用的语法和其它组件相同：

**App.js**

<!-- tabs:start -->
#### **App.js**
```js
// ... other code
const App: () => Node = () => {
  // ... other App code ...
  return (
    // ...other React Native elements...
    <RTNCenteredText
      text="Hello World!"
      style={{width: '100%', height: 30}}
    />
    // ...other React Native Elements
  );
};
```
<!-- tabs:end -->