layout: post
title: "Android Studio 使用 Gradle 打包 Jar"
date: 2015-08-02 23:04:24 +0800
updated: 2015-08-15 12:35:11 +0800
comments: true
categories:
- Android
tags:
- Android
- 技巧
- Gradle
---

> ***注：***该文章只适用于对 Application Plugin 的 module 进行打包 Jar 或 Library 析出的 Jar 有多余的 class

Android Studio 打 Jar 包一直是一个麻烦的事，按照网上现有的教程，打包一个混淆的 jar 需要完成下列步骤：

1. 将 plugin 修改为 `library`（有指定 applicationId 情况下还需要注释对应代码），运行命令 `gradle bundleRelease`，等待完成
3. 找到对应 module 的 `build/intermediates/bundles/debug or release/classes.jar`（感谢 [@ZefanXie][1] 指出）
4. 使用 jarjar 等工具剔除多余的 class

这一个过程要改的东西比较多，于是花了些时间研究了下 Gradle 打 Jar 包。

[1]: http://weibo.com/xiezefan

<!-- more -->
## 代码

废话不多说，先上代码（**注**：只在 Gradle Android Plugin 1.5.0 测试过）

``` Groovy build.gradle
import com.android.build.gradle.AppPlugin
import com.android.build.gradle.LibraryPlugin
import proguard.gradle.ProGuardTask

apply plugin: 'com.android.application'

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.2"

    defaultConfig {
        applicationId "org.chaos.demo.jar"
        minSdkVersion 19
        targetSdkVersion 22
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
}

//dependsOn 可根据实际需要增加或更改
task buildJar(dependsOn: ['compileReleaseJavaWithJavac'], type: Jar) {

    appendix = "demo"
    baseName = "androidJar"
    version = "1.0.0"
    classifier = "release"

    //后缀名
    extension = "jar"
    //最终的 Jar 包名，如果没设置，默认为 [baseName]-[appendix]-[version]-[classifier].[extension]
    archiveName = "AndroidJarDemo.jar"

    //需打包的资源所在的路径集
    def srcClassDir = [project.buildDir.absolutePath + "/intermediates/classes/release"];
    //初始化资源路径集
    from srcClassDir

    //去除路径集下部分的资源
//    exclude "org/chaos/demo/jar/MainActivity.class"
//    exclude "org/chaos/demo/jar/MainActivity\$*.class"
//    exclude "org/chaos/demo/jar/BuildConfig.class"
//    exclude "org/chaos/demo/jar/BuildConfig\$*.class"
//    exclude "**/R.class"
//    exclude "**/R\$*.class"

    //只导入资源路径集下的部分资源
    include "org/chaos/demo/jar/**/*.class"

    //注: exclude include 支持可变长参数
}

task proguardJar(dependsOn: ['buildJar'], type: ProGuardTask) {
    //Android 默认的 proguard 文件
    configuration android.getDefaultProguardFile('proguard-android.txt')
    //manifest 注册的组件对应的 proguard 文件
    configuration project.buildDir.absolutePath + "/intermediates/proguard-rules/release/aapt_rules.txt"
    configuration 'proguard-rules.pro'

    String inJar = buildJar.archivePath.getAbsolutePath()
    //输入 jar
    injars inJar
    //输出 jar
    outjars inJar.substring(0, inJar.lastIndexOf('/')) + "/proguard-${buildJar.archiveName}"

    //设置不删除未引用的资源(类，方法等)
    dontshrink

    Plugin plugin = getPlugins().hasPlugin(AppPlugin) ?
            getPlugins().findPlugin(AppPlugin) :
            getPlugins().findPlugin(LibraryPlugin)
    if (plugin != null) {
        List<String> runtimeJarList
        if (plugin.getMetaClass().getMetaMethod("getRuntimeJarList")) {
            runtimeJarList = plugin.getRuntimeJarList()
        } else if (android.getMetaClass().getMetaMethod("getBootClasspath")) {
            runtimeJarList = android.getBootClasspath()
        } else {
            runtimeJarList = plugin.getBootClasspath()
        }

        for (String runtimeJar : runtimeJarList) {
            //给 proguard 添加 runtime
            libraryjars(runtimeJar)
        }
    }
}
```

### 使用方法

不需要混淆则运行命令

	gradle buildJar
	或
	./gradlew buildjar

需要混淆则运行

	gradle proguardJar
	或
	./gradlew proguardJar

## 最后

**buildJar** 这部分相对比较简单，很多内容网上都有教程。关键在于混淆，由于团队每个人都有自己的安装习惯，JDK、Android SDK 路径不一定一致，并不能直接写死 runtime 的路径，最后直接看 Android Plugin 源码才写出了 **proguardJar** task。

至于想更多个性化的朋友，建议从源码入手。

文章相关代码放置于 [Github](https://github.com/ChaosLeong/AndroidSimples/blob/master/AndroidJarDemo/build.gradle)。

