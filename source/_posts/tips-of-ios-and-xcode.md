---
title: iOS开发Tips(持续更新）·····
date: 2016-07-15 16:21:06
tags:
---

**本文旨在总结iOS开发中一些小技巧，帮助更高效进行代码编写**

## 随机色
<!-- more -->
1、 纯代码开发中便于展示一些控件所在空间位置来使用随机色，代码如下：

```
/**
 *  测试用随机色
 *
 *  @return UIColor
 */
+(UIColor*)RandomColor{
        CGFloat hue = ( arc4random() % 256 / 256.0 ); //0.0 to 1.0
        CGFloat saturation = ( arc4random() % 128 / 256.0 ) + 0.5; // 0.5 to 1.0,away from white
        CGFloat brightness = ( arc4random() % 128 / 256.0 ) + 0.5; //0.5 to 1.0,away from black
        return  [UIColor colorWithHue:hue saturation:saturation brightness:brightness alpha:1];
}
```
## Xcode插件（已废弃）

2、 使用开发工具插件来进行开发，使用一些优秀的插件开发能达到事半功倍的的效果。在此强烈推荐xCode插件管理工具：Alcatraz，流程如下：

- 关闭xcode。
- 安装过Alcatraz,删除之,执行如下命令,否则直接跳过进入下一环节
```
rm -rf ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins/Alcatraz.xcplugin
```
-  重要一环
```
find ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins -name Info.plist -maxdepth 3 | xargs -I{} defaults write {} DVTPlugInCompatibilityUUIDs -array-add defaults read /Applications/Xcode.app/Contents/Info DVTPlugInCompatibilityUUIDsudo xcode-select --reset
```
- 安装命令
```
curl -fsSL https://raw.github.com/supermarin/Alcatraz/master/Scripts/install.sh | sh
```
- 重启Xcode 并点击Load  bundles按钮（[点了Skip Bundles看这里](http://www.jianshu.com/p/0a4766c41171)）


## 配置文件管理

3、删除多余配置文件

```shell
~/Library/MobileDevice/Provisioning Profiles 
```
现在推荐使用天狐大佬的工具[ProfilesManager](**https://github.com/shaojiankui/ProfilesManager**)

## 自行封装网络框架

4、集成网络框架，便于以后替换更新见如下链接：[iOS开发中关于网络框架应用的理解](http://www.jianshu.com/p/d18e70f58fe6)（水文，不看也罢）

## 字符串长度处理

5、字符串长度处理，开发中很多时候需要对字符进行控制，一般情况我会采用string 的length判定但是length在判定时候无论针对字符还是数字判断都将长度设定为1，这样的和话在需求要求是满足一定量的字符的情况下就不是很精确了,处理方式参见[输入框字符控制](http://www.jianshu.com/p/778f2da4b173)

## 日志输出，显示输出位置

6、DBUG输出(MK框架截取的代码)：

```
/**
 *  DEBUG输出
 */
#ifdef DEBUG
#   define DLog(fmt, ...) {NSLog((@"%s [Line %d] " fmt), __PRETTY_FUNCTION__, __LINE__, ##__VA_ARGS__);}
#   define ELog(err) {if(err) DLog(@"%@", err)}
#else
#   define DLog(...)
#   define ELog(err)
#endif
```
## Xcode 卡慢，升级设备吧骚年

5、安装cocopods后打开xcode7.3卡慢问题：
把 Source Control 里面的 Automatically 全部关掉。如下图
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1939330-6e7fa59586d58b90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 代码提示不见了

Q：iOS开发中最害怕什么？

A：Xcode抽风！！！

### 方案一：

百分之九十的问题可以用重启来解决。。。。

1. 退出 Xcode
2. 重启电脑
3. 找到 这个 DerivedData 文件夹 删除 

```shell
#路径
~/Library/Developer/Xcode/DerivedData)
```

1. 删除这个 com.apple.dt.Xcode 文件 

```shell
#路径 
~/Library/Caches/com.apple.dt.Xcode)
```

  然后运行 Xcode  就好了~~

 然而小学生表示这并没有什么卵用

### 方案二

打开终端工具执行下面的代码并重启Xcode

```shell
defaults  write com.apple.dt.XCode IDEIndexDisable 0
```

然并卵

其实我想说的是方法我忘记了😅![](tips-of-ios-and-xcode/01.jpg)