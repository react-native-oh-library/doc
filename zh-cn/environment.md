# 搭建开发环境

这篇文档会帮助你搭建基本的React-Native开发环境。

## 安装node环境

我们需要安装`npm`包管理器，而安装了`node.js`就会自动安装`npm`

### 安装node

官网下载安装程序，双击下载的exe安装，下一步下一步直到完成。

官网地址：<https://nodejs.org/en/>

### 验证安装

输入node -v，npm -v输出版本就是安装成功了

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

!> 请先参考官方的 RN+Android 的环境搭建文档[React Native Step Up](https://www.reactnative.cn/docs/environment-setup)，将 Android 环境搭建好，并成功运行RN官方给定的demo后再进行下一步。

## 搭建 Harmony 环境

### IDE和手机版本

Harmony 环境需要注意IDE版本和手机版本是否符合要求。

- 2023.10.07：

DevEco Studio 版本：4.0.3.501

工程机版本：NOH-ANOO 204.0.0.65(SP1C00E67R1P12)

### 拉取代码

- 绿区：

拉取[ReactNative_OpenHarmony](https://codehub-g.huawei.com/l00496999/ReactNative_OpenHarmony/home)项目中最新的 `third_party` 分支。截止2023.10.07，最新的分支为 `730_third_party`。

- 蓝区：

拉取[rnoh](https://github.com/react-native-openharmony/rnoh)项目中的 `third_party` 分支。

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





