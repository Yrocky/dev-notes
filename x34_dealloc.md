### dealloc

dealloc方法是对象的一个析构操作，主要会在内部对一些持有的对象进行释放，腾出来内存。在MRC的情况下，需要在dealloc中对持有的实例变量进行释放，并且调用父类的dealloc方法，但是到了dealloc中，由于是自动管理内存，不需要操作实例变量的释放以及调用父类的方法：

```objective-c
// MRC
- (void)dealloc {
    self.array = nil;
    self.string = nil;
    // ... //
    // 非Objc对象内存的释放，如CFRelease(...)
    // ... //
    [super dealloc];
}

// ARC
- (void)dealloc{
    // ... //
    // 非Objc对象内存的释放，如CFRelease(...)
    // ... //
}
```

那么再自动管理内存的机制下，对象的实例变量是何时释放的以及何时上层的析构何时调用呢？



### .cxx_destruct

`.cxx_destruct`原本为C++对象的析构，ARC借用这个方法插入代码实现自动释放内存。



---

http://blog.sunnyxx.com/2014/04/02/objc_dig_arc_dealloc/