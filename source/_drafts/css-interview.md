---
title: css-interview
tags:
---

 ### ::before 和 :after 的双冒号和单冒号有什么区别？
（1）冒号(:)用于CSS3伪类，双冒号(::)用于CSS3伪元素。
（2）::before就是以一个子元素的存在，定义在元素主体内容之前的一个伪元素。并不存在于dom之中，只存在在页面之中。
**注意：** :before 和 :after 这两个伪元素，是在CSS2.1里新出现的。起初，伪元素的前缀使用的是单冒号语法，但随着Web的进化，在CSS3的规范里，伪元素的语法被修改成使用双冒号，成为::before、::after。

### display:inline-block 什么时候会显示间隙？

(1)有空格时会有间隙，可以删除空格解决；
(2)margin正值时，可以让margin使用负值解决；
(3)使用font-size时，可通过设置font-size:0、letter-spacing、word-spacing解决；

### Sass、Less 是什么？为什么要使用他们？
他们都是 CSS 预处理器，是 CSS 上的一种抽象层。

为什么要使用它们？
- 结构清晰，便于扩展。 可以方便地屏蔽浏览器私有语法差异。封装对浏览器语法差异的重复处理， 减少无意义的机械劳动。
- 可以轻松实现多重继承。 完全兼容 CSS 代码，可以方便地应用到老项目中。LESS 只是在 CSS 语法上做了扩展，所以老的 CSS 代码也可以与 LESS 代码一同编译。

### 媒体查询的理解？

媒体查询（Media Queries）是CSS3引入的一个重要概念，它允许开发者根据不同的设备特性（如屏幕尺寸、分辨率、色彩深度、设备方向等）来应用特定的CSS样式规则。这使得网站能够提供更加优化的视觉体验，无论是在大屏幕的台式机、笔记本电脑上，还是在小屏幕的智能手机和平板电脑上。

```javascript
@media [not] <media_type> and (<media_feature>: <value>) {
    /* CSS rules */
}
```

其中各个部分的含义如下：
- `not` 是可选的关键词，用于否定媒体类型。
- `<media_type>` 是媒体类型，如 `all、print、screen` 等，表示目标输出设备的类型。
- `and` 连接符用于组合多个媒体特性。
- `<media_feature>` 是媒体特性，比如 `width、height、orientation、color` 等，它们描述了设备的物理属性。
- `<value>` 是与媒体特性相关的值，例如 `600px` 或者 `landscape`
媒体查询是响应式设计的核心组成部分，它使得Web开发人员能够创建灵活、自适应的布局，以适应不断变化的设备生态系统。

### 对 CSS 工程化的理解
CSS工程化是指在大型项目中，为了更好地管理和维护CSS代码，通过一系列的方法、工具和技术来组织、优化、构建和维护CSS的过程。随着前端项目的规模越来越大，CSS工程化变得尤为重要，它帮助开发者应对代码复用、模块化、性能优化、可维护性和团队协作等方面的挑战。

以下是CSS工程化的一些关键方面：
1. 宏观设计
- 目录结构：优化CSS文件的目录结构，通常按照功能模块或者页面组件进行分类，避免单一CSS文件过大。
- 模块化：将CSS代码划分为独立的模块，每个模块负责一个特定的功能或组件，减少样式间的依赖和冲突。
- 命名规范：采用一致的命名约定，如BEM（Block Element Modifier）、OOCSS（Object-Oriented CSS）或SMACSS（Scalable and Modular Architecture for CSS），以增强代码的可读性和可维护性。
2. 编码优化
- 预处理器：使用Sass、Less或Stylus等CSS预处理器，它们提供了变量、嵌套、函数和混合等功能，使CSS代码更具编程性。
- 代码简化：利用PostCSS插件自动添加浏览器前缀、优化选择器、移除冗余代码等，减少人工工作量并提高代码质量。
3. 构建流程
- 自动化构建：集成Webpack、Gulp或Grunt等工具，实现CSS的自动化构建，包括压缩、合并、前缀添加、源映射生成等，以优化最终的生产环境代码。
- 热更新：在开发环境中实时更新CSS，无需刷新页面即可看到样式更改的效果，提高开发效率。
4. 可维护性
- 文档：编写良好的文档和注释，确保代码易于理解和修改。
- 测试：使用视觉回归测试工具确保样式更改不会破坏现有设计。
- 版本控制：利用Git或其他版本控制系统管理CSS文件的更改历史，便于回滚和协作。
5. 性能优化
- 延迟加载：只在需要的时候加载特定的CSS，比如使用JavaScript按需加载样式。
- 关键路径CSS：将首屏渲染所需的CSS内联到HTML中，提高首屏加载速度。
6. 团队协作
- 代码审查：通过代码审查确保CSS遵循既定的规范和最佳实践。
- 共享资源：创建和维护一个共享的样式库或设计系统，确保团队成员使用统一的UI组件和样式。

通过这些策略，CSS工程化不仅提高了代码的质量和性能，还促进了团队协作，降低了维护成本，使前端开发变得更加高效和可持续。


### 如何判断元素是否到达可视区域
以图片显示为例：

