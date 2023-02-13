

### Swift和OC的区别？
1. **Swift**是类型安全的静态语言，有类型推断，OC是动态语言。
 - Swift如果代码中使用一个字符串String，那么你不能错误地传一个整形Int给他。因为Swift是类型安全的，它会在代码编译的时候做类型检查，并且把所有不匹配的类型作为一个错误标记出来。
 - Object-C 则不然，你声明一个NSString 变量，仍然可以传一个 NSNumber 给它，尽管编译器会抱怨，但是你仍然可以作为 NSNumber 去使用它。
2. swift面向协议编程，OC面向对象编程
3. swift注重值类型，OC注重引用类型。
 - Swift 可以面向协议编程、函数式编程、面向对象编程。
 - Object-C 以面向对象编程为主，当然你可以引入类似ReactiveCocoa的类库来进行函数式编程。
4. swift支持泛型，OC只支持轻量泛型
5. swift支持静态派发（效率高）、动态派发（函数表派发、消息派发）方式，OC支持动态派发（消息派发）方式。
6. swift支持函数式编程
7. swift的协议不仅可以被类实现，也可以被struct和enum实现
8. swift有元组类型、支持运算符重载
9. swift支持命名空间
10. swift支持默认参数
11. swift比oc代码更加简洁

### swift 和 oc 的编译区别
![](/assets/swift/swift_oc_llvm.png)
 - OC 通过 clang 编译器，编译成IR，然后再⽣成可执⾏⽂件.o(这⾥也就是我们的机器码)
 - Swift 则是通过 Swift编译器 编译成IR，然后在⽣成可执⾏⽂件。
 ![](/assets/swift/LLVM_compile.png)

### 常用的Swift第三方库
网络请求： https://github.com/Alamofire/Alamofire
图片下载： https://github.com/onevcat/Kingfisher
JSON访问： https://github.com/SwiftyJSON/SwiftyJSON
JSON-Model转换：https://github.com/kakaopensource/KakaJSON


