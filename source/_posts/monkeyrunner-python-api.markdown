---
layout: post
title: "monkeyrunner python API"
date: 2015-02-16 22:27:15 +0800
updated: 2015-02-17 14:02:06 +0800
comments: true
categories:
- 测试
tags:
- 测试
- Android
---

## 简单介绍
目前Android自带的 `Monkey`、`UiAutomator`、`MonkeyRunner` 主要特点如下：

* Monkey：准确来说，Monkey 并不算自动化测试，它只能产生随机事件流，并不能按照既定的步骤操作（建议别跑日常用的机子，重新改回设置都有的烦）；<!-- more -->
* UiAutomator：简单实用，可以通过 `text`、`hint`或 `contentDescription` 属性的值获取控件，还有配套的 `uiautomatorviewer`让测试人员读取到控件中上述三个属性的值，可以做到黑盒测试；(不支持id操作或许能算是个缺点，但我认为真正意义上的黑盒测试，测试人员并不需要知道控件的id)；
* MonkeyRunner：操作最为简单，只需使用 `MonkeyRecorder` 即可录制脚本。其中 `MonkeyDevice`仅支持坐标操作控件，可移植性不强，但由于可用 id 操作控件的 `EasyMonkeyDevice` 的增补，使得 MonkeyRunner 更实用。

本文主要讲述的是**MonkeyRunner**。

## MonkeyRunner 详细用法
思前想后觉得这部分还是不写了，网上很多资料，无谓赘述，附上官方文档 [**monkeyrunner**](https://developer.android.com/tools/help/monkeyrunner_concepts.html)。顺便提一点，MonkeyRunner 的脚本使用 **`Python`** 编写，但也可以使用 Java 来写，具体做法自行搜索。

## MonkeyRunner API
[**MonkeyDevice**](https://developer.android.com/tools/help/MonkeyDevice.html)、[**MonkeyImage**](https://developer.android.com/tools/help/MonkeyImage.html)、[**MonkeyRunner**](https://developer.android.com/tools/help/MonkeyRunner.html) 已有官方文档，所以在这只介绍 **EasyMonkeyDevice** 的 Python API

约定：因为以下示例常用到 id，所以示例代码中 `id=By.id('id/view')`

返回值|方法|描述|示例
-|-|-
|EasyMonkeyDevice(MonkeyDevice device)|构造函数|device = MonkeyRunner.waitForConnection()  eDevice = EasyMonkeyDevice(device)
void|touch(By selector, TouchPressType type)|模拟 touch 事件|eDevice.touch(id,eDevice.DOWN_AND_UP)
void|type(By selector, String text)|为控件（selector）输入文字|eDevice.type(id,'newText')
tuple|locate(By selector)|获取控件的坐标及宽高，返回结果格式为(x,y,w,h)|locate = eDevice.locate(id)
boolean|exists(By selector)|检查控件是否存在，true 为存在|isExists = eDevice.exists(id)
boolean|visible(By selector)|检测控件是否可见，true 为可见|isVisible = eDevice.visible(id)
String|getText(By selector)|获取可输入控件的 text|text = eDevice.getText(id)
String|getFocusedWindowId()|获取焦点所在控件的 id|idStr = getFocusedWindowId()

## 最后
一个简单的 Demo

``` Python demo.py
from com.android.monkeyrunner import MonkeyRunner, MonkeyDevice
from com.android.monkeyrunner.easy import EasyMonkeyDevice, By
# from com.android.monkeyrunner.recorder import MonkeyRecorder as recorder

device = MonkeyRunner.waitForConnection()
# 录制脚本
# recorder.start(device)

package = 'chaos.demo'
runComponent = package + '/.MainActivity'

device.startActivity(component = runComponent)

eDevice=EasyMonkeyDevice(device)
eDevice.type(By.id('id/edittext'),'test')
MonkeyRunner.sleep(2)
eDevice.touch(By.id('id/notification'),eDevice.DOWN_AND_UP)
MonkeyRunner.sleep(0.5)
device.press('KEYCODE_BACK','DOWN_AND_UP')
MonkeyRunner.sleep(0.5)
device.press('KEYCODE_BACK','DOWN_AND_UP')
```

保存后命令行输入：

	monkeyrunner demo.py