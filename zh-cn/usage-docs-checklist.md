# 文档 Checklist

### 简介

在提交 usage-docs 文档时，请自行对照如下表格进行检查。

### Checklist

| 序号 | 内容                                  | 示例                                                                                                                                                  |
| ---- | ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | 是否按最新模板修改                    | 参考每个文件夹中的 model.md                                                                                                                           |
| 2    | 是否修改模板号                        | 模板版本：v0.1.2                                                                                                                                      |
| 3    | 是否填写 github 地址                  | 位于模板版本下面：[!tip] Github 地址                                                                                                                  |
| 4    | 是否未发布                            | 如未 npm 发包，则补充 release 获取说明<br /> **正在 npm 发布中，当前请先从仓库 Release 中获取库 tgz，通过使用本地依赖来安装本库。**                   |
| 5    | 安装方式是否配套                      | 如已适配打补丁，则使用正常安装：<br />yarn add @react-native-oh-tpl/react-native-linear-gradient<br />未适配则需要使用覆盖安装：<br />配置名@npm:库名 |
| 6    | Link 路径是否配套                     | 如已补丁整改的需要带上组织名：<br />file:../../node_modules/@react-native-oh-tpl/lottie-react-native/harmony/lottie.har                               |
| 7    | ArkTs 引入是否配套版本                | 如 rnoh 0.72.11 版本需使用 ctx.componentName 和 ctx.tag                                                                                               |
| 8    | 是否添加权限限制                      | 如 fastimage 需要网络权限才可加载网络图片，则需要特别说明。                                                                                           |
| 9    | 属性章节是否平台说明                  | "Platform"列表示...<br /> "HarmonyOS Support"列为 yes...                                                                                              |
| 10   | 属性章节表格头是否使用英文头          | 参考模板                                                                                                                                              |
| 11   | 属性章节 Description 是否补充平台描述 | 如 lottie imageAssetsFolder 属性中提到 ios 和 android 平台，那么需要补充 HarmonyOS 描述。                                                             |
| 12   | 遗留问题是否有 issue 跟踪             |                                                                                                                                                       |
| 13   | 开源协议是否和原库一致                |                                                                                                                                                       |
| 14   | 纯 JS 库是否补充有可支持版本          |                                                                                                                                                       |