- `window.innerHeight` 是浏览器可视区的高度；
- `document.body.scrollTop || document.documentElement.scrollTop` 是浏览器滚动的过的距离；
- `imgs.offsetTop` 是元素顶部距离文档顶部的高度（包括滚动条的距离）；
内容达到显示区域的：`img.offsetTop < window.innerHeight + document.body.scrollTop`;


###  z-index属性在什么情况下会失效
通常 z-index 的使用是在有两个重叠的标签，在一定的情况下控制其中一个在另一个的上方或者下方出现。z-index值越大就越是在上层。z-index元素的position属性需要是relative，absolute或是fixed。

z-index属性在下列情况下会失效：

- 父元素position为relative时，子元素的z-index失效。解决：父元素position改为absolute或static；
- 元素没有设置position属性为非static属性。解决：设置该元素的position属性为relative，absolute或是fixed中的一种；
- 元素在设置z-index的同时还设置了float浮动。解决：float去除，改为display：inline-block；

### CSS3中的transform有哪些属性
CSS3的transform属性允许对元素进行变形操作，这包括旋转、缩放、倾斜和移动等效果。


### 对Flex布局的理解及其使用场景


### 1. 为什么需要清除浮动？清除浮动的方式

**浮动的定义:** 非IE浏览器下，容器不设高度且子元素浮动时，容器高度不能被内容撑开。 此时，内容会溢出到容器外面而影响布局。这种现象被称为浮动（溢出）。
###### 浮动元素引起的问题？
- 父元素的高度无法被撑开，影响与父元素同级的元素
- 与浮动元素同级的非浮动元素会跟随其后
- 若浮动的元素不是第一个元素，则该元素之前的元素也要浮动，否则会影响页面的显示结构

###### 清除浮动的方式如下：

- 给父级div定义height属性
- 最后一个浮动元素之后添加一个空的div标签，并添加clear:both样式
- 包含浮动元素的父级标签添加overflow:hidden或者overflow:auto
- 使用 :after 伪元素。由于IE6-7不支持 :after，使用 zoom:1 触发 hasLayout**


#### 使用 clear 属性清除浮动的原理？

在CSS中，clear属性用于指定元素的哪一侧不允许出现浮动元素。当一个元素周围有浮动元素时（即，使用了float属性的元素），该元素可能会与这些浮动元素重叠，或者被推到浮动元素后面。clear属性可以防止这种情况发生，确保元素在浮动元素之下或之后开始。

clear属性可以接受以下值：
- none：默认值，允许所有方向的浮动。
- left：不允许左侧有浮动元素。
- right：不允许右侧有浮动元素。
- both：不允许左右两侧有浮动元素。
- inline-start：不允许内联起始位置有浮动元素（在从右到左的书写模式下，这相当于不允许右侧有浮动元素）。
- inline-end：不允许内联结束位置有浮动元素（在从右到左的书写模式下，这相当于不允许左侧有浮动元素）。
需要注意的是，clear属性只对块级元素生效，并且它会影响元素的垂直定位，但不会影响其水平位置。此外，在现代布局方法如Flexbox和Grid布局中，clear和float的使用已经减少，因为这些新方法提供了更强大、更灵活的布局控制。

### 对BFC的理解，如何创建BFC？
BFC，全称 Block Formatting Context（块级格式化上下文），是CSS中一种重要的渲染规则，它决定了页面上块级元素如何布局以及它们与其他元素之间的相互作用。BFC可以看作是一个独立的容器，其中的元素不会受到外部元素的影响，同样，BFC外部的元素也不会受到BFC内部元素的影响，除非它们直接与BFC边缘接触。

**BFC的创建条件如下：**

- 根元素（html）。
- 显式地使用overflow属性设置为非visible的值（如auto，scroll，hidden）的元素。
- 显式地使用display: flex或display: grid的元素。
- 显式地使用float属性（除了none）的元素。
- 显式地使用position: absolute或position: fixed的元素。
- 显式地使用display: inline-block或display: table-cell的元素。

**BFC的主要特性包括：**

- 隔离性：BFC内部的元素不会影响到BFC外部的元素，反之亦然。
- 浮动元素的影响：在BFC内部，浮动元素不会影响到后续的块级元素。在BFC边界之内，浮动元素会按照正常的流布局排列。
- 高度计算：BFC的高度会包含其内部的浮动元素，这样父元素就可以正确地计算其高度，避免高度塌陷的问题。
- 清除浮动：BFC可以用来清除浮动，使得内部的元素能够紧贴在底部而不会被浮动元素遮挡。
- 布局：BFC中的块级元素会按照自上而下的顺序堆叠，而不会与同级的其他BFC中的元素交互。

创建BFC的一个常见用途是解决浮动元素导致的父元素高度塌陷问题。通过将父元素设置为BFC（比如使用overflow: auto），可以确保父元素包含其内部的浮动子元素，从而正确地计算高度。

例如，为了防止高度塌陷，可以这样做：

```Css
.parent {
  overflow: auto; /* 或者使用 other values like 'hidden', 'scroll' */
}
```
```Html
<div class="parent">
  <div style="float: left;">Floating element</div>
  <!-- Other elements -->
</div>
```
在现代布局技术（如Flexbox和Grid）中，BFC的概念仍然重要，但实现布局的方式更为简单和直观。然而，在处理遗留代码或特定布局需求时，理解BFC的工作原理仍然是非常有用的。















































