
## 索引
   * [APP启动流程](#app启动流程)
   * [APP的启动优化](#app的启动优化)
   * [APP安装包瘦身](#app安装包瘦身)

### APP启动流程
APP的启动可以分为2种
- 冷启动（Cold Launch）：从零开始启动APP
- 热启动（Warm Launch）：APP已经在内存中，在后台存活着，再次点击图标启动APP

APP启动时间的优化，主要是针对冷启动进行优化

通过添加环境变量可以打印出APP的启动时间分析（Edit scheme -> Run -> Arguments）  
DYLD_PRINT_STATISTICS设置为1   
如果需要更详细的信息，那就将DYLD_PRINT_STATISTICS_DETAILS设置为1  

APP的冷启动可以概括为3大阶段
1. dyld：dyld（dynamic link editor），Apple的动态链接器，可以用来装载Mach-O文件（可执行文件、动态库等）  
 * 装载APP的可执行文件，同时会递归加载所有依赖的动态库
 * 当dyld把可执行文件、动态库都装载完毕后，会通知Runtime进行下一步的处理

2. runtime。由runtime负责加载成objc定义的结构，runtime所做的事情有
 - 调用map_images进行可执行文件内容的解析和处理
 - 在load_images中调用call_load_methods，调用所有Class和Category的+load方法
 - 进行各种objc结构的初始化（注册Objc类、初始化类对象等等）
 - 调用C++静态初始化器和__attribute__((constructor))修饰的函数。

    到此为止，可执行文件和动态库中所有的符号(Class，Protocol，Selector，IMP，…)都已经按格式成功加载到内存中，被runtime 所管理

3. main  
所有初始化工作结束后，dyld就会调用main函数
接下来就是UIApplicationMain函数，AppDelegate的application:didFinishLaunchingWithOptions:方法

### APP的启动优化
按照不同的阶段进行优化
1. dyld
 + 减少动态库、合并一些动态库（定期清理不必要的动态库）
 + 减少Objc类、分类的数量、减少Selector数量（定期清理不必要的类、分类）
 * 减少C++虚函数数量
 * Swift尽量使用struct

2. runtime
 - 用+initialize方法和dispatch_once取代所有的__attribute__((constructor))、C++静态构造器、ObjC的+load

3. main
 - 在不影响用户体验的前提下，尽可能将一些操作延迟，不要全部都放在finishLaunching方法中
 - 按需加载

### APP安装包瘦身
安装包（IPA）主要由可执行文件、资源组成

1. 资源（图片、音频、视频等）
 - 采取无损压缩
 - 去除没有用到的资源： https://github.com/tinymind/LSUnusedResources

2. 可执行文件瘦身
 - 编译器优化，Strip Linked Product、Make Strings Read-Only、Symbols Hidden by Default设置为YES
 - 去掉异常支持，Enable C++ Exceptions、Enable Objective-C Exceptions设置为NO， Other C Flags添加-fno-exceptions
 - 利用AppCode（https://www.jetbrains.com/objc/）检测未使用的代码：菜单栏 -> Code -> Inspect Code
 - 编写LLVM插件检测出重复代码、未被调用的代码