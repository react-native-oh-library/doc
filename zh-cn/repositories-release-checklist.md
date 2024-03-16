# 发布 Release Checklist

### 简介

在仓库发布 release 版本时，请自行对照如下表格进行检查。

### Checklist

| 序号 | 内容                                            | 示例                                                                                                                                                                                                                               |
| ---- | ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | JS 侧是否切换打补丁形式                         | [react-native-linear-gradient](https://github.com/react-native-oh-library/react-native-linear-gradient)<br />https://github.com/react-native-oh-library/react-native-slider/tree/sig                                               |
| 2    | package 是否升级版本 xxx                        | 3.0.0-0.0.1 => 3.0.0-0.0.2                                                                                                                                                                                                         |
| 3    | package 是否添加组织名                          | react-native-linear-gradient => @react-native-oh-tpl/react-native-linear-gradient                                                                                                                                                  |
| 4    | package 是否配置 publishConfig                  | "publishConfig": { "registry": "https://registry.npmjs.org/", "access": "public" }                                                                                                                                                 |
| 5    | package file 是否配置鸿蒙相关目录文件           | "files": [ "harmony", "src", "index.d.ts", "index.harmony.js" ]                                                                                                                                                                    |
| 6    | package 是否配置 repository                     | "repository": { "type": "git", "url": "https://github.com/react-native-oh-library/react-native-linear-gradient.git" }                                                                                                              |
| 7    | package 是否配置原库依赖，实现打补丁方式        | 如@react-native-oh-tpl/react-native-linear-gradient 配置<br />"dependencies": { "react-native-linear-gradient": "3.0.0-alpha.1" },                                                                                                 |
| 8    | 发布 release 时，是否创建对应 tag               |                                                                                                                                                                                                                                    |
| 9    | 发布 release 时，是否添加 Fixes 和 Version Info | [react-native-linear-gradient Release](https://github.com/react-native-oh-library/react-native-linear-gradient/releases/tag/3.0.0-alpha.1-0.2.6)                                                                                   |
| 10   | 发布 release 时，是否添加 tgz 文件              | [react-native-linear-gradient Release](https://github.com/react-native-oh-library/react-native-linear-gradient/releases/tag/3.0.0-alpha.1-0.2.6)                                                                                   |
| 11   | 鸿蒙版本和 js 版本是否一致                      | 如 package 版本为 3.0.0-0.0.2，则 oh-package.json5 中的 version 也需要一致，修改需要更新到 har 和代码中                                                                                                                            |
| 12   | 鸿蒙模块名是否规范                              | oh-package.json5 中的 name 是否为 rnoh-xxx，使用 rnoh 代替原库 react-native 前缀，如果没有 react-native 前缀则直接添加 rnoh 前缀。<br />如：react-native-fast-image => rnoh-fast-image <br />Shopify/flash-list => rnoh-flash-list |
| 13   | 检查库是否添加平台判断                          | 代码需要对标 IOS 进行移植，涉及 platform 代码、xx.ios.tsx 文件等，需要重点关注与修改，建议源码全局进行搜索检查                                                                                                                     |
| 14   | 检查库是否测试完全所有暴露接口                  |                                                                                                                                                                                                                                    |
