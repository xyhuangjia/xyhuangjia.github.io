---
title: "证书过期之后"
date: 2017/06/05
categories: 杂谈
---

## 业务场景

> 公司的企业版app又双无法下载了。依然记得一年前误删发布的场景。。。。ヽ(*。>Д<)o゜。。。现在想起还一身冷汗，下班开始公司电话一直响个不停一遍遍的跟客户解释出现问题的原因，真是噩梦般的回忆。。。

如果证书已到期或已撤销，会出现什么情况?<!-- more -->
 
## 资料 

**源于[Develop官网说明](https://developer.apple.com/support/certificates/cn/)现摘录如下：**
 
 > - ### Apple Push Notification Service 证书
 您将无法向您的 app 发送推送通知。
 > - <h3>Pass Type ID 证书 (Passbook)</h3>
 如果您的证书到期，用户设备上已安装的通行证将继续正常工作。但是，您不能再签署新的通行证或向现有通行证发送更新。如果证书已撤销，您的通行证将无法正常工作。
 > - <h3>iOS 分发证书 (App Store)</h3>
 如果您的 Apple Developer Program 会员资格有效，则您在 App Store 上的现有 app 将不受影响。但是，您不能再向 App Store 提交新的 app 或更新。
 > - <h3>iOS 分发证书（企业内部、内部使用 App）</h3>
 用户不能再运行已使用此证书签名的 app。您必须发布使用新证书签名的新版本 app。
 > - <h3>Mac App 分发证书和 Mac Installer 分发证书 (Mac App Store)</h3>
 如果您的 Apple Developer Program 会员资格有效，则您在 Mac App Store 上的现有 app 将不受影响。但是，您不能再向 Mac App Store 提交新的 app 或更新。
 > - <h3>Developer ID 应用软件证书和 Developer ID 安装程序证书（Mac 应用软件）</h3>
 如果您的证书到期，用户仍然可以下载、安装和运行使用该证书签名的 Mac 应用软件版本。但是，您需要使用新的证书来为更新和新应用软件签名。如果您的证书已撤销，用户将不能再安装使用该证书签名的应用软件。   

## 证书到期之间的app版本处理
查阅了下相关资料。貌似除了在过期时间到来之前发新版App没有别的有效处理方案。[你可以同时让两个证书处于活跃状态，它们之间相互独立。第二个证书是为了提供一个重叠期，让您能够在第一个证书过期前更新您的应用程序。从 iOS Dev Center（iOS 开发中心）请求您的第二个分发证书时，请确保您没有撤销第一个证书。](http://www.cnblogs.com/xiaonanxia/archive/2013/04/24/3040567.html)

## 最后弱鸡的解决方案

记录证书到期时间将更新事件加入日历中，到指定日志通知手动更新到期证书。当然您有更加优良的方案可以加qq2587171762联系我或者在下方留言交流下。
