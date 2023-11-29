# React-Native 三方库

在 React-Native 中，我们使用的组件和 API 主要有两种：

- RN 框架提供的核心组件和 API：如 View、Image；
- 通过 npm 安装的，来自社区三方库里的组件和 API：如 FastImage、SVG。

React Native 有一个庞大的社区，当核心组件和 API 不能满足需求时，开发者可以到社区中寻求合适的第三方库。一般来说，一个 RN APP 通常都会使用到各种各样的三方库。

## 三方库组成

React-Naitve 三方库主要由 JS 实现和原生实现两部分组成。通常安卓端的原生实现位于 `Android` 文件夹下，iOS 的原生实现位于 `ios` 文件夹下，JS 端的代码位于 `src` 文件夹或根目录下。

## 三方库分类

根据RN三方库的实现方式可分为：原生库，JS库，纯JS库（命名为内部自行命名），根据不同的分类会有不同的移植策略。

### 原生库

这类库通过原生代码实现，通常使用 Java或kotlin（Android）和 Objective-C 或 Swift（iOS）编写。原生模块提供了与底层平台直接交互的能力，可以实现对底层功能的更深度的控制。如[react-native-pager-view](https://github.com/callstack/react-native-pager-view)、[progress-bar-android](https://github.com/react-native-progress-view/progress-bar-android)和[react-native-slider](https://github.com/callstack/react-native-slider/tree/main)。

**特征**

源码项目结构上通常在根目录或者package目录下有`android`或`ios`或`apple`目录，如下图：

```
project-root/
|-- android/          # Android 应用源码和配置文件
|-- ios/              # iOS 应用源码和配置文件

// 或者
project-root/
|-- package/          
|   |-- android/          # Android 应用源码和配置文件
|   |-- ios/              # iOS 应用源码和配置文件
```

**移植**

这类库需要参考原库逻辑来移植，有着如下移植点：

- RN JS侧适配新架构
- RN JS侧适配harmony平台
- 利用codegen搭建Bridge的通信通道（开发相对固定）
- 使用arkts复刻原库逻辑实现原库效果

### JS库

这类库专门为 React Native 开发，但不涉及原生实现，利用 React Native 提供的桥接机制和 API 来实现其功能，或者通过其他三方库实现二次封装的功能。如[recyclerlistview](https://github.com/Flipkart/recyclerlistview)、[react-navigation](https://github.com/react-navigation/react-navigation)和[react-native-qrcode-svg](https://github.com/awesomejerry/react-native-qrcode-svg)

**特征**

源码项目结构上不包含原生相关目录（android/ios/apple）,且package.json中对react-native有依赖。

```
package.json

// 有如下任意一种依赖即可
"dependencies": {
	"react": "",
	"react-native": "*"
}
"peerDependencies": {
	"react": "*",
	"react-native": "*"
}
```

**移植**

由于是基于RN接口的封装，所以涉及较少的开发工作，移植点包含：

- RN JS侧适配harmony平台
- 大量测试对比

### 纯JS库

这类库是纯粹的 JavaScript 模块，不依赖 React Native 平台的特定功能，可以在多种 JavaScript 环境中运行。如[lodash](https://github.com/lodash/lodash)、[deepmerge](https://github.com/TehShrike/deepmerge)和[styled-system](https://github.com/styled-system/styled-system)

**特征**

源码项目结构上不包含原生相关目录（android/ios/apple）,且package.json中对react-native无依赖。

**移植**

由于是基于RN接口的封装，所以没有开发工作，主要是测试对应各个平台效果是否一致，不一致则需要反馈给框架层。因为使用的RN引擎是一致的，所以通常来说接口都是正常可用的。

## 三方库使用

对于 Android&iOS 平台，通过 npm 或 yarn 来安装三方库即可

## Gitee Pages
