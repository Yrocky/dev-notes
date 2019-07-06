* [x01_变量限定符](x01_变量限定符.md)

  - [x] 理解内存管理中的三种变量限定符的含义以及使用场景
* [x02_关于string的一些问题](x02_关于string的一些问题.md)

  - [x] NSString不可变的特性，以及使用这种类簇模式下的类需要的注意事项

* [x03_builder模式和消息转发的具体使用案例](x03_builder模式和消息转发的具体使用案例.md)

  - [x] 使用Builder模式来将对象的创建和使用合理的分离，并结合消息转发机制优雅的完善Builder模式

* [x04_self=[super init]](x04_self=[superinit].md)

  - [x] 是否在初始化的时候一定要调用[super init]，调用它的含义是什么

* [x05_instancetype和id的区别](x05_instancetype和id的区别.md)

  - [x] 两者之间的区别，Apple给的编程建议

* [x06_addBlockToArray](x06_addBlockToArray.md)

  - [ ] 关于block的一些面试题，引申出来block关于变量捕获，在堆区和在栈区的区别

* [x07_load&Initialize](x07_load&Initialize.md)

  - [x] +load方法和+initalize方法的本质意义，以及两者在调用方面的差异

* [x08_selector和IMP的关系](x08_selector和IMP的关系.md)

  - [x] SEL和IMP两者之间的区别，如何通过SEL来获取对应的IMP，根据两者之间的关系可以做一些swizzling操作

* [x09_layer和view](x09_layer和view.md)

  - [x] layer和view之间的关系，这种平行的关系是不是有必要，

* [x10_alloc&init](x10_alloc&init.md)

  - [x] 一个类alloc之后究竟做了哪些操作，init以前是否需要进行alloc

* [x11_内存](x11_内存.md)

  - [ ] 内存的几种划分方式，堆区和栈区上的内存差异

* [x12_堆和栈](x12_堆和栈.md)

  - [ ] 如何区分堆区和栈区

* [x13_Run](x13_Run.md)

  - [x] 当点击了Xcode编译器的run按钮之后到app运行起来，这之间经历了什么

* [x14_代理类和消息转发](x14_代理类和消息转发.md)

  - [x] 对于使用NSProxy和NSObject来实现代理类的区别

* [x15_弱代理和消息转发](x15_弱代理和消息转发.md)

  - [x] 使用弱代理解决循环引用的问题，以及YYWeakProxy的实现原理

* [x16_AOP](x16_AOP.md)

  - [x] 面向切片编程的含义，以及它的作用范围

* [x17_多继承](x17_多继承.md)

  - [x] 在Cocoa开发中如何使用NSProxy来模拟完成多继承这一面向对象的重要特性

* [x18_KVO](x18_KVO.md)

  - [x] 对KVO模式的实现原理进行探究，第三方更加优雅的根据KVO的原理结合runtime实现的KVOController

* [x019_计时器](x19_计时器.md)

  - [x] 实现计时器操作的几种方式，使用计时器过程中会经常遇到的循环引用问题，以及如何解决循环引用问题

* [x20_perfromSelector](x20_perfromSelector.md)

  - [x] 几种`perfromSelector`方法的实质，以及他们之间的区别，在线程层面上的performSelector需要注意什么

* [x21_didReceiveMemoryWarning](x21_didReceiveMemoryWarning.md)

  - [x] 当软件内存过高的时候系统有何措施，软件内部如何应对内存不足的情况

* [x22_优化启动时间](x22_优化启动时间.md)

  - [x] 软件从启动到进入主界面之间经历了什么，如何减少启动时间

* [x23_Autorelease](x23_Autorelease.md)

  - [ ] 在MRC下，除了手动管理内存，还有一个autorelease，那么使用了他的对象何时释放

* [x24_autoreleasePool](x24_autoreleasePool.md)

  - [x] autoreleasePool的本质是什么，他有什么作用，什么时候使用它

* [x25_main()](x25_mian().md)

  - [x] mian函数之前会经历那些事情

* [x26_runloop](x26_runloop.md)

  - [ ] runloop的作用、设计模式、使用场景

* [x27_响应链](x27_响应链.md)

  - [x] 事件是如何传递的，什么是响应链，响应链的作用

* [x28_NSCopying](x28_NSCopying.md)

  - [x] 理解NSCopying协议的作用，以及浅拷贝和深拷贝的区别

* [x29_NSFileManager](x29_NSFileManager.md)

  - [x] 系统提供了几种文件夹，如何获取到沙盒的位置，多线程中使用NSFileManager的注意事项

* [x30_宏](x30_宏.md)

  - [x] 宏的本质含义是什么，RAC中的几种宏的具体实现方式是怎样的

* [x31_优化](x31_优化.md)

  - [ ] 项目优化的方向，你是如何对项目进行优化的

* [x32_布局](x32_布局.md)

  - [ ] iOS到现在为止使用过的布局技术，对控件设置约束的时机
