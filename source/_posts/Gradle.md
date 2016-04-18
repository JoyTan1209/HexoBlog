layout: posts
title: "Android Studio Gradle"
date: 2016-04-09 21:50:04
tags:
    - Android 
    - 移动开发
---
### 简介
Android Studio 使用 Gradle 构建工具，而 Gradle 继承了强大、灵活的 Ant 和 Maven 丰富的依赖管理，配置管理简单，脚本编写方便灵活，插件模块化。

Android Studio 使用 Gradle 构建工具，Eclipse 的 ADT 插件使用的是 Ant 构建工具。因为两个构建工具的区别，导致习惯了 Eclipse 开发环境的开发者刚开始比较难适应 Android Studio。如果要迁移到 Android Studio，建议最好了解下 Gradle 构建工具。Gradle 构建工具是任务驱动型的构建工具，并且可以通过各种 Plugin 插件扩展功能以适应各种构建任务。对应 Android 项目的 Gradle 插件就是 Android Gradle Plugin。

### 为什么使用
Android Studio相比ADT的好处就不啰嗦了，好马配好鞍，Android Studio为什么采用Gradle作为构建工具呢？Gradle是一个优秀的构建系统和构建工具，可以通过插件来创建自定的义构建逻辑。
Gradle的优点：
* 采用了Domain Specific Language(DSL 语言) 来描述和控制构建逻辑。
* 构建文件基于 Groovy，并且允许通过混合声明 DSL 元素和使用代码来控制 DSL 元素以控制自定义的构建逻辑。
* 支持 Maven 或者 Ivy 的依赖管理。
* 非常灵活。允许使用最好的实现，但是不会强制实现的方式。
* 插件可以提供自己的 DSL 和 API 以供构建文件使用。
* 良好的 API 工具供 IDE 集成。

Gradle构建系统的目标：
* 让重用代码和资源变得更加容易
* 让创建同一个APP的不同版本变得更加容易，无论是多个APP发布版本还是同一个发布不同的版本
* 让构建过程变得更加容易配置，扩展和定制
* 整合优秀IDE

### 项目的构建文件
Gradle在Android Studio的项目中是如何使用的呢？一个Gradle项目的构建过程定义在了build.gradle的文件中，在项目的根目录下，如图：

![android项目](../../../../img/project.png)

最简单的Android项目的build.gradle文件包含以下内容：

``` java
    buildscript {
        repositories {
            mavenCentral()
        }

        dependencies {
            classpath 'com.android.tools.build:gradle:0.11.1'
        }
    }

    apply plugin: 'android'

    android {
        compileSdkVersion 19
        buildToolsVersion "19.0.0"
    }
```

简单分析下:

``` java
    buildscrip{...} 
```

配置驱动构建过程的代码，在这个部分声明了使用Maven仓库，并且声明了一个maven文件的依赖路径。这个文件就是包含了0.11.1版本android gradle插件的库。

``` java
apply plugin: 'android'
```

这里添加了android插件

``` java
android{...} 
```

这里配置android构建过程需要的参数

默认情况下，只需要配置目标编译SDK版本和编译工具版本，即compileSdkVersion和buildToolsVersion属性。 这个complieSdkVersion属性相当于旧构建系统中project.properites文件中的target属性。这个新的属性可以跟旧的target属性一样指定一个int或者String类型的值。

与上面基本的构建文件对应的是一个默认的文件夹结构。Gradle遵循约定优先于配置的概念，我们需要在可能的情况尽可能提供合理的默认配置参数。

基本的项目开始于两个名为“source sets”的组件，即main source code和test code。分别位于：
* src/main/
* src/androidTest/
里面每一个文件夹都有与之相应的源组件:
* AndroidManifest.xml
* res/
* assets/
* aidl/
* rs/
* jni/

### 配置结构
当默认的项目结构不适用的时候，你可能需要去配置它。根据Gradle文档，重新为Android项目sourceSets。例如：

``` bash
    android {
        sourceSets {
            main {
                manifest.srcFile 'AndroidManifest.xml'
                java.srcDirs = ['src']
                resources.srcDirs = ['src']
                aidl.srcDirs = ['src']
                renderscript.srcDirs = ['src']
                res.srcDirs = ['res']
                assets.srcDirs = ['assets']
            }

            androidTest.setRoot('tests')
        }
    }
```

它使用了旧项目结构中的main源码，并且将androidTest _sourceSet_组件重新映射到_tests_文件夹。

### 构建任务

添加一个插件到构建文件中将会自动创建一系列构建任务(build tasks)去执行（注：gradle属于任务驱动型构建工具，它的构建过程是基于Task的）。Java plugin和Android plugin都会创建以下task：

* assemble
    这个task将会组合项目的所有输出。
* check
    这个task将会执行所有检查。
* connectedCheck
    这个task将会在一个指定的设备或者模拟器上执行检查，它们可以同时在所有连接的设备上执行。
* deviceCheck
    通过APIs连接远程设备来执行检查，这是在CL服务器上使用的。
* build
    这个task将会执行assemble和check两个task的所有工作
* clean
    这个task将会清空项目的输出。

实际上assemble，check，build这三个task不做任何事情。它们只是一个Task标志，用来告诉android plugin添加实际需要执行的task去完成这些工作。

这就允许你去调用相同的task，而不需要考虑当前是什么类型的项目，或者当前项目添加了什么plugin。 例如，添加了findbugs plugin将会创建一个新的task并且让check task依赖于这个新的task。当check task被调用的时候，这个新的task将会先被调用。

