> [!WARNING] 本文档为非公开文档，仅用于三方库使用和开发指导，不涉及任何 React Native OpenHarmony 框架的信息，且会随着 React Native OpenHarmony 框架持续迭代更新，当前版本不代表最终展示版本。

# 代码格式检查

## 常见代码检测/格式化工具

React-Native 三方库中，常见的代码检测/格式化工具有 ESlint、Prettier、Pre-Commit Hook 等。若三方库用了某个工具，那在上传代码前就需要使用。

### ESlint

ESlint 是一个按照规则给出报告的代码检测工具，也可以自动修改不符合规范的格式。

#### 使用

在 `package.json` 的 script 字段添加脚本或直接使用原库提供的脚本

```json
"script": {
   "lint": "npx eslint src/**/*.js",  // 代码检查
   "lint:fix": "npx eslint src/**/*.js --fix",  // 自动修复
   ...
}
// src 是需要检查的路径
```

可在 `.eslintignore` 文件里添加目标路径里不想执行检查的文件;

`.eslintrc.js` 或 `.eslintrc.json` 是 ESlint 的配置文件。

例子：

> @react-native-community/slider

`package.json` 里已经配置好了相关命令。

```json
 "script": {
    ...
    "lint": "npx eslint src",
    ...
 }
```

该脚本的目的是检查 `src` 文件夹下的 JS 代码并在命令行输出检查报告。

可直接在命令行里执行

<!-- tabs:start -->

#### **npm**

```bash
npm run lint
```

#### **yarn**

```bash
yarn lint
```

<!-- tabs:end -->

### Prettier

Prettier 是代码格式化工具，也可以格式化 MarkDown 文档

#### 使用

在 `package.json` 的 script 字段添加脚本或直接使用原库提供的脚本

```json
 "script": {
    ...
    "prettier": "prettier --write src/**/*.js",
    ...
 }
```

在命令行执行, `src/**/*.js` 是目标路径

```json
npm run prettier
```

<!-- tabs:start -->

#### **npm**

```bash
npm run prettier
```

#### **yarn**

```bash
yarn prettier
```

<!-- tabs:end -->

`.prettierrc.js`、`.prettierrc` 是 Prettier 的配置文件。

例子：

> react-native-svg

`package.json` 里已经配置好了 Prettier 的相关命令

```json
 "script": {
    ...
    "format-js": "prettier --write README.md CONTRIBUTING.md CODE_OF_CONDUCT.md USAGE.md ./src/**/*.{ts,tsx} ./Example/src/**/*.tsx",
    ...
 }
```

该脚本的目的是检查并自动修改 "--write" 参数指向文件的格式。

可直接在命令行里执行

<!-- tabs:start -->

#### **npm**

```bash
npm run format-js
```

#### **yarn**

```bash
yarn format-js
```

<!-- tabs:end -->
