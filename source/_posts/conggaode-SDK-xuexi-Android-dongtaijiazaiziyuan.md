title: 从高德 SDK 学习 Android 动态加载资源
date: 2015-10-25 18:04:45
tags: 插件开发
categories: Android
---

前不久跑去折腾高德 SDK 中的 HUD 功能，相信用过该功能的用户都知道 HUD 界面上的导航转向图标是动态变化的。从[高德官方导航 API 文档][1]中 AMapNaviGuide 类的描述可知，导航转向图标有23种类型。

诶，等等，23 种？那图标应该是放在 assets 文件夹吧？总不可能是在服务器上下载吧？  
<!-- more -->
看下导航 API 的 jar 包结构。

	AMap_ Navi_v1.3.0_20150828.jar
	  |- assets
	    |- autonavi_Resource1_1_0.png
	    |- custtexture*.png (7 张)
	  |- com
	    |- amap.api.navi
	    |- autonavi
	  |- META-INF

纳尼？assets 上的图片总共也只有 8 张，而且图片的内容跟 HUD 毫无关系，莫非真的是从服务器下载资源？  
用 Android Studio 打开 jar 包中的 AMapHudView.class 来看下 AMapHudView 的逻辑（AS 1.2 就引入了反编译功能）。

``` java
...
import com.autonavi.tbt.g;
...

public class AMapHudView extends FrameLayout implements OnClickListener, OnTouchListener, e {

	static final int[] hud_imgActions = new int[]{2130837532, 2130837532, 2130837532, 2130837533, 2130837534, 2130837535, 2130837536, 2130837537, 2130837538, 2130837539, 2130837522, 2130837523, 2130837524, 2130837525, 2130837526, 2130837527, 2130837528, 2130837529, 2130837530, 2130837531};
	...
	private ImageView roadsignimg;// 方向图标对应的 View
	...
	private int resId;// 方向图标的 id，对应 hud_imgActions 的 index，根据高德的文档，该变量值为 0-23
	...
	private void updateHudWidgetContent() {
        ...
        if(this.roadsignimg != null && this.resId != 0 && this.resId != 1) {
            Drawable var1 = g.a().getDrawable(hud_imgActions[this.resId]);// g.a() 返回的是 Resource 对象
            this.roadsignimg.setBackgroundDrawable(var1);
            ...
        }
    }
}
```

先看 `hud_imgActions`，里面的值是不是很熟悉？转成16进制均为 0x7F02 开头（0x7F 是应用资源，而 0x02 则是 drawable 资源）。再看 `updateHudWidgetContent()` 方法，逻辑比较简单，通过 `resId` 获取 `hud_imgActions` 对应的 drawable id，再通过该 id 获取到对应的 Drawable 对象并将其设置到 ImageView 中。

看到这，可以肯定高德 SDK 最终是通过本地资源的索引获取到 Drawable。

然而我们的 apk 中并没有相应的资源，为什么能够正常获取到对应的 Drawable？我们看回上面的第12行代码：

``` java
Drawable var1 = g.a().getDrawable(hud_imgActions[this.resId]);// g.a() 返回的是 Resource 对象
```

我们将注意力集中到 `g.a()` 中，找到 `com.autonavi.tbt.g#a()`

``` java
public static Resources a() {
    if (b == null) {
        b = e.getResources();
    }
    return b;
}
```

其中变量 `e` 为上层传递进来的 Activity，而我们前面说过，我们的 apk 中并没有相应的资源，所以将注意力放到变量 `b` 在其他地方的赋值上。

``` java
public static boolean a(Context context) {
    ...
    a = b(context.getFilesDir() + "/autonavi_Resource1_1_0.jar");
    b = a(context, a);// 变量 a 为 AssetManager
    return true;
}

private static AssetManager b(String str) {
    try {
        Class cls = Class.forName("android.content.res.AssetManager");
        AssetManager assetManager = (AssetManager) cls.getConstructor().newInstance();
        try {
            cls.getDeclaredMethod("addAssetPath", String.class).invoke(assetManager, str);
        } catch (Throwable th) {
        }
        return assetManager;
    } catch (Throwable th2) {
        return null;
    }
}

private static Resources a(Context context, AssetManager assetManager) {
    DisplayMetrics displayMetrics = new DisplayMetrics();
    displayMetrics.setToDefaults();
    return new Resources(assetManager, displayMetrics, context.getResources().getConfiguration());
}
```

可以看到，高德 SDK 中先通过反射实例化 AssetManager，并且调用 addAssetPath(context.getFilesDir() + "/autonavi\_Resource1\_1\_0.jar")，接着实例化 Resources 对象。所以事实上是通过这个新的 Resource 来获取到对应资源的 Drawable 对象。  
但是我们的 apk 对应的 files 目录中并不存在 autonavi\_Resource1\_1\_0.jar，这个文件又是怎么来的？

