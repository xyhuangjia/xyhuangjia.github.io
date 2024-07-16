---
title: 面向面试之-卡顿优化
date: 2020-08-26 07:03:24
tags:
---

### 卡顿出现原因？

首先CPU计算显示内容，例如视图创建，布局计算、图片解码、文本绘制等；接着 CPU 会将计算好的内容提交到 GPU进行合成、渲染。随后 GPU 会把渲染结果提交到帧缓冲区去，等待VSync 信号到来时显示到屏幕上。如果此时下一个VSync 信号到来时，CPU或GPU都没有完成相应的工作时，则那一帧将会丢弃。在每秒低于30帧时候人眼能感知卡顿存在。<!-- more -->

卡顿造成的原因通常是CPU和GPU导致的掉帧引起的，主要原因如下：

1. 主线程在进行大量I/O操作：为了方便代码编写，直接在主线程去写入大量数据；
2. 主线程在进行大量计算：代码编写不合理，主线程进行复杂计算；
3. 大量UI绘制：界面过于复杂，UI绘制需要大量时间；
4. 主线程在等锁：主线程需要获得锁A，但是当前某个子线程持有这个锁A，导致主线程不得不等待子线程完成任务。

### 如何优化？

#### CPU层面

- 对象创建：对象的创建会分配内存、调整属性、甚至还有读取文件等操作，比较消耗CPU资源。尽量采取轻量级对象，尽量放到后台线程处理，尽量推迟对象的创建时间。（如UIView / CALayer）

- 对象调整：frame、bounds、transform及视图层次等属性调整很耗费CPU资源。尽量减少不必要属性的修改，尽量避免调整视图层次、添加和移除视图。

- 布局计算：随着视图数量的增长，Autolayout带来的CPU消耗会呈指数级增长，所以尽量提前算好布局，在需要时一次性调整好对应属性。

- 文本渲染：屏幕上能看到的所有文本内容控件，包括UIWebView，在底层都是通过CoreText排版、绘制为位图显示的。常见的文本控件，其排版与绘制都是在主线程进行的，显示大量文本是，CPU压力很大。对此解决方案唯一就是自定义文本控件，用CoreText对文本异步绘制。（很麻烦，开发成本高）

- 图片解码：当用UIImage或CGImageSource创建图片时，图片数据并不会立刻解码。图片设置到UIImageView或CALayer.contents中去，并且CALayer被提交到GPU前，CGImage中的数据才会得到解码。这一步是发生在主线程的，并且不可避免。SD_WebImage处理方式：在后台线程先把图片绘制到CGBitmapContext中，然后从Bitmap直接创建图片。

- 图像绘制：图像的绘制通常是指用那些以CG开头的方法把图像绘制到画布中，然后从画布创建图片并显示的一个过程。CoreGraphics方法是线程安全的，可以异步绘制，主线程回调。

- 控制一下线程的最大并发数量

#### GPU层面

- 纹理混合：尽量减少短时间内大量图片的显示，尽可能将多张图片合成一张进行显示。GPU能处理的最大纹理尺寸是4096x4096，一旦超过这个尺寸，就会占用CPU资源进行处理，所以纹理尽量不要超过这个尺寸
- 视图混合：尽量减少视图层次和数量，减少透明的视图（alpha<1），不透明的就设置opaque为YES。
- 图形生成：尽量避免离屏渲染，尽量采用异步绘制，尽量避免使用圆角、阴影、遮罩等属性。必要时用静态图片实现展示效果，也可尝试光栅化缓存复用属性。



### UITableview 优化方案

**优化一、Cell重用、标识重用**

使用 `static` 修饰重用标识名称能够保证这个标识只会创建一次，提高性能。接着调用`dequeueReusableCellWithIdentifier:方法获取缓存池中的Cell`。如果没有就调用 `initWithStyle:ReusIdentifier:`方法创建一个新的Cell。注意事先需要调用`registerNib/registerClass`方法为TableView注册一下重用标识。

**优化二、设置预估行高，预先缓存动态行高**

设置预估行高

UITableView通过设置UITableView代理方法`heightForRowAtIndexPath:`方法来设置行高。自从iOS8.0之后，苹果新增了`self-sizing cell`的概念，也是cell可以自己计算行高，使用需要满足三个条件：

