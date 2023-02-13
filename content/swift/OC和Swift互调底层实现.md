
Swift 纯Swift类的函数的调用已经不是OC的运行时发送消息，和C类似，在编译阶段就确定了调用哪一个函数，所以纯Swift的类我们是没办法通过运行时去获取到它的属性和方法的。

Swift 对于继承自OC的类，为了兼容OC，凡是继承与OC的都是保留了它的特性的，所以可以使用Runtime获取到它的属性和方法等等其他我们在OC中获得的东西。

### OC调用Swift
这里创建一个名为Test的OC项目
在OC项目中创建一个swift文件Person.swift，会提示是否创建桥接文件。文件名格式是： {targetName}-Swift.h 

这个桥接文件是用于swift调用OC的，可以创建，若不创建则后续手动创建也是可以的

代码调用
1. swfit类要暴露给OC调用，这个类必须要继承自NSObject。
因为OC调用方法使用的是runtime的消息机制，类需要有isa指针，但是swift类是没有的，所以要继承自NSObject基类，才会有isa指针。
2. swift类需要暴露给OC调用的成员和方法需要用@objc来修饰一下

### Swift调用OC
1. 创建桥接文件。当我们创建一个oc文件时，编译器会自动为我们创建一个1个桥接头文件，文件名格式默认为：{targetName}-Bridging-Header.h
2. 在 {targetName}-Bridging-Header.h 文件中 #import OC需要暴露给Swift的内容, 如
```Objective-C
#import "MJPerson.h"
```

### OC调用Swift方法走的是runtime机制吗
是！因为Swift类继承自NSObject才能让OC使用，所以类也有isa指针，runtime消息机制就是通过isa指针来寻找方法。
OC调用swift的JJSay方法，断点查看汇编可以看到红框部分的注释，有调用objc_msgSend，这个就是OC运行时的消息发送机制。

### Swift调用OC方法走的是runtime机制吗？
是！因为OC方法在.m文件里编译，所以肯定是runtime机制。
Swift调用OC的eat方法，断点查看汇编可以看到红框部分的注释，有调用objc_msgSend，这个就是OC运行时的消息发送机制。

### Swift类继承了NSObject,纯swift调用
swift类继承了NSObject，并且也用 **@objc** 或者 **@objcMembers** 暴露了方法，在swift文件里直接使用这个类的方法是走的runtime机制吗？
不会！
查看汇编可以看到，调用方法这里并没有objc_msgSend。
 - @objc修饰需要暴露给OC的成员
 - @objcMembers修饰类，代表默认所有成员都会暴露给OC（包括扩展中定义的成员）

### swift文件调用swift方法希望它使用runtime怎么做呢
在方法前加上dynamic关键字修饰，如：
```swift
dynamic func say() { print("hello") }
```
查看汇编，可以看到调用say方法时的注释有objc_msgSend。


### 选择器（Selector）
Swift中依然可以使用选择器，使用#selector(name)定义一个选择器
必须是被@objcMembers或@objc修饰的方法才可以定义选择器