``` java
private static String k = "autonavi_Resource1_1_0.png";
...
private static boolean b(Context var0) {
	String filePath = var0.getFilesDir().getAbsolutePath() + "/autonavi_Resource1_1_0.jar";
	...
	InputStream var1 = var0.getResources().getAssets().open(k);
	File var3 = new File(filePath);
	long var21 = var3.length();
	int var6 = var1.available();
	if(!var3.exists() || var21 != (long)var6) {
	    ...
	    File var22 = new File(filePath);
	    FileOutputStream var2 = new FileOutputStream(var22);
	    byte[] var8 = new byte[1024];
	
	    int var9;
	    while((var9 = var1.read(var8)) > 0) {
	        var2.write(var8, 0, var9);
	    }
	}
	...
}
```

还是 com.autonavi.tbt.g 这个类，可以看到，高德是将 jar 包内 assets 目录中的 autonavi\_Resource1\_1\_0.png 复制到当前 apk 对应的 files 目录中，并将新的文件命名为 autonavi\_Resource1\_1\_0.jar。

再回到加载资源的问题上，为什么加载 autonavi\_Resource1\_1\_0.jar 能索引资源？  
因为该文件其实是 apk（高德将后缀名改成了 jar）。AssetManager 加载该 apk 后，Resource 就能通过该 AssetManager 获取到里面的相应资源。

> AssetManager 的相关知识请参考老罗的[《Android应用程序资源管理器（Asset Manager）的创建过程分析》][2]

至此，我们就可以清楚知道高德 SDK 是如何实现动态加载资源的：

1. 将资源 apk 放置在 jar 包的 assets 目录中；
2. 在 View 组件初始化的过程中将 assets 中的资源 apk 复制到 files 目录中；
3. 接着实例化 AssetManager，调用 addAssetPath 方法加载 files 目录中的资源 apk；
4. 然后将 AssetManager 作为参数实例化 Resouce，最后通过 Resource 对象获取资源apk 中相应的资源。

## <span id="summary">总结</span>

将上述内容再简略，动态加载资源所必需的几个核心步骤：

1. 实例化 AssetManager 对象，并通过反射调用 addAssetPath(String) 方法加载目标 apk（或与 apk 文件架构一致的目录）
2. 通过第一步得到的 AssetManager 实例化 Resource 对象
3. 利用第二步得到的 Resource 对象来动态加载资源

这里需要注意的是，目标 apk（目录）需要放在 `context.getFilesDir()` 中，不然会加载失败（addAssetPath 返回 0）。另外，目标 apk 可以不签名，因为 addAssetPath 过程并没有进行签名校验。

### 获取资源 id

实际情况中，如果我们需要获取相应的资源，就必须先获得资源对应的 id，而外部 apk 的 R.java 并不属于主 apk，这就导致了获取资源的困难。  
目前存在的解决方案有：

1. 通过反射对应的 R 类获取对应的 id（极力不推荐，需要知道 field 的 name，若资源 apk 需要混淆，field name 就更不知道是什么了，再者反射的效率并不理想）
2. 通过接口获取对应的 id（优点在于灵活性高，主 apk 不需要关心资源。缺点在于若需要的资源较多，处理也较多。更多出现在获取固定资源的场景中，譬如应用换肤）
3. 直接将资源 apk 的 R.java 放在主 apk 中，通过 R 获取 id（简单粗暴，但若资源 apk 中存在对应的 R.java，会发生冲突。混淆过则不存在这个问题。该方案缺乏灵活性，需要开发人员知道需要的资源名，对应的属性等。）

最后两种方案各有各的优缺点，至于怎么选择，还得结合自身的场景。

### 应用场景

动态加载资源技术目前的一些应用场景主要有：

1. 替换应用皮肤（如：QQ 空间）
2. 减小主 apk 的大小，非重要资源放在服务端
3. 类似于文中高德 SDK 的做法，使得 jar 包可以加载资源（这种应用可能现在比较少，以前这种做法也只是因为还没 aar）

### 后续

动态加载资源技术相关文章有很多，但就我目前所看到的文章只涉及如何获取 drawable、string 等资源，并没有发现关于动态加载资源 apk 中的布局文件（我姿势不对？`_(:зゝ∠)_`）。后续会分享如何动态加载资源 apk 中的布局文件。

最后特别感谢 [Andy Zhang](https://github.com/luckyandyzhang)，[MadisonRong](https://github.com/MadisonRong) 两位朋友帮忙校对并对文章提出了宝贵的意见，谢谢。

[1]: http://lbs.amap.com/Public/reference/AMap_Android_Navi_Doc_V1.4.0_20151015/
[2]: http://blog.csdn.net/luoshengyang/article/details/8791064

---
参考文章：

[《Android中插件开发篇之----应用换肤原理解析》][1]

[1]: http://blog.csdn.net/jiangwei0910410003/article/details/47679843