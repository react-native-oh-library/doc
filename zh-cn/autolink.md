> [!WARNING] 本文档为非公开文档，仅用于三方库使用和开发指导，不涉及任何 React Native OpenHarmony 框架的信息，且会随着 React Native OpenHarmony 框架持续迭代更新，当前版本不代表最终展示版本。

# AutoLink

React Native 0.60 加入了一个很重要的特性: **autolinking** ，从此项目引入第三方库中的原生依赖再也不用额外调用 `react-native link` 命令了，同时 link 时也不会像之前一样入侵原生代码（例如修改 android 的 `setting.gradle`, `bulid.gradle`）

## Android 平台的 Link

参考：[React Native Autolinking 源码深入分析](https://blog.csdn.net/xx326664162/article/details/123478893)

### Android Manual Link

`react-native link` 做的事情有 3 件：

1. 在 `android/setting.gradle` 下添加:

```diff
+ include ':react-native库'
+ project(':react-native库').projectDir = new File(rootProject.projectDir, '../node_modules/react-native依赖/android')
```

2. 在 `android/app/build.gradle`

```diff
  dependencies {
+       implementation project(':react-native库')
  }
```

3. 在 `android/app/src/main/java/com/your-app/MainApplication.java`

```diff
+ import com.ocetnik.timer.BackgroundTimerPackage;

 @Override
 protected List<ReactPackage> getPackages() {
   return Arrays.<ReactPackage>asList(
+       new xx依赖Package()
   );
 }
```

> 第 1 和第 2 步对应 Harmony 在 oh-package.json 和 CMakeLists.txt 上的依赖路径配置，第 3 步对应在 index.ets 上引入包的修改。

### Android AutoLink

autolink 时并不会在这 3 个文件上添加任何代码(autolink 发生在项目编译打包过程中，而不是安装依赖时)

在三处地方比以往多了一些变化。

#### `setting.gradle`变化

```gradle
apply from: file("../node_modules/@react-native-community/cli-platform-android/native_modules.gradle"); applyNativeModulesSettingsGradle(settings)
include ':app'
```

`setting.gradle` 的作用是配置项目所需要用到的依赖，gradle 插件会加载里面的所有依赖并添加到 android 项目中。

从代码可知，这条语句是去调用 `/node_modules/@react-native-community/cli-platform-android/native_modules.gradle` 这个 gradle 文件下的 **applyNativeModulesSettingsGradle** 方法

```gradle
// native_modules.gradle
...
def autoModules = new ReactNativeModules(logger, projectRoot)

ext.applyNativeModulesSettingsGradle = { DefaultSettings defaultSettings, String root = null ->
  if (root != null) {
    logger.warn("${ReactNativeModules.LOG_PREFIX}Passing custom root is deprecated. CLI detects root automatically now.");
    logger.warn("${ReactNativeModules.LOG_PREFIX}Please remove second argument to `applyNativeModulesSettingsGradle`.");
  }
  autoModules.addReactNativeModuleProjects(defaultSettings)
}
...
```

可以看到，**applyNativeModulesSettingsGradle** 会去调用 **ReactNativeModules** 类的实例 **autoModules** 的 **addReactNativeModuleProjects**方法

```java
class ReactNativeModules {
  private Logger logger
  private String packageName
  private File root
  private ArrayList<HashMap<String, String>> reactNativeModules

  private static String LOG_PREFIX = ":ReactNative:"

  ReactNativeModules(Logger logger, File root) {
    this.logger = logger
    this.root = root

    def (nativeModules, packageName) = this.getReactNativeConfig()
    this.reactNativeModules = nativeModules
    this.packageName = packageName
  }

  /**
   * Include the react native modules android projects and specify their project directory
   */
  void addReactNativeModuleProjects(DefaultSettings defaultSettings) {
    reactNativeModules.forEach { reactNativeModule ->
      String nameCleansed = reactNativeModule["nameCleansed"]
      String androidSourceDir = reactNativeModule["androidSourceDir"]
      defaultSettings.include(":${nameCleansed}")
      defaultSettings.project(":${nameCleansed}").projectDir = new File("${androidSourceDir}")
    }
  }
  ...
}
```

**defaultSettings** 是在 `setting.gradle` 里作为**applyNativeModulesSettingsGradle** 的参数传进来，本质是 `setting.gradle` 文件的内置隐含对象 settings

所以

```gradle
defaultSettings.include(":${nameCleansed}")
defaultSettings.project(":${nameCleansed}").projectDir = new File("${androidSourceDir}")
```

等同于在 `setting.gradle` 里添加

```gradle
include ":${nameCleansed}"
project(":${nameCleansed}").projectDir = new File("${androidSourceDir}")
```

#### `android/app/build.gradle`变化

配置依赖项并生成依赖包

```gradle
apply from: file("../../node_modules/@react-native-community/cli-platform-android/native_modules.gradle");
applyNativeModulesAppBuildGradle(project)
```

这条语句跟 `setting.gradle` 那里一样，目标文件都是 `native_modules.gradle`，只不过这次调用的是其中的 **applyNativeModulesAppBuildGradle** 方法

```gradle
// native_modules.gradle
ext.applyNativeModulesAppBuildGradle = { Project project, String root = null ->
  if (root != null) {
    logger.warn("${ReactNativeModules.LOG_PREFIX}Passing custom root is deprecated. CLI detects root automatically now");
    logger.warn("${ReactNativeModules.LOG_PREFIX}Please remove second argument to `applyNativeModulesAppBuildGradle`.");
  }
  autoModules.addReactNativeModuleDependencies(project)

  def generatedSrcDir = new File(buildDir, "generated/rncli/src/main/java")
  def generatedCodeDir = new File(generatedSrcDir, generatedFilePackage.replace('.', '/'))

  task generatePackageList {
    doLast {
      autoModules.generatePackagesFile(generatedCodeDir, generatedFileName, generatedFileContentsTemplate)
    }
  }

  preBuild.dependsOn generatePackageList // build之前执行生成包列表的task

  android {
    sourceSets {
      main {
        java {
          srcDirs += generatedSrcDir //添加一个java源文件目录
        }
      }
    }
  }
}
```

这里首先调用了 **addReactNativeModuleDependencies** 方法

```gradle
 void addReactNativeModuleDependencies(Project appProject) {
    reactNativeModules.forEach { reactNativeModule ->
      def nameCleansed = reactNativeModule["nameCleansed"]
      appProject.dependencies {
        // TODO(salakar): are other dependency scope methods such as `api` required?
        implementation project(path: ":${nameCleansed}")
      }
    }
  }
```

作用等同于

```gradle
   dependencies {
       implementation project(':react-nativexx库')
       implementation project(':react-nativexxx库')
  }
```

然后调用 **generatePackagesFile** 方法

```gradle
 void generatePackagesFile(File outputDir, String generatedFileName, String generatedFileContentsTemplate) {
    ArrayList<HashMap<String, String>>[] packages = this.reactNativeModules
    String packageName = this.packageName

    String packageImports = ""
    String packageClassInstances = ""

    if (packages.size() > 0) {
      def interpolateDynamicValues = {
        it
                .replaceAll(~/([^.\w])(BuildConfig|R)([^\w])/, {
                  wholeString, prefix, className, suffix ->
                    "${prefix}${packageName}.${className}${suffix}"
                })
      }
      packageImports = packages.collect {
        "// ${it.name}\n${interpolateDynamicValues(it.packageImportPath)}"
      }.join('\n')
      packageClassInstances = ",\n      " + packages.collect {
        interpolateDynamicValues(it.packageInstance)
      }.join(",\n      ")
    }

    String generatedFileContents = generatedFileContentsTemplate
      .replace("{{ packageImports }}", packageImports)
      .replace("{{ packageClassInstances }}", packageClassInstances)

    outputDir.mkdirs()
    final FileTreeBuilder treeBuilder = new FileTreeBuilder(outputDir)
    treeBuilder.file(generatedFileName).newWriter().withWriter { w ->
      w << generatedFileContents
    }
  }
```

最终结果是生成了一个文件 `PackageList.java`，路径是 `/android/build/generated/rn/src/main/java/com/facebook/react/PackageList.java`。
这个代码定义了一个 **PackageList** 类，重点是 **getPackages()** 方法,返回所有的 package。

#### `android/app/src/main/java/com/your-app/MainApplication.java`变化

```java
  protected List<ReactPackage> getPackages() {
          @SuppressWarnings("UnnecessaryLocalVariable")
          List<ReactPackage> packages = new PackageList(this).getPackages();
          // Packages that cannot be autolinked yet can be added manually here, for example:
          // packages.add(new MyReactNativePackage());
          return packages;
        }
```

可以看出，一个叫 `com.facebook.react.PackageList` 包被导入了，这个包就是上一步生成的 `PackageList.java`，通过 **sourceSets** 指定源文件目录被编译和识别。

`MainApplication.java`的 **getPackages()** 调用了 **PackageList** 类实例的 **getPackages()**，将返回的所有包封装成一个数组返回，相当于自动化做了如下的修改

```java
  @Override
    protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
            new xx依赖Package()
      );
    }
```

这 3 处变化使得 Android 的 link 过程自动化，同时避免了原生代码的改动

## HarmonyOS 平台的 Link

### HarmonyOS Link

### HarmonyOS AutoLink

目前 HarmonyOS 平台的 AutoLink 还在规划阶段，预计会在未来推出。