在命令行环境中，你可以执行以下命令来获取更多高级别的task：

    gradle tasks
查看所有task列表和它们之间的依赖关系可以执行以下命令：

    gradle tasks --all
注意：
Gradle会自动监视一个task声明的所有输入和输出。
两次执行build task并且期间项目没有任何改动，gradle将会使用UP-TO-DATE通知所有task。这意味着第二次build执行的时候不会请求任何task执行。这允许task之间互相依赖，而不会导致不需要的构建请求被执行。

一个Android项目至少拥有两个输出：debug APK（调试版APK)和release APK（发布版APK）。每一个输出都拥有自己的标识性task以便能够单独构建它们。
* assemble
    * assembleDebug
    * assembleRelease
它们都依赖于其它一些tasks以完成构建一个APK需要多个步骤。其中assemble task依赖于这两个task，所以执行assemble将会同时构建出两个APK。

check task也拥有自己的依赖：
* check
    * lint
* connectedCheck
    * connectedAndroidTest
    * connectedUiAutomatorTest(目前还没有应用到）
* deviceCheck
    * 这个test依赖于test创建时，其它实现测试扩展点的插件。

最后，只要task能够被安装（那些要求签名的task），android plugin就会为所有构建类型（debug，release，test）安装或者卸载。

### 基本的构建定制
Android plugin提供了大量DSL用于直接从构建系统定制大部分事情。

Manifest 属性

通过SDL可以配置一下manifest选项：

* minSdkVersion
* targetSdkVersion
* versionName
* applicationId (有效的包名 -- 更多详情请查阅ApplicationId 对比 PackageName)
* package Name for the test application
* Instrumentation test runner

例如：

``` java
    android {
        compileSdkVersion 19
        buildToolsVersion "19.0.0"

        defaultConfig {
            versionCode 12
            versionName "2.0"
            minSdkVersion 16
            targetSdkVersion 16
        }
    }
```
在android元素中的defaultConfig元素中定义所有配置。

### 构建类型

默认情况下，Android Plugin会自动给项目设置同时构建应用程序的debug和release版本。 两个版本之间的不同主要围绕着能否在一个安全设备上调试，以及APK如何签名。

Debug版本采用使用通用的name/password键值对自动创建的数字证书进行签名，以防止构建过程中出现请求信息。Release版本在构建过程中没有签名，需要稍后再签名。

这些配置通过一个BuildType对象来配置。默认情况下，这两个实例都会被创建，分别是一个debug版本和一个release版本。

Android plugin允许像创建其他构建类型一样定制debug和release实例。这需要在buildTypes的DSL容器中配置：

``` java
    android {
        buildTypes {
            debug {
                applicationIdSuffix ".debug"
            }

            jnidebug.initWith(buildTypes.debug)
            jnidebug {
                packageNameSuffix ".jnidebug"
                jnidebugBuild true
            }
        }
    }
```
以上代码片段实现了以下功能：
* 配置默认的debug构建类型
    * 将debug版本的包名设置为.debug以便能够同时在一台设备上安装_debug_和_release_版本的apk。
    * 创建了一个名为jnidebug的新构建类型，并且这个构建类型是debug构建类型的一个副本。
    * 继续配置jnidebug构建类型，允许使用JNI组件，并且也添加了不一样的包名后缀。

创建一个新的构建类型就是简单的在buildType标签下添加一个新的元素，并且可以使用initWith()或者直接使用闭包来配置它。

### 签名配置

对一个应用程序签名需要以下：
* 一个Keystory
* 一个keystory密码
* 一个key的别名
* 一个key的密码
* 存储类型

位置，键名，两个密码，还有存储类型一起形成了签名配置。

默认情况下，debug被配置成使用一个debug keystory。 debug keystory使用了默认的密码和默认key及默认的key密码。 debug keystory的位置在$HOME/.android/debug.keystroe，如果对应位置不存在这个文件将会自动创建一个。

debug Build Type(构建类型) 会自动使用debug SigningConfig (签名配置)。

可以创建其他配置或者自定义内建的默认配置。通过signingConfigs这个DSL容器来配置：

``` java
    android {
        signingConfigs {
            debug {
                storeFile file("debug.keystore")
            }

            myConfig {
                storeFile file("other.keystore")
                storePassword "android"
                keyAlias "androiddebugkey"
                keyPassword "android"
            }
        }

        buildTypes {
            foo {
                debuggable true
                jniDebugBuild true
                signingConfig signingConfigs.myConfig
            }
        }
    }
```
以上代码片段修改debug keystory的路径到项目的根目录下。在这个例子中，这将自动影响其他使用到debug构建类型的构建类型。

### 运行 Proguard

从Gradle Plugin for ProGuard version 4.10之后就开始支持ProGuard。ProGuard插件是自动添加进来的。如果_Build Type_的_runProguard_属性被设置为true，对应的task将会自动创建。

``` java
    android {
        buildTypes {
            release {
                runProguard true
                proguardFile getDefaultProguardFile('proguard-android.txt')
            }
        }

        productFlavors {
            flavor1 {
            }
            flavor2 {
                proguardFile 'some-other-rules.txt'
            }
        }
    }
```
发布版本将会使用它的Build Type中声明的规则文件，product flavor（定制的产品版本）将会使用对应flavor中声明的规则文件。

这里有两个默认的规则文件：
* proguard-android.txt
* proguard-android-optimize.txt

这两个文件都在SDK的路径下。使用_getDefaultProguardFile()_可以获取这些文件的完整路径。它们除了是否要进行优化之外，其它都是相同的。