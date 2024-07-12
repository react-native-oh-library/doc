> [!WARNING] 本文档为非公开文档，仅用于三方库使用和开发指导，不涉及任何 React Native OpenHarmony 框架的信息，且会随着 React Native OpenHarmony 框架持续迭代更新，当前版本不代表最终展示版本。

# 本地打包三方库

打包的目的是获得 `.tgz` 压缩包，让开发者能在本地通过 npm/yarn 引入三方库依赖。

## 打包配置

> [!WARNING] 打包前请先配置好 bob-build（bob-build 不是必须项，但这里只介绍使用了 bob-build 的打包形式，其他形式请自行查阅资料）

一般来说，`src` 里的 JS 代码是使用 typescript 语言的话，都需要用 bob-build。如果是 Flow 的话，则需要看是否有 index.d.ts 文件，若没有也需要使用。

### 配置 bob-build

> [!tip] 若源库的 JS 代码并没有改动或原本就是新架构，则可以跳过

`react-native-builder-bob`可以在 React Native 库中为不同目标构建代码，其中鸿蒙需要构建 CommonJS 代码，生成的代码会存储到 `lib`。

打开 `package.json` 添加

<!-- tabs:start -->

#### **package.json**

```json
    ...,
    // 将JS入口改为bob-build生成的内容
    "main": "lib/commonjs/index",
    "module": "lib/module/index",
    "typings": "lib/typescript/index.d.ts",
    "react-native": "src/index",
    "source": "src/index",
    // 添加bob到prepack step
    "scripts": {
        ...
        "bob": "bob build",
        "prepare": "npm run bob"
    },
    // 添加依赖
    "devDependencies": {
        ...,
        "react-native-builder-bob": "^0.20.4"
    },
    // 添加构建目标
    "react-native-builder-bob": {
        "source": "src",
        "output": "lib",
        "targets": [
            "commonjs",
            "module",
            [
                "typescript",
                {
                    "project": "tsconfig.build.json"  // 若有
                }
            ]
        ]
    },
       ...
```

<!-- tabs:end -->

### 配置 `package.json`

<!-- tabs:start -->

#### **package.json**

```json
{
    ...,
    "version": "x.x.x", // 发布前请确认好包的版本
    "files": [
        // 把tgz包里要包含的文件和文件夹添加到这里面，下面是示例
        android,
        ios,
        harmony,
        lib,
        src
    ],
    ...
}
```

<!-- tabs:end -->

#### **tsconfig.build.json**

把 harmony 添加到exclude中，否则执行bob-build会将其中的ts/ets编译为js，编译失败会报错，导致bob失败。例如harmony/**/hvigorfile.ts

```json
{
  "extends": "./tsconfig",
  "exclude": ["example","harmony"]
}
```
报错信息如下：

```js
harmony/blur/hvigorfile.ts:2:26 - error TS2307: Cannot find module '@ohos/hvigor-ohos-plugin' or its corresponding type declarations.

2 export { harTasks } from '@ohos/hvigor-ohos-plugin';
```

### 执行打包命令

```bash
// 运行bob-build，生成lib文件夹（若没bob-build则跳过）
npm run prepare
// 打包成tgz
npm pack
```
