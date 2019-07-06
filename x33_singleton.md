> 什么是单例模式
>
> 如何设计一个严格单例

---

### Singleton

单例模式是设计模式中的一种，其作用主要是确保改类在项目中只有一个实例对象。要保证这个实例是唯一的，就需要在设计单例的时候确保遵循严格单例的要求，需要考虑到多种初始化方法、子类化、多线程操作、copy操作等等。



### 严格单例

要确保实例对象的唯一性，需要设置一个全局静态对象：

``` objective-c
// 单例类的静态实例对象，因对象需要唯一性，故只能是static类型
static MMSingleton *instance = nil;
```

在OC中，一个对象可以通过alloc获取内存地址，其内部实际是调用的`+allocWithZone:` 方法，所以为了保证通过alloc-init初始化的事例也具备唯一性，需要在这个方法内部使用父类的分配的内存：

``` objective-c
+(id)allocWithZone:(struct _NSZone *)zone{
    static dispatch_once_t token;
    dispatch_once(&token, ^{
        if(instance == nil){
            instance = [super allocWithZone:zone];
        }
    });
    return instance;
}
```

对于单例对象，都会为调用方提供一个特殊的方法，内部提供对单例对象的生成，同时也使用GCD来保证多线程中的唯一性，以及子类继承下的唯一性：

``` objective-c
+ (instancetype)mgr{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[MMSingleton alloc] init];
    });
    return instance;
}
```

对于可能存在的copy操作，还需要重写对应的方法来返回当前单例对象：

``` objective-c
- (id)copy{
    return self;
}
- (id)mutableCopy{
    return self;
}
```

以上操作虽然可以确保创建的对象唯一性，但需要注意的是，一般单例对象都会存储一些数据，比如有一个叫`name`的NSString属性，我们需要在`+mgr` 内部对其进行初始化，而不是重写`-init` 方法来进行初始化，一个好的做法是不允许调用`init`方法，对应的`mgr`方法也要进行修改：

``` objective-c
- (instancetype)initInstance {
    self.name = @"MMSingleton-initInstance-value";
    return [super init];
}

- (instancetype)init{
    [NSException raise:@"SingletonPattern"
                format:@"不允许调用init来初始化，请使用mgr方法来初始化."];
    return nil;
}

+ (instancetype)mgr{
    
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[MMSingleton alloc] initInstance];
    });
    return instance;
}
```

另外一点，上面虽然对子类化单例也做了唯一性的操作，但是在实际应用中，是不允许有子类化单例这种情况出现的，因此，mgr方法最好加上类型的断言：

```objective-c
+ (instancetype)mgr{
    if (self != [MMSingleton class]) {
        [NSException raise:@"SingletonPattern"
                    format:@"不允许子类化单例类."];
    }
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[MMSingleton alloc] initInstance];
    });
    return instance;
}
```



---

https://www.cnblogs.com/YouXianMing/p/4709209.html