---
title: 一次失败的next主题升级之旅
date: 2018-10-10 09:28:00
tags:
categories: "杂谈"
---

## 起因

内心极度忐忑，静不下心来工作，想找一一些与工作无关的事情来缓解下心情。看到next 的主题都升级6X了，于是乎决定升级一下当前的主题。<!-- more -->

## 经过

首先克隆新的 v6.x 仓库到任一异于 `next` 的目录（如 `next-reloaded`）：
```shell
git clone https://github.com/theme-next/hexo-theme-next themes/next-reloaded
```
然后在 Hexo 的主配置文件中设置主题：
```
...
theme: next-reloaded
...
```

在不修改原有的 NexT v5.1.x 目录的同时使用 `next-reloaded` 目录中的新版本主题。
到这里你开心的调试环节了。
终端键入`hexo s`
but 接下来作为一个外语不太好的你来讲
![](hexo-theme-next-update/01.jpg)
好眼熟，貌似不是音格利息啊。毛玩意？乱码？

不要惊慌，出现这个主要是文本的语言有问题，`language: zh-Hans`->`language: zh-CN`
这时你以为你改完了？no no no ,找个有图的文章试试看？
我的图尼？我的图尼？我的图尼？
安装插件吧骚年[fancybox3](https://github.com/theme-next/theme-next-fancybox3)，然而你安装后还是没个卵用，为啥？我也不知道，但是我比对其他的大佬的图片路径后得知，这个插件把图片放在一个新的images文件中才可以访问。（PS：你也可以放在第三方托管平台）。
其他的部分小问题在conmfig文件中有链接讲更新处理方式，个人不再一一赘 叙。

## 结论

更换主题，见仁见智，就我个人而言更换当前主题虽然成功了，但这个是一件得不偿失的事情。我要想使用新的主题需要对当前的改动牵扯太多太大。不太划算。