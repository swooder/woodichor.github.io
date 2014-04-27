---
layout: post
title: "Handy adb commands for Android"
date: 2014-03-24 23:05:52 +0800
comments: true
categories: Android
---
####查看连接的设备
使用这个命令查看当前连接的所有设备和它们的ID

``adb devices``

如果你有多个设备连接，使用``adb -s DEVICE_ID``选中你需要操作的设备


####安装应用
使用``install``命令安装apk，如果你已经安装过当前程序，并且想保存当前的数据，可以加上``-r```参数来覆盖安装当前程序

``
adb 
``
记录下在使用android adb操作时常用的命令，可以方便我们在开发过程中构建和测试程序。


####卸载程序
```
adb uninstall PACKAGE_NAME
//example
adb uninstall com.shaojie.name.example
```

####启动一个Activity
```
adb shell am start PACKAGE_NAME/ACTIVITY_IN_PACKAGE
adb shell am start PACKAGE_NAME/FULLY_QUALIFIED_ACTIVITY
//example
adb shell am start -n com.shaojie.name.example/.MainActiivty
adb shell am start -n com.shaojie.name.example/com.shaojie.name.MainActivity
```
####进入设备的shell
```
adb shell
```

####截屏
```
adb shell screencap -p | perl -pe 's/\x0D\x0A/\x0A/g' > screen.png
```

####点击电源键
这个命令发送一个关闭屏幕的事件到设备
```
adb shell input keyevent 26
```

####屏幕解锁
与上面的命令作用相反，用于点亮屏幕解锁
```
adb shell input keyevent 82
```

####显示安装的程序包
```
adb shell pm list packages -f
```

####Logging
显示系统的日志到命令行
```
adb logcat
```

####TagName过滤
```
adb logcat -s TAG_NAME
adb logcat -s TAG_NAME_1 TAG_NAME_2
//
adb logcat -s TEST
adb logcat -s TEST MYAPP
```

####优先级过滤
显示某个级别以及以上优先级的日志
```
adb logcat "*:PRIORITY"
//example
adb logcat "*:W"
```
* V-Verbose
* D_Debug
* I-Info
* W-Warning
* E-Error
* F-Fatal
* S-Silent(最高级别)

####优先级和tagname过滤
```
adb logcat -s TAG_NAME:PRIORITY
adb logcat -s TAG_NAME_1:PRIORITY TAG_NAME_2:PROPRITY
//example
adb logcat -s TEST: W
```

####grep过滤
```
adb logcat | grep "SEARCH_TERM"
adb logcat | grep "SEARCH_TERM_1\|SEARCH_TERM_2"
//example
adb logcat | grep "Exception"
adb logcat | grep "Exveption\|Error"
```
####清除logcat日志
```
adb logcat -c
```

####更多
更加细节，当然是官方文档，[传送门](http://developer.android.com/tools/help/adb.html)






