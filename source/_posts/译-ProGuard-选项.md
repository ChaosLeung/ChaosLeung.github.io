title: '[译] ProGuard 选项'
date: 2015-12-03 23:15:54
tags: proguard
categories: 工具
---

> 注：只针对 Android 平台翻译了 ProGuard 选项的相关用法，更多信息请查看[官方原文](http://proguard.sourceforge.net/)。

## Input/Output Options
**@filename**  
简短版 `-include filename`

**-include** *filename*  
读取指定文件的配置参数

<!-- more -->

**-basedirectory** *directoryname*  
指定当前配置文件中所有相对文件路径的基础目录

**-injars** *class_path*  
指定需要处理的 jar/aar/war/ear/zip/apk/dir。文件(目录)内的类文件会被处理并写入到输出的文件(目录)。默认情况下，非类文件会直接复制而不会被修改。在 *class_path* 中的内容可以过滤，在 filters 部分会详细说明。为了更好的可读性，建议使用多个 **-injars** 来指定多个 *class_path*

**-outjars** *class_path*  
指定处理完后输出的 jar/aar/war/ear/zip/apk/dir 名字。前面指定的 **-injars** 会在处理后写入到该文件(目录)中。可以过滤输出内容，在 filters 部分会详细说明。  
需要避免输出文件覆盖输入文件。没有 **-outjars** 配置时，不会有输出文件。

**-libraryjars** *class_path*  
指定相关必需的 jar/aar/war/ear/zip/apk/dir 类库。这些文件不会包含在输出的文件中。在 *class_path* 中的内容可以过滤，在 filters 部分会详细说明。为了更好的可读性，建议使用多个 **-libraryjars** 来指定多个 *class_path*

**-skipnonpubliclibraryclasses**  
指定读取类库时跳过非公有类，该操作可以加快处理及减少 ProGuard 的内存使用。默认情况下 ProGuard 会读取非公有类。然而，大多时候非公有类不会与 injars 有关联，如果它不会影响到 injars 的文件，建议使用该选项。如果使用该选项导致找不到类，ProGuard 会输出相应的警告信息。

**-dontskipnonpubliclibraryclasses**  
指定不忽略非公有的 library 类。4.5 版后，该选项默认开启。

**-dontskipnonpubliclibraryclassmembers**  
指定不忽略包内可见的 library 类的成员(字段及方法)。默认情况下，ProGuard 解析类库时会跳过这些成员，因为 injars 一般不会使用它们。然而有时 injars 与 library 的类同属一个包，并且 jnjars 的类使用了这些包内可见成员。在这种情况下，这些类成员是有用的。

**-keepdirectories** [*directory_filter*]  
指定要保留在输出文件内的目录。默认情况下，目录会被移除。这会减少输出文件的大小，但如果你的代码引用到它们时可能会导致程序崩溃（例：`mypackage.MyCalss.class.getResource("")`）。这时就需要指定 `-keepdirectories mypackage`。如果没有指定过滤器，所有目录会被保留。例如，`-keepdirectories mydirectory` 匹配 mydirectory 目录；`-keepdirectories mydirectory/*` 匹配 mydirectory 的直接子目录；`-keepdirectorie mydirectory/**` 匹配所有子目录。

**-target** *version*  
指定类文件的版本号。版本号可以是 1.0，1.1，1.2，1.3，1.4，1.5(or 5)，1.6(or 6)，1.7(or 7)，1.8(or 8)。默认情况下，类文件的版本号保持不变。一般不应降低类文件的版本号，因为可能包含部分旧版本不支持的代码。

**-forceprocessing**  
指定即使输出是一样的，也强制处理 injars。一致性判断是基于指定的输入，输出和配置文件或配置文件夹的时间戳比较的。

## Keep Options

**-keep** [,*modifier*,...] *class_specification*  
保留指定的类及类成员

**-keepclassmembers** [,*modifier*,...] *class_specification*  
保留指定的类的成员

**-keepclasseswithmembers** [,*modifier*,...] *class_specification*  
保留满足指定条件的类和类成员

**-keepnames** *class_specification*  
简短版 `-keep,allowshrinking class_specification`  
如果指定的类和类成员在压缩期没被移除，则保留它们的名称。只有开启混淆时可用。

**-keepclassmembernames** *class_specification*  
简短版 `-keepclassmembers,allowshrinking class_specification`  
如果指定的类的成员在压缩期没被移除，则保留它们的名称。只有开启混淆时可用。

**-keepclasseswithmembernames** *class_specification*  
简短版 `-keepclasseswithmembers,allowshrinking class_specification`  
如果满足指定条件的类和类成员在压缩期没被移除，则保留它们的名称。只有开启混淆时可用。

**-printseeds** [*filename*]  
详尽地列出类和类成员匹配的 -keep 选项清单，标准输出到指定的文件中。该清单可用于验证预期的类和类成员是否真正被找到，特别是使用了通配符的情况下。

## Keep Option Modifiers

**includedescriptorclasses**  
指定保留 -keep 选项保留的方法、字段的类型描述符对应的类。通常用于保留 Native 方法。

**allowshrinking**  
允许压缩

**allowoptimization**  
允许优化

**allowobfuscation**  
允许混淆

## Shrinking Options

**-dontshrink**  
指定不压缩输入的类文件，默认开启。被指定 -keep 选项的及其直接或简直依赖的类和类成员外均会被移除。压缩也会发生在优化阶段后，因为优化后可能会使类和类成员存在压缩的可能性。

**-printusage** [*filename*]  
列出没有被使用的类和类成员，标准输出到指定的文件中。只有开启压缩时可用。

**-whyareyoukeeping** *class_specification*  
输出指定的类和类成员在压缩步骤被保留的原因。一般情况下，可能会存在多个原因。该选项会输出到目标的最短方法链。如果开启了 -verbose 选项，最短链会包含字段及方法签名。只有开启压缩时可用。

## Optimization Options

**-dontoptimize**  
指定不优化输入的类文件，默认开启。所有的方法进行字节码级优化。

**-optimizations** *optimization_filter*  
开启或关闭更细粒度的优化。只有开启优化时可用。高级选项。

**-optimizationpasses** *n*  
指定优化次数，默认执行一次优化。多次优化可能会得到更好的结果。如果某次优化后没有变化，优化会自动结束。只有开启优化时可用。

**-assumenosideeffects** *class_specification*  
假定某些方法可以移除。在优化步骤，如果可以确定返回值没被使用，ProGuard 会移除这些方法。ProGuard 会分析你的代码以查找相关调用。例如：指定 `System.currentTimeMillis()`，任何返回值没有被使用的调用均会删除。某些情况下，你可以使用该选项删除日志代码。只有开启优化时可用。一般情况下该操作比较危险，可能轻易导致程序崩溃，**请明确该操作的影响时使用该选项**

**-allowaccessmodification**  
优化时允许扩大类和类成员的访问修饰符。该选项可以改善优化的结果。例如，内联公有 getter 可能需要将字段也改成公有。只有开启优化（并开启了 `-repackageclasses`）。

**-mergeinterfacesaggressively**  
指定接口可以合并，即使实现类没实现所有的方法。该选项可以通过减少类的总数减少输出文件的大小。只有开启优化时可用。

## Obfuscation Options

**-dontobfuscate**  
指定不混淆类文件，默认开启。

**-printmapping** [*filename*]
输出类和类成员新旧名字之间的映射到指定文件中。只有开启混淆时可用。

**-applymapping** *filename*  
重用映射，映射文件未列出的类和类成员会使用随机的名称。如果代码结构从根本上发生变化，ProGuard 可能会输出映射会引起冲突警告。你可以通过添加 `-useuniqueclassmembernames` 选项来降低风险。只能指定一个映射文件。只有开启混淆时可用。

**-obfuscationdictionary** *filename*  
使用文件中的关键字作为方法及字段混淆后的名称。默认使用 'a'，'b' 等短名称作为混淆后的名称。你可以指定保留关键字或不相关的标识符。文件中的空格、标点符号、重复的单词及注释会被忽略。只有开启混淆时可用。

**-classobfuscationdictionary** *filename*  
使用文件中的关键字作为类混淆后的名称，类似于 `-obfuscationdictionary`。只有开启混淆时可用。

**-packageobfuscationdictionary** *filename*  
使用文件中的关键字作为包混淆后的名称，类似于 `-obfuscationdictionary`。只有开启混淆时可用。

**-overloadaggressively**  
开启侵入性重载混淆。多个字段及方法允许同名，只要它们的参数及返回值类型不同。该选项可使处理后的代码更小（及更难阅读）。只有开启混淆时可用。  
注：Dalvik 不能处理重载的静态字段

**-useuniqueclassmembernames**  
方法同名混淆后亦同名，方法不同名混淆后亦不同名。不使用该选项时，类成员可被映射到相同的名称。因此该选项会增加些许输出文件的大小。只有开启混淆时可用。

**-dontusemixedcaseclassnames**  
混淆时不会产生大小写混合的类名。默认混淆后的类名可以包含大写及小写。如果 jar 被解压到非大小写敏感的系统（比如 Windows），解压工具可能会将命名类似的文件覆盖另一个文件。只有开启混淆时可用。

**-keeppackagenames** [*package_filter*]  
不混淆指定的包名。过滤器是由逗号分隔的包名列表。包名可以包含 ？、\*、\*\* 通配符，并且可以在包名前加上 ! 否定符。只有开启混淆时可用。

**-flattenpackagehierarchy** [*package_name*]  
重新打包所有重命名的包到给定的包中。如果没参数或字符串为空，包移动到根包下。该选项是进一步混淆包名的例子，可以使处理后的代码更小更难阅读。只有开启混淆时可用。

**-repackageclasses** [*package_name*]  
重新打包所有重命名的类到给定的包中。如果没参数或字符串为空，类的包会被完全移除。该选项覆盖 `-flattenpackagehierarchy` ，是进一步混淆包名的另一个例子，可以使处理后的代码更小更难阅读。曾用名为 `-defaultpackage`。只有开启混淆时可用。

**-keepattributes** [*attribute_filter*]  
保留任何可选属性。过滤器是由逗号分隔的 JVM 及 ProGuard 支持的属性列表。属性名可以包含 ？、\*、\*\* 通配符，并且可以在属性名前加上 ! 否定符。例如：处理库文件时应该加上 `Exceptions`,`InnerClasses`,`Signature` 属性。同时保留 `SourceFile` 及 `LineNumberTable` 属性使混淆后仍能获取准确的堆栈信息。同时如果你的代码有使用注解你可能会保留 `annotations` 属性。只有开启混淆时可用。

**-keepparameternames**  
保留已保留方法的参数的名称及类型。只有开启混淆时可用。

**-renamesourcefileattribute** [*string*]  
指定一个常量字符串作为 `SourceFile`（和 `SourceDir`）属性的值。需要被 `-keepattributes` 选项指定保留。只有开启混淆时可用。

**-adaptclassstrings** [*class_filter*]  
混淆与完整类名一致的字符串。没指定过滤器时，所有符合现有类的完整类名的字符串常量均会混淆。只有开启混淆时可用。

**-adaptresourcefilenames** [*file_filter*]  
以混淆后的类文件作为样本重命名指定的源文件。没指定过滤器时，所有源文件都会重命名。只有开启混淆时可用。

**-adaptresourcefilecontents** [*file_filter*]  
以混淆后的类文件作为样本混淆指定的源文件中与完整类名一致的内容。没指定过滤器时，所有源文件中与完整类名一致的内容均会混淆。只有开启混淆时可用。

## Preverification Options

**-dontpreverify**  
指定不对处理后的类文件进行预校验。默认情况下如果类文件的目标平台是 Java Micro Edition 或 Java 6 或更高时会进行预校验。目标平台是 Android 时没必要开启，关闭可减少处理时间。

**-microedition**  
指定处理后的类文件目标平台是 Java Micro Edition。

## General Options

**-verbose**  
指定处理期间打印更多相关信息。

**-dontnote** [*class_filter*]  
指定配置中潜在错误或遗漏时不打印相关信息。类名错误或遗漏选项时这些信息可能会比较有用。*class_filter* 是一个可选的正则表达式。类名匹配时 ProGuard 不会输出这些类的相关信息。

**-dontwarn** [*class_filter*]  
指定找不到引用或其他重要问题时不打印警告信息。*class_filter* 是一个可选的正则表达式。类名匹配时 ProGuard 不会输出这些类的相关信息。  
注意：如果找不到引用的类或方法在处理过程中是必须的，处理后的代码将会无法正常运行。**请明确该操作的影响时使用该选项**。

**-ignorewarnings**  
打印找不到引用或其他重要问题的警告信息，但继续处理代码。
注意：如果找不到引用的类或方法在处理过程中是必须的，处理后的代码将会无法正常运行。**请明确该操作的影响时使用该选项**。

**-printconfiguration** [*filename*]  
将已解析过的配置标准输出到指定的文件。该选项可用于调试配置。

**-dump** [*filename*]  
标准输出类文件的内部结构到给定的文件中。例如，你可能要输出一个 jar 文件的内容而不需要进行任何处理。