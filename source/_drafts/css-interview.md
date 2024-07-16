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






























