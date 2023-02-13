
 * [索引](#索引)
      * [guard 使用场景](#guard-使用场景)
      * [final关键词的用法](#final关键词的用法)
      * [递归枚举](#递归枚举)
      * [MemoryLayout](#memorylayout)
      * [关联值跟原始值的区别](#关联值跟原始值的区别)
      * [值类型与引用类型](#值类型与引用类型)
      * [map 与 flatmap 的区别](#map-与-flatmap-的区别)
      * [闭包](#闭包)
      * [逃逸闭包&amp;非逃逸闭包&amp;尾随闭包](#逃逸闭包非逃逸闭包尾随闭包)
      * [inout](#inout)
      * [required](#required)
      * [Any、AnyObject](#anyanyobject)
      * [访问控制(Access Control)](#访问控制access-control)
      * [内存管理](#内存管理)
      * [循环引用](#循环引用)
      * [闭包的循环引用](#闭包的循环引用)
      * [指针](#指针)
      * [dynamic(daɪˈnæmɪk)](#dynamicdaɪˈnæmɪk)
      * [dynamic framework 和 static framework 的区别是什么](#dynamic-framework-和-static-framework-的区别是什么)
      * [@objc与@objcMembers的区别](#objc与objcmembers的区别)
      * [什么是高阶函数](#什么是高阶函数)


### guard 使用场景
guard 和 if 类似, 不同的是, guard 总是有一个 else 语句, 如果表达式是假或者值绑定失败的时候, 会执行 else 语句, 且在 else 语句中一定要停止函数调用
例如
```swift
guard 1 + 1 == 2 else {
    fatalError("something wrong")
}
```
常用使用场景为, 用户登录的时候, 验证用户是否有输入用户名密码等
```swift
guard let userName = self.userNameTextField.text,
  let password = self.passwordTextField.text else {
    return
}
```
### final关键词的用法
**final**关键词的作用：它修饰的类、方法、变量是不能被继承或重写的，编译器会报错。另外，通过它可以显示指定函数的派发机制
 - 被final修饰的方法、下标、属性，禁止被重写
 - 被final修饰的类，禁止被继承

### 递归枚举
递归枚举是一种枚举类型，它有一个或多个枚举成员使用该枚举类型的实例作为关联值。使用递归枚举时，编译器会插入一个间接层。你可以在枚举成员前加上 indirect 来表示该成员可递归

```swift
enum ArithmeticExpression {
    case number(Int)
    indirect case addition(ArithmeticExpression, ArithmeticExpression)
    indirect case multiplication(ArithmeticExpression, ArithmeticExpression)
}

// 你也可以在枚举类型开头加上 indirect 关键字来表明它的所有成员都是可递归的：
// 定义的枚举类型可以存储三种算术表达式：纯数字、两个表达式相加、两个表达式相乘。枚举成员 addition 和 multiplication 的关联值也是算术表达式——这些关联值使得嵌套表达式成为可能。
indirect enum ArithmeticExpression {
    case number(Int)
    case addition(ArithmeticExpression, ArithmeticExpression)
    case multiplication(ArithmeticExpression, ArithmeticExpression)
}

// 使用 ArithmeticExpression 这个递归枚举创建表达式 (5 + 4) * 2
let five = ArithmeticExpression.number(5)
let four = ArithmeticExpression.number(4)
let sum = ArithmeticExpression.addition(five, four)
let product = ArithmeticExpression.multiplication(sum, ArithmeticExpression.number(2))

// 算术表达式求值的函数
// 该函数如果遇到纯数字，就直接返回该数字的值。如果遇到的是加法或乘法运算，则分别计算左边表达式和右边表达式的值，然后相加或相乘。
func evaluate(_ expression: ArithmeticExpression) -> Int {
    switch expression {
    case let .number(value):
        return value
    case let .addition(left, right):
        return evaluate(left) + evaluate(right)
    case let .multiplication(left, right):
        return evaluate(left) * evaluate(right)
    }
}
print(evaluate(product)) // 打印“18”
```

### MemoryLayout
```swift 
/// 用法一
MemoryLayout<password>.stride // 分配的占用内存
MemoryLayout<password>.size // 实际的内存大小
MemoryLayout<password>.alignment // 对齐参数

/// 用法二
let pwd = password.number(1, 1, 1, 1)        
MemoryLayout.stride(ofValue: pwd)
MemoryLayout.size(ofValue: pwd)
MemoryLayout.alignment(ofValue: pwd)
```

### 关联值跟原始值的区别
- 枚举类型变量会占用1个字节 不论枚举变量类型是Int还是String 都不会根据定义类型来计算字节大小
- 关联值跟原始值的区别
 1、关联类型的枚举，其实是会存储对应的关联类型的值的，关联类型占用多少个字节就影响枚举的内存
 关联值会占用枚举变量的内存，会根据外界传值类型计算大小
 2、原始值不允许你自定义，也不会根据枚举类型计算内存大小
 原始值不会占用枚举变量的内存，只会占用1个字节，用来标记枚举类型

### 值类型与引用类型
**值类型**
值类型赋值给var、let或者给函数传参，是直接将所有内容拷贝一份
类似于对文件进行copy、paste操作，产生了全新的文件副本。属于深拷贝（deep copy）

**引用类型**
引用赋值给var、let或者给函数传参，是将内存地址拷贝一份
类似于制作一个文件的替身（快捷方式、链接），指向的是同一个文件。属于浅拷贝（shallow copy）

### map 与 flatmap 的区别
flatmap 有两个实现函数实现
```swift 
public func flatMap<ElementOfResult>(_ transform: (Element) throws -> ElementOfResult?) rethrows -> [ElementOfResult]
```
这个方法, 中间的函数返回值为一个可选值, 而 flatmap 会丢掉那些返回值为 nil 的值
```swift
["1", "@", "2", "3", "a"].flatMap{Int($0)}
// [1, 2, 3]
["1", "@", "2", "3", "a"].map{Int($0)}
//[Optional(1), nil, Optional(2), Optional(3), nil]
```
另一个实现
```swift
public func flatMap<SegmentOfResult>(_ transform: (Element) throws -> SegmentOfResult) rethrows -> [SegmentOfResult.Iterator.Element] where SegmentOfResult : Sequence
```
中间的函数, 返回值为一个数组, 而这个 flapmap 返回的对象则是一个与自己元素类型相同的数组
```swift
func someFunc(_ array:[Int]) -> [Int] {
    return array
}
[[1], [2, 3], [4, 5, 6]].map(someFunc)
// [[1], [2, 3], [4, 5, 6]]
[[1], [2, 3], [4, 5, 6]].flatMap(someFunc)
// [1, 2, 3, 4, 5, 6]
```
其实这个实现, 相当于是在使用 map 之后, 再将各个数组拼起来一样的
```swift
[[1], [2, 3], [4, 5, 6]].map(someFunc).reduce([Int]()) {$0 + $1}
// [1, 2, 3, 4, 5, 6]
```

### 闭包
**闭包**
是我们最常用的闭包的一种，是一个利用轻量级语法所写的可以捕获其上下文中变量或常量值的匿名闭包

1. 闭包捕获值的本质是在堆区开辟内存然后存储其在上下文中捕获到的值
2. 修改值也是修改的堆空间的值
3. 闭包是一个引用类型
4. 闭包的底层结构是一个结构体
    - 首先存储闭包的地址
    - 加上捕获值的地址
5. 在捕获的值中，会对定义的变量和函数中的参数分开存储
6. 存储的时候内部会有一个HeapObject结构，用于管理内存，引用计数
7. 函数是特殊的闭包，只不过函数不捕获值，所以在闭包结构体中只存储函数地址，不存储指向捕获值的指针

使用闭包有很多好处：
1. 可以利用上下文自动推断参数和返回值的类型
2. 隐士返回单表达式闭包，也就是但表达式的时候可以省略return关键字
3. 参数名称可以简写或者不写，使用$0表示第一个参数
4. 尾随闭包表达式

**自动闭包**
 - 自动闭包是一种自动创建的闭包，用于包装传递给函数作为参数的表达式。
 - 这种闭包不接受任何参数，当它被调用的时候，会返回被包装在其中的表达式的值。
 - 这种便利语法让你能够省略闭包的花括号，用一个普通的表达式来代替显式的闭包。
 - 自动闭包让你能够延迟求值，因为直到你调用这个闭包，代码段才会被执行。
 - 延迟求值对于那些有副作用（Side Effect）和高计算成本的代码来说是很有益处的，因为它使得你能控制代码的执行时机。

### 逃逸闭包&非逃逸闭包&尾随闭包
**非逃逸闭包的特性** 一个接受闭包作为参数的函数，闭包是在这个函数结束前内被调用，即可以理解为闭包是在函数作用域结束前被调用.
 - 执行时机：在函数体内执行
 - 声明周期：在函数执行完后，闭包也就消失了
 - 不会产生循环引用，因为闭包的作用域在函数作用域内，在函数执行完成后，就会释放闭包捕获的所有对象
 - 针对非逃逸闭包，编译器会做优化：省略内存管理调用

**逃逸闭包** 当闭包作为一个实际参数传递给一个函数的时候，并且是在函数返回之后调用，我们就说这个闭包逃逸了。当我们声明一个接受闭包作为形式参数的函数时，你可以在形式参数前些 **@escapeing** 来明确闭包是允许逃逸的。
另外使用 **@escapeing** 修饰的闭包在其闭包体内需要显示的使用self，对于显示的使用self主要是希望开发者注意循环引用。

**尾随闭包** 函数的最后一个参数是闭包，就叫做尾随闭包。好处：可以增强可读性。

逃逸闭包的使用时机分为以下两种情况：
保存在一个函数外部定义的变量中
延迟调用

总结
关于逃逸闭包和非逃逸闭包基本就说完了，下面稍作总结：
1. 首先两者都是作为函数参数
2. 非逃逸闭包在函数中调用，并且是函数结束前就调用完成
3. 非逃逸闭包不会产生循环引用，因为作用域在函数内，函数执行完毕会释放函数内的所有对象
4. 针对非逃逸闭包编译器会优化内存管理的调用
5. 根据官方文档的说明，非逃逸闭包会保存在栈上（未验证出来）
6. 逃逸闭包在函数返回后才会调用，也就是闭包逃离出了函数的作用域
7. 逃逸闭包中可能会产生循环引用
8. 逃逸闭包中需要显示的引用self，主要作用是提醒开发者注意循环引用

### inout
1. 如果实参有物理内存地址，且没有设置属性观察器,直接将实参的内存地址传入函数（实参进行引用传递）
2. 如果实参是计算属性或者设置了属性观察器,采取了Copy In Copy Out的做法
  - 调用该函数时，先复制实参的值，产生副本【get】
  - 将副本的内存地址传入函数（副本进行引用传递），在函数内部可以修改副本的值
  - 函数返回后，再将副本的值覆盖实参的值【set】
3. 总结：inout的本质就是引用传递（地址传递）

### required
用required修饰指定初始化器，表明其所有子类都必须实现该初始化器（通过继承或者重写实现）

如果子类重写了required初始化器，也必须加上required，不用加override

### Any、AnyObject
Swift提供了2种特殊的类型：Any、AnyObject
 - Any：可以代表任意类型（枚举、结构体、类，也包括函数类型）
 - AnyObject：可以代表任意类类型（在协议后面写上: AnyObject代表只有类能遵守这个协议）

在协议后面写上: class也代表只有类能遵守这个协议

### 访问控制(Access Control)
在访问权限控制这块，Swift提供了5个不同的访问级别（以下是从高到低排列， 实体指被访问级别修饰的内容）
 - open：允许在定义实体的模块、其他模块中访问，允许其他模块进行继承、重写（open只能用在类、类成员上）
 - public：允许在定义实体的模块、其他模块中访问，不允许其他模块进行继承、重写
 - internal：只允许在定义实体的模块中访问，不允许在其他模块中访问
 - fileprivate：只允许在定义实体的源文件中访问
 - private：只允许在定义实体的封闭声明中访问

绝大部分实体默认都是internal级别

### 内存管理
Swift的ARC中有3种引用
 - 强引用（strong reference）：默认情况下，引用都是强引用
 - 弱引用（weak reference）：通过weak定义弱引用
必须是可选类型的var，因为实例销毁后，ARC会自动将弱引用设置为nil
ARC自动给弱引用设置nil时，不会触发属性观察器
 - 无主引用（unowned reference）：通过unowned定义无主引用
不会产生强引用，实例销毁后仍然存储着实例的内存地址（类似于OC中unsafe_unretained）
试图在实例销毁后访问无主引用，会产生运行时错误（野指针）
• Fatal error: Attempted to read an unowned reference but object 0x0 was already deallocated

### 循环引用
weak、unowned 都能解决循环引用的问题，unowned 要比weak 少一些性能消耗
 - 在生命周期中可能会变为nil 的使用weak
 - 初始化赋值后再也不会变为nil 的使用unowned

如何解决引用循环
 1. 转换为值类型, 只有类会存在引用循环, 所以如果能不用类, 是可以解引用循环的
 2. delegate 使用 weak 属性.
 3. 闭包中, 对有可能发生循环引用的对象, 使用 **weak** 或者 **unowned** 修饰

作者：温水煮青蛙a
链接：https://www.jianshu.com/p/665cb3cb0003
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### 闭包的循环引用
 如果想在定义闭包属性的同时引用self，这个闭包必须是lazy的（因为在实例初始化完毕之后才能引用self）

如果lazy属性是闭包调用的结果，那么不用考虑循环引用的问题（因为闭包调用后，闭包的生命周期就结束了）

### 指针
Swift中也有专门的指针类型，这些都被定性为“Unsafe”（不安全的），常见的有以下4种类型
 - UnsafePointer<Pointee> 类似于const Pointee *
 - UnsafeMutablePointer<Pointee> 类似于Pointee *
 - UnsafeRawPointer 类似于const void *
 - UnsafeMutableRawPointer 类似于void *

### dynamic(daɪˈnæmɪk)
 被 **@objc dynamic** 修饰的内容会具有动态性，比如调用方法会走runtime那一套流程

 由于 swift 是一个静态语言, 所以没有 Objective-C 中的消息发送这些动态机制, dynamic 的作用就是让 swift 代码也能有 Objective-C 中的动态机制, 常用的地方就是 KVO 了, 如果要监控一个属性, 则必须要标记为 dynamic。

### dynamic framework 和 static framework 的区别是什么
 静态库和动态库, 静态库是每一个程序单独打包一份, 而动态库则是多个程序之间共享

### @objc与@objcMembers的区别
 @objc 用途是为了在 Objective-C 和 Swift 混编的时候, 能够正常调用 Swift 代码. 可以用于修饰类, 协议, 方法, 属性.
常用的地方是在定义 delegate 协议中, 会将协议中的部分方法声明为可选方法, 需要用到@objc
```swift
@objc protocol OptionalProtocol {
    @objc optional func optionalFunc()
    func normalFunc()
}
class OptionProtocolClass: OptionalProtocol {
    func normalFunc() {
    }
}
let someOptionalDelegate: OptionalProtocol = OptionProtocolClass()
someOptionalDelegate.optionalFunc?()
```
在Swift中，继承自NSObject的类如果有比较多的属性或方法都需要加上@objc的话，会多比较多的代码。那么可以利用@objcMembers减少代码。被@objcMembers修饰的类，会默认为类、子类、类扩展和子类扩展的所有属性和方法都加上@objc。当然如果想让某一个扩展关闭@objc，则可以用@nonobjc进行修饰。
```swift
@objcMembers
class People: NSObject {
    var name = ""
    var age = 10
    func playGame() {
        print("玩游戏")
    }
}
```

### 什么是高阶函数
一个函数如果可以以某一个函数作为参数, 或者是返回值, 那么这个函数就称之为高阶函数, 如 map, reduce, filter



