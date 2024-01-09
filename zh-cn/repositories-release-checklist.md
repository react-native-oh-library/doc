# 发布 Release Checklist

### 简介

在仓库发布 release 版本时，请自行对照如下表格进行检查。

### Checklist

| 序号   | 内容                                     | 示例                                       |
| ---- | -------------------------------------- | ---------------------------------------- |
| 1    | JS 侧是否切换打补丁形式                          | [react-native-linear-gradient](https://github.com/react-native-oh-library/react-native-linear-gradient)<br />https://github.com/react-native-oh-library/react-native-slider/tree/sig |
| 2    | package 是否升级版本 xxx                     | 3.0.0-0.0.1 => 3.0.0-0.0.2               |
| 3    | package 是否添加组织名                        | react-native-linear-gradient => @react-native-oh-tpl/react-native-linear-gradient |
| 4    | package 是否配置 publishConfig             | "publishConfig": { "registry": "https://registry.npmjs.org/", "access": "public" } |
| 5    | package file 是否配置鸿蒙相关目录文件              | "files": [ "harmony", "src", "index.d.ts", "index.harmony.js" ] |
| 6    | package 是否配置 repository                | "repository": { "type": "git", "url": "https://github.com/react-native-oh-library/react-native-linear-gradient.git" } |
| 7    | package 是否配置原库依赖，实现打补丁方式               | 如@react-native-oh-tpl/react-native-linear-gradient 配置<br />"dependencies": { "react-native-linear-gradient": "3.0.0-alpha.1" }, |
| 8    | 发布 release 时，是否创建对应 tag                |                                          |
| 9    | 发布 release 时，是否添加 Fixes 和 Version Info | [react-native-linear-gradient Release](https://github.com/react-native-oh-library/react-native-linear-gradient/releases/tag/3.0.0-alpha.1-0.2.6) |
| 10   | 发布 release 时，是否添加 tgz 文件               | [react-native-linear-gradient Release](https://github.com/react-native-oh-library/react-native-linear-gradient/releases/tag/3.0.0-alpha.1-0.2.6) |
