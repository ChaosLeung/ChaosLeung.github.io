title: Activity 数据恢复时移除 Fragment 数据
date: 2016-05-14 21:55:53
categories:
- Android
tags: 
- 小知识
---

某些场景下（如内存不足），系统销毁 Activity 时会调用 **onSaveInstanceState()**，而 Fragment 也会在此方法中保存自身的状态。之后用户重新打开对应的 Activity 时，系统则会通过 **onCreate(Bundle)** 或者 **onRestoreInstanceState(Bundle)** 恢复 Activity 的状态，而 Fragment 的状态会在 **onCreate(Bundle)** 中恢复。

某些特殊需求下，我们并不需要 Fragment 恢复之前的状态，那么就需要在 Fragment 数据恢复前移除 Fragment 的数据。首先我们来看一下 Activity 中是如何恢复 Fragment 数据的：<!-- more -->

``` java
static final String FRAGMENTS_TAG = "android:fragments";

protected void onCreate(Bundle savedInstanceState) {
    ...
    if (savedInstanceState != null) {
        Parcelable p = savedInstanceState.getParcelable(FRAGMENTS_TAG);
        mFragments.restoreAllState(p, mLastNonConfigurationInstances != null
                ? mLastNonConfigurationInstances.fragments : null);
    }
    mFragments.dispatchCreate();
    ...
}
```

从代码中可以看到，Activity 通过读取 **savedInstanceState** 中对应 key 为 **FRAGMENTS_TAG** 的 Parcelable 对象的值来恢复 Fragment 的状态。

所以我们可以通过这点入手来阻止 Fragment 状态恢复，代码如下：

原生：

``` java 原生 Fragment
savedInstanceState.remove("android:fragments");
```

Support 包：

``` java Support 包的 Fragment
savedInstanceState.remove("android:support:fragments");
```

## 总结

实际环境下这种方法用的不多，只要不是奇葩需求的情况下，一般都希望恢复 Fragment 的状态。假设某个 Activity 中有两个 Fragment，B Fragment 显示时覆盖 A。如果某个时刻显示的是 B，并且此时 Activity 因为内存不足被销毁，恢复的时候我们当然也希望显示的仍是 B。  
但如果真遇到恢复数据时需要移除 Fragment 数据的场景，上面的方法比较实用。

最后，衷心祝福正在看这篇文章你永远不会遇到这样的需求。
