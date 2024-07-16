---
title: 写一个正确的单例
date: 2018-04-16 00:00:00
tags:
---


### 业务场景

> 最近面试时候遇到小哥问一个问题？如何去写一个能够在随时随地获取准确值的单例？当时我的表情是这样的
>  ![](create-safe-singleton/黑人问号.jpg)
> 以我祖传开发经验写单例的方法的的莫不是 gcd 里的 disptach_once 然后加两个方法啥的 balala 啥？这样会出现问题？
>  ![](create-safe-singleton/震惊.jpg) 
>
> 结果是大佬说了三点说了如下来规避上述问题 
> （1）线程安全，多个线程同时访问线程竞速（非GCD写法）。
> （2）init方法，避免初始化被重写。
> （3）防止被继承。

<!-- more -->

​     大佬解释的很细致，然而我觉得自己觉得他说的貌似有那么些问题。自己实验做的少，没有发言权。没底气亦无力去反驳，晚饭前复盘时觉得有必要将这个弄清楚。

### 是什么？
#### 单例是什么
单例模式，也叫单子模式，是一种常用的软件设计模式。在应用这个模式时，单例对象的类必须保证只有一个实例存在。这个实例在类的内部构造,外部不能直接调用其构造方法,只能获取它的实例。

### 为什么？

#### 为什么要用单例？
- 节省内存开销。如果某个对象需要被多个其它对象使用，那可以考虑使用单例，因为这样该类只使用一份内存资源。
- 使用单例，可以确保其它类只获取类的一份数据。

#### 单例的优点缺点在哪？
##### 优点
 - 在单例模式中，活动的单例只有一个实例，对单例类的所有实例化得到的都是相同的一个实例。这样就 防止其它对象对自己的实例化，确保所有的对象都访问一个实例 
 - 单例模式具有一定的伸缩性，类自己来控制实例化进程，类就在改变实例化进程上有相应的伸缩性。 
 - 提供了对唯一实例的受控访问。 
 - 由于在系统内存中只存在一个对象，因此可以 节约系统资源，当 需要频繁创建和销毁的对象时单例模式无疑可以提高系统的性能。 
 - 允许可变数目的实例。 
 - 避免对共享资源的多重占用。 

##### 缺点
 - 由于单利模式中没有抽象层，因此单例类的扩展有很大的困难。
 - 单例类的职责过重，在一定程度上违背了“单一职责原则”。
 - 滥用单例将带来一些负面问题，如为了节省资源将数据库连接池对象设计为的单例类，可能会导致共享连接池对象的程序过多而出现连接池溢出；如果实例化的对象长时间不被利用，系统会认为是垃圾而被回收，这将导致对象状态的丢失。
### 怎么做？

#### 使用dispatch_once之前的做法
```objective-c
static MHCSingletonClass *singleton =  nil;

+ (instancetype)allocWithZone:(struct _NSZone *)zone{
    if (singleton == nil) {
        @synchronized(self) {
            if (singleton == nil) {
                singleton = [super allocWithZone:zone];
            }
        }
    }
    return singleton;
}

+ (instancetype)sharedInstance{
    if (singleton == nil) {
        @synchronized(self) {
            if (singleton == nil) {
                singleton = [[self alloc] init];
            }
        }
    }
    return singleton;
}

- (id)copyWithZone:(NSZone *)zone{
    return singleton;
}
- (id)mutableCopyWithZone:(NSZone *)zone{
    return singleton;
}
```
#### 使用dispatch_once之前的做法


#### 如何去实现一个稳定可靠的单例对象
```objective-c
static MHASingletonClass * singleton = nil;

+ (instancetype)allocWithZone:(struct _NSZone *)zone{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        singleton = [super allocWithZone:zone ];
    });
    return singleton;
}

+ (instancetype) sharedInstance{
    if (singleton == nil) {
        singleton = [[super alloc]init];
    }
    return singleton;
}

- (id)copyWithZone:(NSZone *)zone{
    return singleton;
}

- (id)mutableCopyWithZone:(NSZone *)zone{
    return singleton;
}
```
demo 是不可能给demo的




