> [!WARNING] 本文档为非公开文档，仅用于三方库使用和开发指导，不涉及任何 React Native OpenHarmony 框架的信息，且会随着 React Native OpenHarmony 框架持续迭代更新，当前版本不代表最终展示版本。

# 开源管理

本章节仅供内部开发者查阅，方便 RNOH 三方库后续展开开源工作。

目前所有的 RNOH 三方库的源码统一放在 Github 的 [react-native-oh-library](https://github.com/orgs/react-native-oh-library/repositories) 组织管理，请相关开发者先申请加入。

## 建仓

> [!tip] 如果没有建仓权限，请联系组织管理员帮忙操作。

首先确认需要移植的 React-Native 三方库的原始仓库地址，然后 fork 默认分支（一般是 `master` 或 `main`）到 react-native-oh-library。Description 的内容改为这个三方库在 NPM 上的包名。

![create repo](../img/create_repo.png)

点击 "View all branches" 修改分支名称为 `harmony`，作为仓库的默认分支。开发时，请另外新建开发分支（推荐统一为 `dev`）。

![branch](../img/branch.png)

如果发现落后原库默认分支，可点击 `Sync fork` 来跟进，请在开发分支进行此操作。sync 之后要注意原库版本有没有变更。

## 发布

发布步骤：

1. 确保已通过自检测试，并完成代码格式检查；
2. 将变更合入 `harmony` 分支；
3. 发布新的 Tag 和 Release（github 操作）；
4. 发布新的 NPM Package（本地操作）。

### 代码格式检查

请查看 [代码格式检查](zh-cn/codelint.md) 章节

### Tags and Releases

按以下步骤操作：

1. Tag 名称、Release 名称一致:

> x.x.x-y.y.y

其中 x.x.x 为基版本，即基于原库哪一个版本；y.y.y 为鸿蒙化过程中自行定义的临时版本号。

2. Target 分支 选择默认分支 harmony（请确保变更已通过测试、代码检查和合入了默认分支）

3. Release 描述按以下格式：

```md
[Fixes]:

1. 问题修复 1
2. 问题修复 2
3. ...

[Version Info]:

- RNOH: 0.72.11
- DevEco Studio: 4.0.3.700
- OH SDK: 4.0.10.11
- ROM: 4.0.0.66(SP3C00E73R1P14log)
```

4. 需要上传本地打包的 tgz 文件

![tag&release](../img/tag&release.png)

### Package 的命名规则：

**情况 1.** 如果原库在 NPM 上的包带有组织前缀，如 "@react-native-community/slider", "@react-native-async-storage/async-storage"等；

去掉组织名取后半段：

> "@react-native-oh-library/原包名后半段" // 私仓

> "@react-native-oh-tpl/原包名后半段" // 公仓

example：

```md
// 私仓
"@react-native-community/slider" → "@react-native-oh-library/slider"

// 公仓
"@shopify/flash-list" → "@react-native-oh-tpl/flash-list"
```

**情况 2.** 如果原库在 NPM 上的包没有组织前缀，如 "react-native-pager-view" 等；

直接添加新的组织名：

> "@react-native-oh-library/原包名" // 私仓

> "@react-native-oh-tpl/原包名" // 公仓

example：

```md
// 私仓
"react-native-translucent-modal" → "@react-native-oh-library/react-native-translucent-modal"

// 公仓
"react-native-pager-view" → "@react-native-oh-tpl/react-native-pager-view"
```

如果有重名等其他特殊情况，请联系组织管理员协商。

### Package 版本：

> x.x.x-y.y.y

其中 x.x.x 为基版本，即基于原库哪一个版本；y.y.y 为鸿蒙化过程中自行定义的临时版本号。

### 添加别名

由于当前版本设置组织名后，会导致设置别名的库无法被找到，所以展示不需要设置别名。

~~在 `package.json` 里添加 "harmony" 字段：~~

```json
// 暂不配置
{
    "harmony": {
        "alias": {原NPM包名}
    },
}
```

~~RNOH 的打包工具会识别出 `node_modules` 下第一级目录的所有 RNOH 三方库的别名（第二级目录暂不支持，已提 issue），这样在 JS 端 import 三方库使用的时候，可以使用原库的名字。如 import xxx from "@react-native-community/slider"。~~

### 发布 npm 包

#### 将私有 NPM 包托管到 Github Packages

请查阅 [发布三方库到 Github Packages](zh-cn/github-package.md)，也可参考已发布的 [@react-native-oh-library/react-native-slider](https://github.com/react-native-oh-library/react-native-slider)。

#### 将三方库发布到 npm 官方仓

请查阅 [发布三方库到 NPM 官方仓](zh-cn/npm.md)，也可参考已发布的 [@react-native-oh-tpl/react-native-linear-gradient](https://github.com/react-native-oh-library/react-native-linear-gradient)。
