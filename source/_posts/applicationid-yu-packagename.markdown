---
layout: post
title: "ApplicationId 与 PackageName"
date: 2015-06-04 23:24:06 +0800
comments: true
categories:
- 翻译
tags:
- 翻译
- Android
---

> 基于 <http://blog.csdn.net/maosidiaoxian/article/details/41719357> 校对整理，感谢原文作者 [貌似掉线](http://my.csdn.net/maosidiaoxian)

Android 应用都有自己的包名。包名是设备上每个应用程序的唯一标识，同样也是 Google Play 商店里的唯一标识。就是说，假如你已经使用某个包名来发布应用，**就不能再去改变应用的包名**，因为这样做会导致你的应用被视为一个全新的应用，你现有的用户也不会收到应用的更新通知。
<!-- more -->

旧版的 Android Gralde 构建系统中，应用的包名由 manifest 中根节点的 package 属性决定：

``` Groovy AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.my.app"
    android:versionCode="1"
    android:versionName="1.0" >
```

然而，这里所定义的 package 还有另一个作用：用来命名资源类 R（以及用于解析相关的 Activity）。在上面的示例中，最终生成的 R 类为 **<font color='green'>com.example.my.app.R</font>**，所以如果你在其他包中的代码需要引用资源，对应的 .java 文件需要导入 `com.example.my.app.R`。

在新的 Android Gradle 构建系统中，你可以轻松地构建多个不同版本的应用。例如，你可以同时构建免费版和专业版的应用（使用 flavor），并且它们在 Google Play 上也应该要有不同的包名，这样它们就能够在同一设备上安装并且能够单独购买使用等等。同样的，你也可以构建 “debug”、“alpha”、“beta” 版的应用（使用 build type），它们也同样可以有唯一的包名。

同时，代码中引用的 R 类要保持不变；在构建不同版本的应用时，对应的（引用了 R 的） .java 源文件也不能改动。

因此，我们将包名的两种作用**解耦**：

* “application id” 对应 apk 中 manifest 定义的应用包名，同时用于设备以及 Google Play 的应用唯一标识。
* “package” 用于在源码中引用 R 类以及解析注册相关的 activity/service，对应 Java 的包名概念。

你可以在 Gradle 文件中指定 application id，如下所示：

``` Groovy app/build.gradle
apply plugin: 'com.android.application'

android {
    compileSdkVersion 19
    buildToolsVersion "19.1"

    defaultConfig {
        applicationId "com.example.my.app"
        minSdkVersion 15
        targetSdkVersion 19
        versionCode 1
        versionName "1.0"
    }
    ...
```

像以前一样，你需要像前面的 AndroidManifest.xml 示例在 Manifest 中指定给代码用的 “package”。

**关键部分**：参照上面的做法，即能解耦 **<font color="green">applicationId</font>** 和 **<font color="green">package</font>**。意思是你能够完全自由地重构你的代码，改变用于 Activity 和 Service 的内部包，改变 Manifest 的 package，重构导入语句。这都不会影响到 app 的最终 id，app 的 id 对应 Gradle 文件中 applicationId 的值。

你可以通过以下的 Gradle DSL 方法来为不同的 flavor 和 build type 定义不同的 applicationId：

``` Groovy app/build.gradle
    productFlavors {
        pro {
            applicationId = "com.example.my.pkg.pro"
        }
        free {
            applicationId = "com.example.my.pkg.free"
        }
    }

    buildTypes {
        debug {
            applicationIdSuffix ".debug"
        }
    }
    ....
```

(在 Android Studio 中，你可以通过 Project Structure 图形化界面来进行这些配置。)

注意：出于兼容性考虑，如果**没有**在 build.gradle 文件中定义 applicationId，那么 applicationId 将默认为 AndroidManifest.xml 中所指定的 package 的值。在这种情况下，applicationId 和 package 显然未解耦，此时重构代码也将会更改应用的 id ！在 Android Studio 中，新建的项目会指定这两个值。

注意：package 始终必须在默认 AndroidManifest.xml 文件中指定。如果存在多个 manifest（例如一个 flavor 有特定的 manifest 或一个 buildType 有特定的 manifest），package 可不指定，但如果被指定，必须和主 manifest 中指定的 package 完全相同。