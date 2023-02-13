### RunLoop内部实现逻辑
RunLoop 顾名思义是运行循环，在程序运行过程中循环做一些事情。

RunLoop的基本作用
- 保持程序的持续运行
- 处理App中的各种事件（比如触摸事件、定时器事件等）
- 节省CPU资源，提高程序性能：该做事时做事，该休息时休息

RunLoop与线程
- 每条线程都有唯一的一个与之对应的RunLoop对象
- RunLoop保存在一个全局的Dictionary里，线程作为key，RunLoop作为value
- 线程刚创建时并没有RunLoop对象，RunLoop会在第一次获取它时创建
- RunLoop会在线程结束时销毁
- 主线程的RunLoop已经自动获取（创建），子线程默认没有开启RunLoop

RunLoop的运行逻辑 
1. 通知Observers：进入Loop
2. 通知Observers：即将处理Timers
3. 通知Observers：即将处理Sources
4. 处理Blocks
5. 处理Source0（可能会再次处理Blocks）
6. 如果存在Source1，就跳转到第8步
7. 通知Observers：开始休眠（等待消息唤醒）
8. 通知Observers：结束休眠（被某个消息唤醒）
- 处理Timer
- 处理GCD Async To Main Queue
- 处理Source1
9. 处理Blocks
10. 根据前面的执行结果，决定如何操作
- 回到第02步
- 退出Loop
11. 通知Observers：退出Loop

RunLoop休眠的实现原理   
有个用户态和内核态，mach_msg()，等待消息
- 没有消息就让线程休眠
- 有消息就唤醒线程

CFRunLoopModeRef
- CFRunLoopModeRef代表RunLoop的运行模式
- 一个RunLoop包含若干个Mode，每个Mode又包含若干个Source0/Source1/Timer/Observer
- RunLoop启动时只能选择其中一个Mode，作为currentMode。如果需要切换Mode，只能退出当前Loop，再重新选择一个Mode进入
- 不同组的Source0/Source1/Timer/Observer能分隔开来，互不影响
- 如果Mode里没有任何Source0/Source1/Timer/Observer，RunLoop会立马退出

model主要是用来指定事件在运行循环中的优先级的，分为：
- NSDefaultRunLoopMode（kCFRunLoopDefaultMode）：默认，空闲状态
- UITrackingRunLoopMode：界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响
- UIInitializationRunLoopMode：启动时
- NSRunLoopCommonModes（kCFRunLoopCommonModes）：Mode集合

苹果公开提供的 Mode 有两个(常见的2种Mode)：
- NSDefaultRunLoopMode（kCFRunLoopDefaultMode）App的默认Mode，通常主线程是在这个Mode下运行
- NSRunLoopCommonModes（kCFRunLoopCommonModes）不是一个真正的Mode，比如定时器，在滑动的时候还能继续倒计时，解决运行模式的缺陷


### RunLoop在实际开中的应用
- 控制线程生命周期（线程保活）
- 解决NSTimer在滑动时停止工作的问题
- 监控应用卡顿
- 性能优化