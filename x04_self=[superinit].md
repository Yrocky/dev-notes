> 类簇的概念
>
> self的具体含义

---

### [super init]的实质

`[super init]`在运行时会被转化为：

```objective-c
objc_msgSendSuper(self, @selector(init));
```

可以看到，这时候已经有一个self存在了，这个self也是隐藏的参数，它是通过`[SomeClass alloc]`方法得到的内存地址之后存在的，目的是指向这块地址。

`[super init]`会做三件事：

* 返回自己的接收者（也就是需要的self）
* 返回不同的对象（一般在类簇中比较常见，还不可以避免）
* 返回一个初始化错误的nil

上面的第二种，如果返回的是不同的对象，那么对这个`不同的对象`设置属性有可能会引起内存泄露以及crash。一个标准的初始化方法：

```objective-c

- (instancetype)init{

    self = [super init];// 从super init中获取self指针，
    if (nil != self) {// 针对获取的self指针进行判断，如果
		//... 初始化一些变量
    }
    return self;
}
@end
```



### Cocoa中的类簇

Class Clusters 是抽象工厂模式在iOS下的一种实现，众多常用类，如NSString、NSArray、NSDictionary以及NSNumber都是这种设计模式，它是接口简单性和扩展性的权衡体现。



### 为什么要调用self = [super init]

在初始化失败的时候会返回nil，你需要知道`super`对象的初始化是否失败。另外，基于类簇的设计模式，`super`可以选择将返回的`self`通过`+alloc`方法替换为另一个对象。



### 隐藏的self

每一个方法内部都有两个隐藏的参数：`self`和`_cmd`。

```objective-c
- (void)setValueToZero{
    value = 0;
}
// 这个方法其实是这样的一个函数
void setValueToZero(id self, SEL _cmd)
{
    self->value = 0;
}
```

在运行时，函数都会转化成一种发消息的方式来完成内部代码逻辑的实现，比如：

```objective-c
[myObject someMethodWithParameter:someValue];
```

在运行时会被转化成这样：

```objective-c
SEL methodSelector = @selector(someMethodWithParameter:);

objc_msgSend(myObject, methodSelector, someValue);
```

发消息的实质是方法指针（IMP）的实现，上面的msgSend可以如下面这样写，可以提升函数的调用速度：

```objective-c
SEL methodSelector = @selector(someMethodWithParameter:);

IMP someMethodFunction = class_getMethodImplementation([myObject class], methodSelector);

someMethodFunction(myObject, methodSelector, someValue);
```

myObject指针在这里就是self，self指针可以告诉方法要去哪个地方(类、结构体)获取数据，





---

https://www.cnblogs.com/breezemist/p/5477288.html

https://www.jianshu.com/p/9b36e1b636d8

http://blog.sunnyxx.com/2014/12/18/class-cluster/

https://stackoverflow.com/questions/2956943/why-should-i-call-self-super-init

https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaEncyclopedia/ClassClusters/ClassClusters.html