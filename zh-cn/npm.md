# 发布三方库到 NPM 官方仓

> [!ATTENTION] 请确保本地可以成功打包 tgz 再来发布

已经开源的三方库，我们可以发布到 npm 官方公共仓上。

## 加入 npm organization

首先需要创建一个 [npm](https://www.npmjs.com/) 账户，然后联系管理员邀请加入 npm 组织：[react-native-oh-tpl](https://www.npmjs.com/org/react-native-oh-tpl)。

## 配置 package.json

需要更改 `package.json` 里几个字段的内容。

```json
{
    ...
    "name": "@react-native-oh-tpl/包名",
    "version": "自行管理好版本号",
    "repository": {
        "type": "git",
        "url": "三方库仓库地址.git"
    },
    "publishConfig": {
        "registry": "https://registry.npmjs.org/",
        "access": "public"
    },
    ...
}
```

## 发布

在命令行登录 npm

```bash
npm login
```

发布包到 npm

```bash
npm publish
```

若需要发布 next 版本，可输入

```bash
npm publish --tag=next
```
