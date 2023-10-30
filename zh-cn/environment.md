# 搭建开发环境

这篇文档会帮助你搭建基本的 React-Native 开发环境。

## 安装node环境

我们需要安装`npm`包管理器，而安装了`node.js`就会自动安装`npm`

### 安装node

官网下载安装程序，双击下载的 exe 安装，下一步下一步直到完成。

官网地址：<https://nodejs.org/en/>

### 验证安装

输入 node -v，npm -v 输出版本就是安装成功了

```
#验证node
node -v

#验证npm
npm -v
```

### Yarn

这里更推荐 Yarn 来作为包管理工具

Yarn是 Facebook 提供的替代 npm 的工具，可以加速 node 模块的下载。
```
npm install -g yarn
```

安装完 yarn 之后就可以用 yarn 代替 npm 了，例如用 `yarn` 代替 `npm install` 命令，用 `yarn add 某第三方库名` 代替 `npm install 某第三方库名`。

## 搭建 Android 环境

我们需要搭建 Android 和 Harmony 两套环境。安卓环境主要用于效果比对和RN Demo的开发。

!> 请先参考官方的 React-Naitve + Android 的环境搭建文档 [React Native Step Up](https://www.reactnative.cn/docs/environment-setup)，将 Android 环境搭建好，并成功运行 React-Native 官方给定的 demo 后再进行下一步。

## 搭建 Harmony 环境

### IDE和手机版本

Harmony 环境需要注意IDE版本、OpenHarmony SDK版本和手机版本是否符合要求。

- 2023.10.30：

DevEco Studio 版本：4.0.3.601

OpenHarmony(API10): 4.0.10.11

工程机版本：NOH-AN00 204.0.0.65(SP4C00E70R1P12)

### 拉取 RNOH 代码

RNOH 包含的内容：

1. react-native-harmony：react-native 的 Hamrony 拓展；
2. react-native-harmony-cli：react-native-cli 的 Hamrony 拓展；
3. tester：配置好 Harmony 支持的 React-Native 项目；

- 绿区：

拉取[ReactNative_OpenHarmony](https://codehub-g.huawei.com/l00496999/ReactNative_OpenHarmony/home)项目

- 蓝区：

拉取[rnoh](https://github.com/react-native-openharmony/rnoh)项目

按需选择分支：

`master` 是最新的RNOH主干分支，包含了 RNOH 的核心SDK；

`third_party` 是在 `master` 的基础上，加入了 React-Native 三方库。

## 创建新项目

!> 待完善能力：目前 harmony 还不支持新建项目的工具，需要手动引入，可跳过此步直接使用 RNOH 的 tester 项目。

使用 React Native 内建的命令行工具来创建一个名为 "AwesomeProject" 的新项目。这个命令行工具不需要安装，可以直接用 node 自带的npx命令来使用：

!> **必须要看的注意事项一**：请不要在目录、文件名中使用中文、空格等特殊符号。请不要单独使用常见的关键字作为项目名（如 class, native, new, package 等等）。请不要使用与核心模块同名的项目名（如 react, react-native 等）。

!> **必须要看的注意事项二**：请不要在某些权限敏感的目录例如 System32 目录中 init 项目！会有各种权限限制导致不能运行！

!> **必须要看的注意事项三**：请不要使用一些移植的终端环境，例如git bash或mingw等等，这些在 windows 下可能导致找不到环境变量。请使用系统自带的命令行（CMD 或 powershell）运行。

```bash
npx react-native@0.72.5 init AwesomeProject --version 0.72.5
```
### Android

创建工具会自动为 Android 和 iOS 生成工程底座，可直接使用。

### Harmony

如果要使用上述创建的 AwesomeProject，需要自行引入一个 harmony 工程（一般不推荐 Harmony 平台自行新建项目，可以直接使用 RNOH 提供的示例项目 `tester`。）

## 编译并运行 React Native 应用

### Android

确保你先运行了模拟器或者连接了真机，然后在你的项目目录中运行 yarn android 或者 yarn react-native run-android：

```bash
cd tester
yarn android
# 或者
yarn react-native run-android
```

### Harmony

- 安装所需依赖：

在 `tester` 目录下，运行：
```sh
npm run preinstall
npm i
```

打开 DevEco Studio，点击右上方出现的 "Sync"

- 打包：

在 `tester` 目录下打包生成bundle，运行：
```sh
npm run dev
```

生成的bundle会放在 `tester/harmony/entry/src/main/resources/rawfile` 目录下。

- 运行

打开 DevEco Studio，Build并运行entry模块

## 常见问题





