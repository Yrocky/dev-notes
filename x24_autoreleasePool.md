> autoreleasepool的作用：对自动释放对象进行释放，减少内存峰值
>
> autoreleasepool的实现原理：**AutoreleasePoolPage**是核心
>
> autoreleasepool和runloop、thread之间的关系

---

### autoreleasepool

在MRC管理内存的模式下，通常可以为对象调用release将对象释放，也可以使用autorelease，将对象加入到autoreleasePool中，自动释放池“稍后某个时刻”会释放里面所有的对象。那么加入到autoreleasepool的对象什么时候会释放呢？

在ARC模式下，内存的回收是编译器控制的，这个和在MRC下调用autorelease是一样的，不过这个自动释放池不确定是在哪个地方，假如整个app只有一个自动释放池（位于main函数中的那一个），那么所有的内存都会在程序结束运行的时候进行释放，这显然是不合理的（当然，这个假设本来就不合理，不可能只有一个自动释放池）。

自动释放池是需要等当前线程执行下一次事件循环才会清空，这就意味着一些临时对象即使不再使用，也要在内存中存在着。这个时候就需要手动的添加一个autoreleasepool来将临时对象释放，加入到池子中的对象会在池子的末尾进行回收对象内存。



### AutoreleasePoolPage

autoreleasepool本身像是一个“栈”，所有新加入的对象都会加入到栈顶，直到清空池子的时候，将里面的对象移除出栈。

`AutoreleasePoolPage`是使用C++实现的类，多个PoolPage以`双向链表`的形式组合，这样就形成了一个pool。每一个PoolPage都是有大小限制的，大小为4096字节（虚拟内存一页的大小）。由于每一个线程都有一个autoreleasepool，所以，PoolPage内部会有指代当前线程的字段，在当前PoolPage的内存空间占满之后，会重新创建一个PoolPage对象，以链表的形式链接到后面，新加入的自动释放对象加入到链表最后的PoolPage的栈顶。

创建一个autoreleasepool是通过PoolPage的`push()`方法，这个push()在runtime中被封装成：

```objective-c
void *objc_autoreleasePoolPush(void){
    if (UseGC) return nil;
    return AutoreleasePoolPage::push();
}
```

然后block中的自动释放对象都会被加入到pool中，直到block结束（pool的结尾花括号），调用PoolPage的`pop()`方法，在runtime中被封装为：

```objective-c
void objc_autoreleasePoolPop(void *ctxt){
    if (UseGC) return;
    // fixme rdar://9167170
    if (!ctxt) return;
    AutoreleasePoolPage::pop(ctxt);
}
```

在调用`objc_autoreleasePoolPush`函数的时候会返回一个哨兵对象，用来作为`objc_autoreleasePoolPop`函数的入参，在pool清空的时候，将哨兵对象后面所有添加的对象全部realse。

一个pool可以简单的理解为：

```objective-c
/* @autoreleasepool */ {
    void *autoreleasepoolObj = objc_autoreleasePoolPush();
    // 用户代码，所有接收到 autorelease 消息的对象会被添加到这个 autoreleasepoolObj 中
    objc_autoreleasePoolPop(autoreleasepoolObj);
}
```



### main()

在main文件下的main()函数是整个软件的入口，这里使用了autoreleasepool：

```objective-c
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```



### runloop

在iOS应用启动后会注册两个Observer管理和维护AutoreleasePool。在应用程序刚刚启动时（main函数）打印currentRunLoop（主线程的runloop）可以看到系统默认注册了很多个Observer，其中有两个Observer的callout都是**_wrapRunLoopWithAutoreleasePoolHandler**，这两个是和自动释放池相关的两个监听：

```objective-c
<CFRunLoopObserver 0x6080001246a0 [0x101f81df0]>{valid = Yes, activities = 0x1, repeats = Yes, order = -2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x1020e07ce), context = <CFArray 0x60800004cae0 [0x101f81df0]>{type = mutable-small, count = 0, values = ()}}

<CFRunLoopObserver 0x608000124420 [0x101f81df0]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x1020e07ce), context = <CFArray 0x60800004cae0 [0x101f81df0]>{type = mutable-small, count = 0, values = ()}}
```

