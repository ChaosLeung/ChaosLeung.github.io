title: Android 动态加载 layout 资源
date: 2015-11-28 15:24:11
tags: 插件开发
categories: Android
---

> 在阅读前，请确保你已经了解动态加载资源的基本知识，相关知识请阅读 [《从高德 SDK 学习 Android 动态加载资源》][1]

[1]: http://chaosleong.github.io/blog/2015/10/25/conggaode-SDK-xuexi-Android-dongtaijiazaiziyuan/#summary

由于 LayoutInflater 在 inflate 的过程中会调用 Context 的 **getResources()**、**getTheme()** 方法，所以在进行动态加载 layout 资源前，我们需要修改上述方法的返回值。根据不同的情况，存在两种修改方法。<!-- more -->

> 注：**getTheme()** 的返回值不一定要修改，只是出于各种考虑，建议将 **getTheme()** 也一并修改。视个人情况而定吧…… \_(:зゝ∠)\_

如果宿主 apk 是我们自己控制的，那么可以直接重写 **getResources()**、**getTheme()** 方法，使宿主 Activity 直接返回客体 apk 的资源以及 Theme。

``` java
public class HostActivity extends Activity {

    Resources mPluginRes;
    Resources.Theme mPluginTheme;

    @Override
    public Resources getResources() {
        return mPluginRes != null ? mPluginRes : super.getResources();
    }

    @Override
    public Resources.Theme getTheme() {
        return mPluginTheme != null ? mPluginTheme : super.getTheme();
    }
}    
```

特殊情况下，如果宿主 apk 不在我们控制范围内，我们只能通过反射达到目的。

``` java
public class ClientView extends View {

    Theme mTheme;
    Theme mOriginTheme;
    Resource mResources;
    Resource mOriginResources;
    
    Field mResourcesField;
    Field mThemeField;
    
    Activity mHostActivity;
    
    // do something ...
    
    private void setup() {
        // initialize mResources ...
    
        mTheme = mResources.newTheme();
        // 如果需要使用其他 Theme 只需将 '.' 改为 '_'，例 Theme.DeviceDefault 对应的变量名为 Theme_DeviceDefault 
        mTheme.applyStyle(Class.forName("com.android.internal.R$style").getDeclaredField("Theme").getInt(null), true);
    
        setupField();
    
        inflate();
    }
    
    private void setupField() {
        mThemeField = Class.forName("android.view.ContextThemeWrapper").getDeclaredField("mTheme");
        mThemeField.setAccessible(true);
        mResourcesField = Class.forName("android.app.ContextImpl").getDeclaredField("mResources");
        mResourcesField.setAccessible(true);
    }
    
    private void inflate() {
        beforeInflate();
    
        // inflating ...
    
        afterInflate();
    }
    
    /**
     * 在 inflate 前，由于 LayoutInflater 需要调用 Context 的 getResources() 以及 getTheme()
     * 方法，所以我们需要反射修改对应的变量的值
     */
    private void beforeInflate() {
        Context baseContext = mHostActivity.getBaseContext();
        mOriginResources = (Resources) mResourcesField.get(baseContext);
        mOriginTheme = (Theme) mThemeField.get(mHostActivity);
        mResourcesField.set(baseContext, mResources);
        mThemeField.set(mHostActivity, mTheme);
    }
    
    /**
     * 在 inflate 后，因为宿主之后也需要使用到自身的 getResources() 以及 getTheme()
     * 所以我们在执行完 inflate 后需要还原这两个变量的值
     */
    private void afterInflate() {
        mResourcesField.set(baseContext, mOriginResources);
        mThemeField.set(mHostActivity, mOriginTheme);
    }
}
```

## 总结

一般的动态资源加载场景中，宿主 apk 以及客体 apk 都是遵从某种守则进行开发的，此时方法一即可满足动态加载 layout 的需求。  
方法二的应用场景则是以假设宿主不可能满足条件而使用，比较典型的就是高德地图以及百度地图的导航 SDK。这两家提供的都只是 jar 包，而且不能依赖开发者重写自己 app 的 Context 对象的 **getResources()**、**getTheme()** 方法，所以只能通过反射来达到 inflate 自定义 View。