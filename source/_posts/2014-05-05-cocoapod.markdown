---
layout: post
title: "Cocoapods安装和使用"
date: 2014-05-05 22:44:04 +0800
comments: true
categories: IOS
---
####CocoaPods是什么
当你开发iOS应用时，会经常使用到很多第三方开源类库，比如JSONKit，AFNetWorking等等。可能某个类库又用到其他类库，所以要使用它，必须得另外下载其他类库，而其他类库又用到其他类库，“子子孙孙无穷尽也”，这也许是比较特殊的情况。总之小编的意思就是，手动一个个去下载所需类库十分麻烦。另外一种常见情况是，你项目中用到的类库有更新，你必须得重新下载新版本，重新加入到项目中，十分麻烦。如果能有什么工具能解决这些恼人的问题，那将“善莫大焉”。所以，你需要 CocoaPods。

CocoaPods应该是iOS最常用最有名的类库管理工具了，上述两个烦人的问题，通过cocoaPods，只需要一行命令就可以完全解决，当然前提是你必须正确设置它。重要的是，绝大部分有名的开源类库，都支持CocoaPods。所以，作为iOS程序员的我们，掌握CocoaPods的使用是必不可少的基本技能了

####如何下载和安装CocoaPods

在安装CocoaPods之前，首先要在本地安装好Ruby环境

假如你在本地已经安装好Ruby环境，那么下载和安装CocoaPods将十分简单，只需要一行命令。在Terminator（也就是终端）中输入以下命令

```
sudo gem install cocoapods
pod setup
```
上面第二行执行时，会输出Setting up CocoaPods master repo，但是会等待比较久的时间。这步其实是Cocoapods在将它的信息下载到 ~/.cocoapods目录下，如果你等太久，可以试着cd到那个目录，用du -sh *来查看下载进度

如果你在终端中敲入这个命令之后，会发现半天没有任何反应,可以更新一下ruby的源：
```
gem sources --remove https://rubygems.org/
gem sources -a http://ruby.taobao.org/
gem sources -l
```

####如何使用CocoaPod
1. 利用CocoaPods，在项目中导入AFNetworking类库
	AFNetworking类库在GitHub地址是：https://github.com/AFNetworking/AFNetworking
 为了确定AFNetworking是否支持CocoaPods，可以用CocoaPods的搜索功能验证一下。在终端中输入：
```
pod search AFNetWorking
```
过几秒钟之后，你会在终端中看到关于AFNetworking类库的一些信息

``
-> AFNetworking (2.2.1)
   A delightful iOS and OS X networking framework.
   pod 'AFNetworking', '~> 2.2.1'
   - Homepage: https://github.com/AFNetworking/AFNetworking
   - Source:   https://github.com/AFNetworking/AFNetworking.git
   - Versions: 2.2.1, 2.2.0, 2.1.0, 2.0.3, 2.0.2, 2.0.1, 2.0.0-RC3, 2.0.0-RC2, 2.0.0-RC1, 2.0.0, 1.3.3, 1.3.2, 1.3.1, 1.3.0, 1.2.1, 1.2.0, 1.1.0, 1.0RC3, 1.0RC2, 1.0RC1, 1.0.1, 1.0, 0.9.2, 0.9.1, 0.9.0, 0.7.0, 0.5.1, 0.10.1, 0.10.0 [master repo]
   - Sub specs:
     - AFNetworking/Serialization (2.2.1)
     - AFNetworking/Security (2.2.1)
     - AFNetworking/Reachability (2.2.1)
     - AFNetworking/NSURLConnection (2.2.1)
     - AFNetworking/NSURLSession (2.2.1)
     - AFNetworking/UIKit (2.2.1)
``
这说明，AFNetworking是支持CocoaPods，所以我们可以利用CocoaPods将AFNetworking导入你的项目中。
先利用Xcode创建一个项目，在项目的目录下新建一个名为Podfile的文件
```
platform :ios, '7.0'
pod "AFNetworking", "~> 2.2.0"
```
这两句文字的意思是，当前AFNetworking支持的iOS最高版本是iOS 7.0, 要下载的AFNetworking版本是2.2.0。

这时候，你就可以利用CocoPods下载AFNetworking类库了。还是在终端中的当前项目目录下，运行以下命令：
```
pod install
```
运行完成后，使用CocoaPods生成的 .xcworkspace 文件来打开工程，而不是以前的 .xcodeproj 文件
每次更改了Podfile文件，你需要重新执行一次pod install命令

####.gitignore
当你执行pod install之后，除了Podfile外，cocoapods还会生成一个名为Podfile.lock的文件，你不应该把这个文件加入到.gitignore中。因为Podfile.lock会锁定当前各依赖库的版本，之后如果多次执行pod install 不会更改版本，要pod update才会改Podfile.lock了。这样多人协作的时候，可以防止第三方库升级把程序搞挂
 
 

