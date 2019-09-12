
### 前言
安卓工程通过gradle进行构建，为此了解安卓中的gradle十分重要。

### 文件结构
* 工程中有一个setting.gradle文件
* 工程中有一个build.gradle文件
* 每一个模块（如app）中有一个build.gradle文件

### 工程级build.gradle
范例：
```
// 用于配置gradle自身的仓库与依赖
buildscript {
    // 该仓库被gradle用于搜索与下载依赖
    repositories {}

    // 该依赖为用于构建工程所需要使用的依赖
    dependencies {}
}

// 此处用于配置所有模块的仓库和依赖。对于模块特有的依赖，需要在模块的build.gradle中配置。
allprojects {
   repositories {
       google()
       jcenter()
   }
}

// 通过ext可以配置project级的属性，即该项目中所有的module均可访问
ext {
    compileSdkVersion = 28
}
```
在模块中，可以通过以下方式访问：
```
android {
  compileSdkVersion rootProject.ext.compileSdkVersion
}
dependencies {
    implementation "com.android.support:appcompat-v7:${rootProject.ext.supportLibVersion}"
}
```
> 应尽量避免使用project的级别的全局变量。这会使得到处某个模块十分困难，以及影响gradle的并行编译效率。

### 模块级build.gradle
```
// 配置插件，插件提供了预制的构建能力。apply plugin表示二进制插件，其后是插件名
// 安卓应用的插件名为application,模块的插件名为library。
apply plugin: 'com.android.application'

// 配置安卓相关的构建选项
android {
    // gradle编译时使用的API版本
    compileSdkVersion 28
    // 表示SDK工具的版本，该属性指定的版本需要通过SDK Manager下载。该属性为可选的。
    buildToolsVersion "28.0.3"

    // 对于不同Build Variable的默认配置，在main/AndroidManifest中或者具体的Build Flavor中可以覆盖
    defaultConfig {
        applicationId 'com.example.myapp'
        minSdkVersion 15
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"
    }

    // 配置构建类型
    buildTypes {
        // 默认有debug类型
        release {
            minifyEnabled true // Enables code shrinking for the release build type.
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    // 配置产品类型
    flavorDimensions "area"
    productFlavors {
        china {
        dimension "area"
        applicationId 'com.example.myapp.free'
        }

        oversea {
        dimension "area"
        applicationId 'com.example.myapp.paid'
    }

    // 配置多个apk的构建信息，不同apk可以适配不同的屏幕密度，或者不同的版本号
    splits {
    density {
      // Enable or disable building multiple APKs.
      enable false
      // Exclude these densities when building multiple APKs.
      exclude "ldpi", "tvdpi", "xxxhdpi", "400dpi", "560dpi"
    }
  }
}

// 模块依赖
dependencies {
    implementation project(":lib")
    implementation 'com.android.support:appcompat-v7:28.0.0'
    implementation fileTree(dir: 'libs', include: ['*.jar'])
}
```

### gradle配置文件
* gradle.properties
项目级gradle配置，如gradle守护进程堆大小

* local.properties
配置本地环境信息，如SDK路径。该文件一般由AndroidStudio生成，不建议手动修改。

### Source Set
源码集有多个维度，同时支持两两组合，如：
```
src/buildType/
src/productFlavor/
src/productFlavorBuildType/
```
对于BuildVariables为fullDebug的类型，其源码、设置及资源一共来自于：
* src/fullDebug/
* src/debug/ 
* src/full/
* src/main/ 

如果在这些目录中拥有相同的文件，则高优先级的会覆盖低优先级的。具体的优先级规则如下：
```
build variant > build type > product flavor > main source set > library dependencies
```
