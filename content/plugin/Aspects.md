iOS AOP框架Aspects实现原理

### 整体原理

Aspects主要是利用了forwardInvocation进行转发，Aspects其实利用和kvo类似的原理，通过动态创建子类的方式，把对应的对象isa指针指向创建的子类，然后把子类的forwardInvocation的IMP替成__ASPECTS_ARE_BEING_CALLED__，假设要hook的方法名XX,在子类中添加一个Aspects_XX的方法，然后将Aspects_XX的IMP指向原来的XX方法的IMP，这样方便后面调用原始的方法，再把要hook的方法XX的IMP指向_objc_msgForward，这样就进入了消息转发流程，而forwardInvocation的IMP被替换成了__ASPECTS_ARE_BEING_CALLED__，这样就会进入__ASPECTS_ARE_BEING_CALLED__进行拦截处理，这样整个流程大概结束。



### 参考链接
[iOS AOP框架Aspects实现原理](https://www.jianshu.com/p/0d43db446c5b)
[IOS架构：组件化方案与AOP面向切面编程](https://www.jianshu.com/p/872c798034ea)
