title: Android Gradle 获取 APK 签名状态及对齐状态
date: 2016-05-08 01:35:37
categories: 
- Android
tags:
- 技巧
- Gradle
---

最近由于公司 CI 对所有项目做规范化处理，现有项目都需要按照规范修改 APK 的文件名。 由于 CI 只需要已签名并已对齐的 APK，所以需要获取 APK 的签名状态及资源对齐状态。

在 StackOverflow 及 Google 上搜索了一番只找到 zipAlign 的代码，没办法只能去看 ApplicationVariant 的代码。费了些时间才找到获取签名状态的 API，遂记录之。

<!-- more -->

代码如下：（已在 `com.android.tools.build:gradle` 1.5.0，2.0.0 及 2.1.0 验证）

``` Groovy
android.applicationVariants.all { variant ->
    if (variant.apkVariantData.signed){
        println "signed"
    }
    variant.outputs.each { output ->
        if (output.zipAlign){
            println "aligned"
        }
        println output.outputFile
    }
}
```

PS: 有没有更简便的方法可以查看 Android Gradle Build Tools 的 API 呢？ Google 给出的 DSL 文档也只有一部分 API……求各位大大科普