第一个Observer会监听**RunLoop的进入**，它会回调`objc_autoreleasePoolPush()`向当前的AutoreleasePoolPage增加一个哨兵对象标志==创建自动释放池==。这个Observer的order是-2147483647优先级最高，确保发生在所有回调操作之前。

第二个Observer会监听**RunLoop的进入休眠**和**即将退出RunLoop**两种状态，这个Observer的order是2147483647，优先级最低，确保发生在所有回调操作之后：

* 在即将进入休眠时会调用`objc_autoreleasePoolPop() `和 `objc_autoreleasePoolPush() `根据情况从最新加入的对象一直往前清理直到遇到哨兵对象

* 而在即将退出RunLoop时会调用`objc_autoreleasePoolPop()` ==释放自动自动释放池内对象==。

主线程的其他操作通常均在这个AutoreleasePool之内（main函数中的那个），以尽可能减少内存维护操作，不过在循环产生大量变量的时候，可以自己创建AutoreleasePool，否则一般不需要自己创建。



### 线程

每一个线程都会维护自己的 autoreleasepool 堆栈。也就是说 autoreleasepool 是与线程紧密相关的，每一个 autoreleasepool 只对应一个线程。

> Each thread (including the main thread) maintains its own stack of NSAutoreleasePool objects.

在子线程中默认是不会开启runloop的，需要手动调用run，上面说了，runloop会有两个autoreleasepool的observer，那么如果在子线程中产生了自动释放的对象，怎么处理？

在子线程（没有run起来runloop的情况下）中如果有手动创建pool，产生的自动释放对象会放入pool中；如果没有创建pool，就会调用 autoreleaseNoPage 方法。在这个方法中，会自动创建一个 hotpage（hotPage 可以理解为当前正在使用的 AutoreleasePoolPage ），并调用 page->add(obj)将对象添加到 AutoreleasePoolPage 的栈中，也就是说即使不进行手动的内存管理，也不会内存泄漏！



### 循环

在支持block枚举回调的集合类中，内部都会自动添加一个autoreleasepool，这样会合理的处理内存峰值。

不过在for以及for-in循环中，是没有释放池的，因此，如果会产生大量的临时数据，可以使用autoreleasepool来控制内存峰值。

假如在循环的时候有两种方式：

```objective-c
int i = 0;
NSMutableArray * arr_1 = [@[] mutableCopy];
// opt-1
@autoreleasepool {
    while (i < 100) {
        MMString * obj = [[MMString alloc] init];
        [arr_1 addObject:obj];
        i ++;
    }
}

i = 0;
NSMutableArray * arr_2 = [@[] mutableCopy];
// opt-2
while (i < 100) {
    @autoreleasepool {
        MMString * obj = [[MMString alloc] init];
        [arr_2 addObject:obj];
        i ++;
    }
}
```

在第一种情况下，pool会在循环结束之后才会去清空内部的对象，对于内部循环不会产生大量的临时内存的情况下，这种方法还是可以接受的；而第二种情况下，每一次循环都会将pool内部的对象清空，这对于每次循环都会产生的数据结构会导致内存膨胀的情况很有用。总的来说，建议直接使用第二种方法来避免内存峰值过高。



### 自动加入pool

在main函数中，有一个自动释放池，并不代表里面所有的对象都交给pool来处理。

如果是使用`alloc/new/copy/mutableCopy`初始化的对象，由于会引起引用计数加1，所以不需要使用pool来管理，但是使用其他初始化方法（[NSString string]、[NSArray array]等）就不会引起引用计数加一，会将这些对象放入pool中。



----

https://www.jianshu.com/p/f87f40592023

http://blog.sunnyxx.com/2014/10/15/behind-autorelease/

http://www.cocoachina.com/ios/20150610/12093.html

https://opensource.apple.com/tarballs/objc4/

https://www.jianshu.com/p/32265cbb2a26