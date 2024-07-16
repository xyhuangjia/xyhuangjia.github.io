---
title: Texture的读书笔记
tags:
---

`Texture`原名`AsyncDisplayKit`，是FaceBook为解决页面卡顿提供方案。他自己重新实现一整套异步布局和渲染机制来达到预期结果。

该框架从一下三个方面来优化

>- **渲染**，对于大量图片，或者大量文字（尤其是CJK字符）混合在一起时。而文字区域的大小和布局，恰恰依赖着渲染的结果。ASDK尽可能后台线程进行渲染，完成后再同步回主线程相应的UIView。
>
>- **布局**。ASDK完全弃用了Autolayout，另辟蹊径实现了自己的布局和缓存机制。关于布局的问题会在下一篇讲到。
>
>- 系统objects的**创建与销毁**。由于UIKit封装了CALayer以支持触摸等显示以外的操作，耗时也相应增加。而这些同样也需要在主线程上操作。ASDK基于Node的设计，突破了UIKit线程的限制。



对于一般UIView和CALayer来说，因为他们不是线程安全的，任何相关操作都需要在主线程进行。ASDK引入了Node的概念来解决UIView/CALayer只能在主线程上操作的限制。

> ### ASDisplayNode主要特点：
>
> 1. 每个Node对应相应的UIView或者CALayer，从开发者的角度而言，只需要将初始化UIView的代码稍作修改，替换为创建ASDisplayNode即可。在不需要接受用户操作的Node上可以开启isLayerBacked，直接使用CALayer进一步降低开销。根据Scott的研究UIView的开销大约是CALayer的5倍。
> 2. Node**默认是异步布局/渲染**，只有在需要将frame/contents等同步到UIView上才会回到主线程，使其空出更多的时间处理其他事件。
> 3. ASDK只有在认为需要的时候才会异步地为Node加载相应的View，因此创建Node的开销变得非常低。同时Node是**线程安全**的，可以在任意queue上创建和设置属性。
> 4. ASDK不仅有与UIView对应的大部分控件（如ASButtonNode、ASTextNode、ASImageNode、ASTableNode等等），同时也bridge了大多数UIView的方法和属性，可以非常方便的操作frame/backgroundColor/addSubnode等，因此一般情况下只要对Node进行操作，ASDK就会在适当的时候同步到其View。如果需要的话，当相应的View加载之后（或访问node.view手动触发加载），也可以通过node.view的方式直接访问，回到我们熟悉的UIKit。
> 5. 当实现自定义View的时候，ASDisplayNode提供了一个初始化方法initWithViewBlock/initWithLayerBlock，就可以将任意UIView/CALayer用Node包裹起来（被包裹的view可以使用autolayout），从而与ASDK的其他组件相结合。虽然这样创建的Node与一般view在布局和渲染上的差异不大，但是由于Node管理着何时何地加载view，我们仍然能得到一定的性能提升。

### 注意事项

1. ASDK不支持Storyboard和Autolayout，但是可以与使用Autolayout的view兼容共存。同样React native和Component Kit等其他Facebook出品的iOS库也不支持Storyboard。
2. 由于Node的异步渲染，很有可能在其View到达屏幕之后，内容仍然在渲染过程中。此时需要额外考虑每个Node的placeholder状态，使用户不至于看到一片空白。
3. 在使用ASDisplayNode初始化initWithViewBlock时，由于Node需要在适当的时候调用该block来创建view，因此并不会立即调用block（block可能capture其他变量，例如self），而是存在一个ivar当中。如果该view始终没被创建，而此时拥有该node的父元素被销毁，容易造成retain cycle导致memory leak。