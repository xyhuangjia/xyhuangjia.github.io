---
title: Swift问题
tags:
---
#### Objective-C与Swift的异同？
**共同点** 
 - OC出现过的绝大多数概念，比如引用计数、ARC（自动引用计数）、属性、协议、接口、初始化、扩展类、命名参数、匿名函数等，在Swift中继续有效（可能最多换个术语）。
 - Swift和Objective-C共用一套运行时环境，Swift的类型可以桥接到Objective-C（下面我简称OC），反之亦然。<!-- more -->

**Swift的优点**
 - Swift注重安全，OC注重灵活
 - Swift注重面向协议编程、函数式编程、面向对象编程，OC注重面向对象编程
 - Swift注重值类型，OC注重指针和引用
 - Swift是静态类型语言，OC是动态类型语言
 - Swift容易阅读，文件结构和大部分语法简易化，只有.swift文件，结尾不需要分号
 - Swift中的可选类型，是用于所有数据类型，而不仅仅局限于类。相比于OC中的nil更加安全和简明
 - Swift中的泛型类型更加方便和通用，而非OC中只能为集合类型添加泛型
 - Swift中各种方便快捷的高阶函数（函数式编程） (Swift的标准数组支持三个高阶函数：map，filter和reduce,以及map的扩展flatMap)
 - Swift新增了两种权限，细化权限。open > public > internal(默认) > fileprivate > private
 - Swift中独有的元组类型(tuples)，把多个值组合成复合值。元组内的值可以是任何类型，并不要求是相同类型的。 
- Swift中函数可以当作参数或返回值，OC中不可以。

#### 类(class) 和 结构体(struct) 有什么区别? 类(class) 和 结构体(struct) 比较,优缺点?
二者的本质区别：`struct`是深拷贝，拷贝的是内容；`class`是浅拷贝，拷贝的是指针。
`struct`是值类型，`class`是引用类型。
（值类型的变量直接包含它们的数据，对于值类型都有它们自己的数据副本，因此对一个变量操作不可能影响另一个变量。
引用类型的变量存储对他们的数据引用，因此后者称为对象，因此对一个变量操作可能影响另一个变量所引用的对象。）

相同点：
1.都可以定义以下成员：属性、方法、下标、初始化器
2.都支持类型扩展、协议

不同点：
1.类支持继承和多态，结构体不支持
2.类必须自己定义初始化器，结构体会有默认的按成员初始化器
3.类支持析构器（deinit），结构体不支持
4.类的实例在堆上，由ARC负责释放；结构体的实例在栈上，栈结束自动释放，不参与ARC管理
5.变量赋值方式不同：struct是值拷贝；class是引用拷贝
6.immutable变量：swift的可变内容和不可变内容用var和let来甄别，如果初始为let的变量再去修改会发生编译错误。struct遵循这一特性；class不存在这样的问题
7.mutating function： struct 和 class 的差別是 struct 的 function 要去改变 property 的值的时候要加上 mutating，而 class 不用。
8.类支持引用相等比较（===于!==）,结构体不支持

#### Swift 中的枚举,关联值 和 原始值的区分
**关联值**:有时会将枚举的成员值跟其他类型的变量关联存储在一起，会非常有用
```swift
// 关联值
enum Date {
  case digit(year: Int, month: Int, day: Int)
  case string(String)
}
```
**原始值**:枚举成员可以使用相同类型的默认值预先关联，这个默认值叫做:原始值
```swift
// 原始值
enum Grade: String {
  case perfect = "A"
  case great = "B"
  case good = "C"
  case bad = "D"
}
```
#### Swift中, 存储属性和计算属性的区别?
1.存储属性(Stored Property):
类似于成员变量这个概念
存储在实例对象的内存中
结构体、类可以定义存储属性
枚举不可以定义存储属性

2.计算属性(Computed Property):
本质就是方法(函数)
不占用实例对象的内存
枚举、结构体、类都可以定义计算属性
```swift
struct Circle {
    // 存储属性
    var radius: Double
    // 计算属性
    var diameter: Double {
        set {
            radius = newValue / 2
        }
        get {
            return radius * 2
        }
    }
}
```