1. 使用Autolayout进行UI布局约束
2. 指定TableView的`estimatedRowHeight`属性的默认值
3. 指定TableView的rowHeight的属性为`UITableViewAutomaticDimension`。

TableView在加载数据时会先通过`estimatedHeightForRowAtIndexPath`处理全部数据，此时我们只需要提供一个粗略的高度，待到cell对象创建之后再去设置cell的真实高度。而且只会处理当前屏幕范围内的cell，这样子会显著的提升加载的性能。

```objective-c
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {  
    return 50.0;  
}  

- (CGFloat)tableView:(UITableView *)tableView estimatedHeightForRowAtIndexPath:(NSIndexPath *)indexPath {  
    return 30.0;  
} 
复制代码
```

预先计算并缓存行高

iOS8.0之后获取cell对象之后会再次调用`heightForRowAtIndexPath: `方法获取行高，这也就意味着我们其实可以先创建cell对象，之后再提供行高。具体方法我们可以在cell类中添加`layoutAttribute`属性，记录相应的`UIEdgeInsets`，然后在设置cell真实高度的时候返回。iOS7.0之前则是必须在cell对象创建之前先获得所有Cell的高度。

**优化三、减少Subviews层级、异步绘制、避免离屛渲染、使用hidden隐藏图层**

1. 减少图层层级数
2. 异步绘制。通过重写本身是异步的drawReact:方法，调用Core Graphics 框架中的API 进行异步绘制，提高效率。另外drawRect:中大量的绘制操作也会造成内存的增长，可以使用[CAShapeLayer]()来代替。
3. 减少多余的绘制操作。在实现drawRect:方法的时候，他的参数rect就是我们需要绘制的区域，在rect范围之外的区域不要绘制，否则会消耗相当大的资源。
4. 图片异步加载并及时释放缓存。在Cell类中添加图片应该避免使用imageWithName:方法，因为该方法会将图片缓存到内存中。而是应该使用imageWithContensOfFile:方法来替换，该方法在图片使用完后系统会自动释放资源，并不会缓存下来。另外结合[SDWebImage]()框架的使用可以显著的提高图片加载的性能。
5. 避免动态添加图层。在初始化cell的时候一并将所有图层预先创建好，通过hidden属性控制子图层的显示或隐藏，因为单纯的显示操作要比创建快的多。
6. 避免[离屛渲染]()。开启离屛渲染的代价就是需要新开辟一块新的缓冲区，在渲染的过程中还会多次的切换上下文，这些都是很消耗性能的。以下情况均会造成离屛渲染。

> - 为图层设置遮罩（layer.mask）
> - 设置图层的 layer.masksToBounds/view.clipsToBounds属性为True
> - 设置图层的 layer.allowsGroupOpacity的属性为True和layer.opacity小于1.0
> - 设置图层阴影（layer.shadow）
> - 设置图层的 layer.shouldRasterize的属性为True
> - 具有 layer.cornerRadius, layer.edgeAntialiasingMask, layer.allowsAntialiasing的图层
> - 文本（任何种类，包括UILabel、CATextLayer、Core Text等）
> - 使用CGContext在drawReact:方法中绘制

​	7.图片圆角优化

使用贝塞尔曲线 + Core Graphics框架设置圆角

```
- (void)setImageCircularEdge:(UIImageView *)imageView {  

    //开始对imageView进行画图  
    UIGraphicsBeginImageContextWithOptions(imageView.bounds.size, NO, 1.0);  
    //使用贝塞尔曲线画出一个圆形图  
    [[UIBezierPath bezierPathWithRoundedRect:imageView.bounds cornerRadius:imageView.frame.size.width] addClip];  
    [imageView drawRect:imageView.bounds];  
    imageView.image = UIGraphicsGetImageFromCurrentImageContext();  
    //结束画图  
    UIGraphicsEndImageContext();  
} 
```

使用贝塞尔曲线 + CAShapeLayer 设置圆角

```
- (void)setImageCircularEdge2:(UIImageView *)imageView {  

    UIBezierPath *maskPath = [UIBezierPath bezierPathWithRoundedRect:imageView.bounds byRoundingCorners:UIRectCornerAllCorners cornerRadii:imageView.bounds.size];  
    CAShapeLayer *maskLayer=[[CAShapeLayer alloc] init];  
    //设置大小  
    maskLayer.frame = imageView.bounds;  
    //设置图形样子  
    maskLayer.path = maskPath.CGPath;  
    imageView.layer.mask = maskLayer;  
}  
```

