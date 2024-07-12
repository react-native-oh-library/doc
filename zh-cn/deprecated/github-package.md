> [!WARNING] 本文档为非公开文档，仅用于三方库使用和开发指导，不涉及任何 React Native OpenHarmony 框架的信息，且会随着 React Native OpenHarmony 框架持续迭代更新，当前版本不代表最终展示版本。

# 发布三方库到 Github Packages

> [!ATTENTION] 请确保本地可以成功打包 tgz 再来发布

我们可以配置 npm 以将包发布到 GitHub Packages 并将存储在 GitHub Packages 上的包用作 npm 项目中的依赖项。

首先需要加入 github 组织：[react-native-oh-library](https://github.com/react-native-oh-library)，然后在组织里创建私有(private)仓库。

## 创建 Github 个人访问令牌（personal access token）

该步骤可以直接跳过，目前 react-native-oh-library 组织共用一个访问令牌，请向组织管理员问取访问令牌。

## 方法一：使用 Github Registry 使用 NPM 进行身份验证

要发布 npm 包，需要通过 Github 包注册表对 npm 进行身份验证。有两种方法可以做到这一点：

1. 使用.npmrc 文件进行身份验证
2. 使用命令行

### 方法二：使用 .npmrc 验证（推荐）

在三方库目录下新建 ~/.npmrc 文件（如果不存在）并添加下行，将 TOKEN 替换为你的 personal access token。

```
//npm.pkg.github.com/:_authToken=TOKEN
```

### 使用命令行验证

执行以下命令，将 USERNAME 替换为你的 GitHub 用户名，将 TOKEN 替换为你的 personal access token (classic)，@NAMESPACE 替换为托管包的命名空间

```bash
$ npm login --scope=@NAMESPACE --auth-type=legacy --registry=https://npm.pkg.github.com

> Username: USERNAME
> Password: TOKEN
```

## 发布包

同样的，有两种发布包的方式：

1. 使用 package.json 中的 publishConfig 选项来设置项目的作用域映射来发布
2. 使用项目中的本地 .npmrc 文件发布

这里只介绍第 1 种。

### 使用 package.json 文件中的 publishConfig 发布包

1. 编辑包的 `package.json` 文件并包含一个 `publishConfig` 条目。

<!-- tabs:start -->

#### **package.json**

```json
"publishConfig": {
  "registry": "https://npm.pkg.github.com"
},
```

<!-- tabs:end -->

2. 验证项目 `package.json` 中的 repository 字段。 repository 字段必须与 GitHub 存储库的 URL 匹配。 例如，如果存储库 URL 是 github.com/my-org/test，则存储库字段应为 https://github.com/my-org/test.git。

3. 发布包

```bash
npm publish
```

# 使用私仓的库

通过在项目的 package.json 文件中将包添加为依赖项，可以从 GitHub Packages 安装包。

1. 在与 package.json 文件相同的目录中，创建或编辑 .npmrc 文件以包含指定 GitHub Packages URL 和托管包的命名空间的行。 将 NAMESPACE 替换为作为包限定范围的用户或组织帐户的名称，rnoh 三方库对应的是@react-native-oh-library。将 TOKEN 替换为你的 personal access token (classic)，用来向 GitHub Packages 验证。

```npmrc
@NAMESPACE:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=TOKEN
```

2. 在项目中配置 package.json 以使用要安装的包。 将 ORGANIZATION_NAME/PACKAGE_NAME 替换为包依赖项，x.x.x 替换为版本。

<!-- tabs:start -->

#### **package.json**

```json
"dependencies": {
  "PACKAGE_NAME": "npm:@ORGANIZATION_NAME/PACKAGE_NAME@x.x.x"
},
```

<!-- tabs:end -->

> [!tip] 若有疑问请参考 [@react-native-oh-library/react-native-slider](https://github.com/react-native-oh-library/react-native-slider)。
