---
title: Autolayout和布局第三方库
tags:

---

### AutoLayout小结

#### AutoLayout什么

> Auto Layout ，是苹果公司提供的一个基于约束布局，动态计算视图大小和位置的库.集成于XCode

#### AutoLayout实现

> Auto Layout拥有一套Layout Engine引擎，由它来主导页面的布局。App启动后，主线程的Run Loop会一直处于监听状态，当约束发生变化(1)后会触发Deffered Layout Pass（延迟布局传递），在里面做容错处理（约束丢失等情况）并把view标识为dirty状态，然后Run Loop再次进入监听阶段。当下一次刷新屏幕动作来临（或者是调用layoutIfNeeded）时，Layout Engine 会从上到下调用 layoutSubviews() ，通过 Cassowary算法计算各个子视图的位置，算出来后将子视图的frame从Layout Engine拷贝出来，接下来的过程就跟手写frame是一样的了。

- 添加、删除视图时会触发约束变化。Activating 或 Deactivating，设置 Constant 或 Priority 时也会触发约束变化

#### Auto Layout 性能问题

Cassowary 是以高效的界面线性方程求解算法被提出来的。它解决的是界面的线性规划问题。它能够对界面进行高效添加和修改更新操作。

在iOS 12 以前对页面视图之间关系不复杂时候autolayout性能不会出现指数级影响。但是兄弟视图之间有关联的话就会出现所述问题，其原因是

> iOS 12 之前，很多约束变化时都会重新创建一个计算引擎 NSISEnginer 将约束关系重新加进来，然后重新计算。结果就是，涉及到的约束关系变多时，新的计算引擎需要重新计算，最终导致计算量呈指数级增加.



#### 什么时候用

简单布局用AutoLayout，复杂布局还是要Frame，修改和更新更清晰