​	8.0图片阴影优化

```
- (void)setImageShadow:(UIImageView *)imageView {  

    imageView.layer.shadowColor = [UIColor grayColor].CGColor;  
    imageView.layer.shadowOpacity = 1.0;  
    imageView.layer.shadowRadius = 2.0;  
    UIBezierPath *path=[UIBezierPath bezierPathWithRect:imageView.frame];  
    imageView.layer.shadowPath = path.CGPath;  
}
```

**优化四、分页加载数据，预先异步请求数据**

**优化五、滑动TableView时，按需加载内容**

有些情况下我们可能会去快速的滑动列表，这时候其实会有大量的cell对象被创建、被重用，但其实我们可能只是去浏览列表停止的那一页的上下一定范围内的信息，前面快速划过的那些信息对我们来说都是无用的。此时我们可以通过ScrollView的代理方法

```
scrollViewWillEndDragging: withVelocity: targetContentoffset:
复制代码
```

来按需加载内容。

```
#pragma mark - UIScrollViewDelegate  
//按需加载 - 如果目标行与当前行相差超过指定行数，只在目标滚动范围的前后指定3行加载。  
- (void)scrollViewWillEndDragging:(UIScrollView *)scrollView withVelocity:(CGPoint)velocity targetContentOffset:(inout CGPoint *)targetContentOffset {  

    NSIndexPath *targetPath = [_myTableView indexPathForRowAtPoint:CGPointMake(0, targetContentOffset->y)];  
    NSIndexPath *firstVisiblePath = [[_myTableView indexPathsForVisibleRows] firstObject];  
    NSInteger skipCount = 8;  
    if (labs(firstVisiblePath.row - targetPath.row)>  skipCount) {  
        NSArray *temp = [_myTableView indexPathsForRowsInRect:CGRectMake(0, targetContentOffset->y, _myTableView.frame.size.width, _myTableView.frame.size.height)];  
        NSMutableArray *arr = [NSMutableArray arrayWithArray:temp];  
        if (velocity.y<0) {  
            NSIndexPath *indexPath = [temp lastObject];  
            if (indexPath.row+33) {  
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row-3 inSection:0]];  
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row-2 inSection:0]];  
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row-1 inSection:0]];  
            }  
        }  
        [_dataList addObjectsFromArray:arr];  
    }  
}  
复制代码
```

targetContentOffset 是TableView减速到停止的地方, velocity 表示速度向量。

**优化六、Cell类中应该避免请求网络加载数据**

如果确实有需求不可避免，可以将网络加载任务添加到Runloop中，设置DefaultRunloopModule模式。这样子可以起到延迟加载的作用。

**优化七、在willDisplayCell:forRowAtIndexPath:代理方法中绑定数据**

很多人都喜欢在cellForRowAtIndexPath：方法中绑定数据，然后此时的Cell其实还未显示，该方法中包含了大量的布局、绘制相关的操作。我们应该在该方法中尽量简化我们自身的逻辑操作。这时我们可以使用在willDisplayCell:forRowAtIndePath:方法中绑定数据。

```
#pragma mark - UITableViewDataSource  
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {  
    static NSString *cellIdentifier = @"MyTableViewCell";  
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellIdentifier];  
    if (!cell) {  
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellIdentifier];  
    }  

    return cell;  
}  

#pragma mark - UITableViewDelegate  
- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath {  

    NSDictionary *dict = self.dataList[indexPath];  
    [cell updateData:dict];  
} 
```



## 参考文档

[[UITableView性能优化 - 中级篇](https://segmentfault.com/a/1190000018161741)](https://segmentfault.com/a/1190000018161741)

[UITableView的优化技巧](https://www.dazhuanlan.com/2019/09/25/5d8b2dfd33d63/)

[iOS 性能优化-让UITableView如丝般顺滑（布局、离屏渲染、事务优化）](https://www.jianshu.com/p/6eea49f1f06e)

[iOS UITableView 优化面试竟然是这样子的](https://zhuanlan.zhihu.com/p/153056072)

[iOS手动实现重用池](https://zhuanlan.zhihu.com/p/126485234)

[UITableView 流畅度优化实践](https://blog.aberlt.com/2017/12/30/UITableView-流畅度优化实践/)

