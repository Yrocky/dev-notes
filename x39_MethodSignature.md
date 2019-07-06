> 方法签名是什么
>
> 方法签名在消息转发机制中所扮演的角色是什么

---

### objc_msgSend

我们知道，当对一个对象调用一个方法的时候，内部其实是通过runtime调用的`objc_msgSend`，



### MethodSignature

每个类会为它内部所有的方法对应一个方法签名，这个方法签名记录的是该方法的返回值和参数类型的信息，可以通过SEL来获取一个类中对应的方法签名，在Funcation中使用NSMethodSignature来表示。

方法签名主要用于消息转发机制中，比如`-methodSignatureForSelector:` 函数返回的就是在转发时候要处理的消息（方法）的签名，这一步是是用来转发对象响应不了的消息的地方，可以使用一个备援对象对这个方法进行转发，通过调用备援对象的`-methodSignatureForSelector:` 方法可以返回对应的方法签名，这样就达到了转发消息的目的。

![msg](https://github.com/aozhimin/iOS-Debug-Hacks/raw/master/Images/message_forward.png)



### methodSignatureForSelector:

既然是一个方法的签名，那么就会存在一些情况，方法声明但是没有实现，方法实现但是没有声明，这个方法可以是协议中的方法也可以是自己创建的，基于这些情况还是否可以正确的获取到对应的方法签名呢？在[这篇文章中](http://tutuge.me/2017/04/08/diy-methodSignatureForSelector/)给到的结论是：

> **对于类，只要@implementation里实现了方法，不管@interface是否有声明，都可以返回methodSignature**
>
> **对于Protocol声明的方法，不管实例方法还是类方法，不管required还是optional的，都可以返回methodSignature**

这其中的原因就需要探究`-methodSignatureForSelector:`的内部实现原理。其内部的汇编代码如下：

```objective-c
void * +[NSObject methodSignatureForSelector:](void * self, void * _cmd, void * arg2) {
    rdi = self;
    rbx = arg2;
    if ((rbx != 0x0) && (___methodDescriptionForSelector(object_getClass(rdi), rbx) != 0x0)) {
            rax = [NSMethodSignature signatureWithObjCTypes:rdx];
    }
    else {
            rax = 0x0;
    }
    return rax;
}
```

作者在一番斗智斗勇之后得出`___methodDescriptionForSelector:`内部实现流程如下：

![img](http://zorrochen.qiniudn.com/blog_RemakeMethodSignatureForSelector_2.png)

根据流程，可以得出，`___methodDescriptionForSelector:`方法：

1. 用`class_copyProtocolList`、`protocol_getMethodDescription`方法，检查是否有对应的selector，不管是否实现
2. 用`class_getInstanceMethod`检查selector是否有**implementation**，注意，是**implementation**，跟是否声明了没有关系



### invocation

在消息转发机制的最后一步`-forwardInvocation:` 中会对参数NSInvocation中的target进行备援接收者的替换。除了在消息转发机制中使用NSInvocation将消息处理交给备援对象，最多的还是使用他来替换`-performSelector:withObject:` 完成多参数方法的调用。它主要由NSTimer和分布式对象组成，内部包含有一个OC消息中所有的信息：target、selector、arguments、return-value，并且这些信息都可以进行设置，当设置好了这一系列方法之后，调用invoke就可以实现方法的调用。

```  objective-c
// 常规调用    
    NSMethodSignature *sig = [self.a methodSignatureForSelector:methodForDelegateA];
    NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:sig];
    [invocation setTarget:self.a];
    [invocation setSelector:methodForDelegateA];
    [invocation invoke];

// 多参数调用
    NSMethodSignature *sig = [self.a methodSignatureForSelector:methodForDelegateASum];
    NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:sig];
    [invocation setTarget:self.a];// argument index is 0
    [invocation setSelector:methodForDelegateASum];// argument index is 1

    [invocation setArgument:&a atIndex:2];
    [invocation setArgument:&b atIndex:3];
    [invocation setArgument:&c atIndex:4];
    [invocation invoke];
    NSUInteger d;
    // 取这个返回值
    [invocation getReturnValue:&d];
    NSLog(@"return value :%lu",(unsigned long)d);

// Class A
- (NSUInteger) a:(NSInteger)a addB:(NSInteger)b addC:(NSInteger)c{
    return a + b + c;
}
```



---

https://www.mikeash.com/pyblog/friday-qa-2011-08-05-method-signature-mismatches.html

http://tutuge.me/2017/04/08/diy-methodSignatureForSelector/