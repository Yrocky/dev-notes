> 了解**Builder设计模式**的使用场景以及用法
>
> 针对Builder模式下结合**消息转发机制**进行优化改进

---

### Builder模式

在开发中，有一个类需要在代码中进行初始化，并且这个类的属性有很多，这里假如有一个MMPerson类：

```objective-c
@interface MMPerson : NSObject
@property (nonatomic ,retain ,readonly) NSString * name;
@property (nonatomic ,assign ,readonly) NSInteger age;
@property (nonatomic ,retain ,readonly) NSString * address;

- (instancetype) initWithName:(NSString *)name age:(NSInteger)age address:(NSString *)address;
@end

@implementation MMPerson
- (instancetype) initWithName:(NSString *)name age:(NSInteger)age address:(NSString *)address{

    if (self == [super init]) {
        _name = name;
        _age = age;
        _address = address;
    }
    return self;
}
@end
```

模型的作用在项目中基本上是对数据的封装，在初始化实例对象的时候可以使用提供的指定初始化方法设置对应的属性，也可以先初始化对象，然后依次设置属性:

```objective-c
MMPerson * p1 = [[MMPerson alloc] initWithName:@"name" age:20 address:@"shanghai"];

MMPerson * p2 = [[MMPerson alloc] init];
p2.name = @"name";
p2.age = 20;
p2.address = @"shanghai";
```

这里为了保证数据的安全性，该类的属性都是readonly的，因此为p2设置属性的时候，会编译报错。那么就使用指定初始化方法来设置属性好了。但是这样有两个缺点：

* 如果一些属性可以不用设置，就需要写好多的属性组合初始化方法。另外，OC的初始化方法没有办法像swift那样为函数的参数设置默认值，不过，这一点可以在-init方法中实现


* 拓展性不好，增加、减少、修改属性的时候，如果根据参数组合的初始化方法比较多，那么改起来很麻烦，如果有继承，那么修改起来也是灾难的代码

[Builder模式](https://en.wikipedia.org/wiki/Builder_pattern)可以将一个类的构建和表示进行分离，为创建多属性的对象的时候提供灵活的方式。如果这时候增加一个builder类，来完成对MMPerson的初始化以及属性赋值:

```objective-c
@interface MMPersonBuilder : NSObject
@property (nonatomic ,retain ,readwrite) NSString * name;
@property (nonatomic ,assign ,readwrite) NSInteger age;
@property (nonatomic ,retain ,readwrite) NSString * address;

- (MMPerson *) build;
@end
@implementation MMPersonBuilder

- (MMPerson *) build{
    return [[MMPerson alloc] initWithName:self.name
                                      age:self.age
                                  address:self.address];
}
@end

// 那么MMPerson的初始化可以这样写
MMPersonBuilder * p3Builder = [[MMPersonBuilder alloc] init];
p3Builder.name = @"name";
p3Builder.age = 20;
p3Builder.address = @"shanghai";

MMPerson * p3 = [p3Builder build];
```

不过这样的写法，会发现仅仅是将MMPerson中的属性拷贝了一份，本质上没有解决增删改属性下的灾难代码，并且，这样的写法也不是很优雅，对外暴漏了builder类的-build函数，改进一下可以这样：

```objective-c
// MMPerson
+ (instancetype) personWith:(void(^)(MMPersonBuilder *builder))buildHandle{

    if (buildHandle) {
        MMPersonBuilder * builder = [[MMPersonBuilder alloc] init];
        buildHandle(builder);
        return [builder build];
    }
    return nil;
}
// 那么MMPerson的初始化现在可以这样写
MMPerson * p4 = [MMPerson personWith:^(MMPersonBuilder *builder) {
    builder.name = @"name";
    builder.age = 20;
    builder.address = @"shanghai";
}];
```

可以看出来，目前为止，添加一个Builder类很好的解决了属性过多，初始化方法不可以向下兼容的缺点。但是使用这种Builder模式的缺点有：

* 需要增加一个builder类，增加代码量，针对于需要使用builder模式的类都需要新增一个builde人类，不利于重用
* 虽然增减改属性的时候修改的代码相对会少一些，但是拓展性依然不好




### 使用协议改善Builder模式

为了不增加一个中间的builder类，可以使用协议这种弱类型来完成这样的需求。假如有这样的一个协议：

```objective-c
@protocol MMPersonBuilder <NSObject>

@property (nonatomic ,retain ,readwrite) NSString * name;
@property (nonatomic ,assign ,readwrite) NSInteger age;
@property (nonatomic ,retain ,readwrite) NSString * address;
@end
```

那么对于MMPerson的初始化方法进行改善，可以这样：

```objective-c
- (instancetype) initWithBuilderProtocol:(void(^)(id<MMPersonBuilder>builder))buildHandle{

    if (self == [super init]) {
        if (buildHandle) {
            id<MMPersonBuilder> _builder;// todo-初始化遵守 MMPersonBuilder 协议的类
            buildHandle(_builder);
            _name = _builder.name;
            _age = _builder.age;
            _address = _builder.address;
        }
    }
    return self;
}
```

那么问题来了，由于使用协议本来就是为了消除builder类，在上面第5行，却需要一个遵守 `MMPersonBuilder` 协议的类进行初始化，好像陷入了死循环。

基于runtime的特性，所有的数据在运行时才会确定具体的类型，所以在这里可以使用一个id类型来替换 `id<MMPersonBuilder>`，具体这个id类型的类（假如叫做AHKForwarder）需要准守每一个通过它builder实例的类（这里是MMPerson类）的对应协议（这里是MMPersonBuilder协议），基于builder模式下的约定，协议（MMPersonBuilder）中的属性和类（MMPerson）中需要设置的属性是一致的。那么就可以通过消息转发，实现一个不需要为类（MMPerson类）创建对应的builder类（这里是MMPersonBuilder类），仅仅需要创建对应的协议（这里是MMPersonBuilder协议）即可。



### 消息转发

如果在编译期间向对象发送了其无法解读的消息并没什么大碍，运行时对象接收到无法解读的方法后，它将开启消息转发机制，来完成一些补救措施，让程序员有机会在crash前做一些操作。

具体有四个方法，分为三步：

1. ==- (BOOL)resolveInstanceMethod:(SEL)sel==：对象收到未知消息后，首先会调用该方法，参数就是未知消息的 selector，返回值则表示`能否新增一个实例方法处理 selector 参数`。如果这一步成功处理了 selector 后，返回 `YES`，后续的转发机制不再进行。
   `+ (BOOL)resolveClassMethod:(SEL)sel`：和上面方法类似，区别就是上面是实例方法，这个是类方法。
2. ==- (id)forwardingTargetForSelector:(SEL)aSelector==：这个方法`提供处理未知消息的备援接收者`，这个比 `forwardInvocation:` 标准转发机制更快。通常可以用这个方案来**模拟多继承的某些特性**。这一步我们无法操作转发的消息，如果想修改消息的内容，则应该开启完整的消息转发流程来实现。
3. ==- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector==：如果消息转发的算法执行到这一步，代表已经开启了完整的消息转发机制，这个方法返回 `NSMethodSignature` 对象，其中包含了指定 selector 参数中的有关方法的描述，在消息转发流程中，如果需要创建 `NSInvocation` 对象也需要重写这个方法，`NSInvocation` 对象包含了 SEL、Target 和参数。
4. ==- (void)forwardInvocation:(NSInvocation *)anInvocation==：方法的实现通常需要完成以下任务：找出能够处理 `anInvocation` 对象封装的消息的对象；使用 `anInvocation` 给前面找出的对象发送消息，`anInvocation` 会保存返回值，运行时会将返回值发送给原来的 sender。其实通过简单的改变调用目标，然后在改变后的目标上调用，该方法就能实现与 `forwardingTargetForSelector:` 一样的行为，然而基本不这样做。

![msg](https://github.com/aozhimin/iOS-Debug-Hacks/raw/master/Images/message_forward.png)

由于**在开启完整的消息转发机制以前，我们无法操作转发的消息**，所以这里使用后两个方法来完成这个改善Builder模式的任务。

为NSObject添加一个分类：

```objective-c
// interface 
@interface NSObject (AHKBuilder)
- (instancetype)initWithBuilder_ahk:(void (^)(id))builderBlock;
@end

// implementation
- (instancetype)initWithBuilder_ahk:(void (^)(id))builderBlock{
  NSParameterAssert(builderBlock);
  self = [self init];
  if (self) {
    AHKForwarder *forwarder = [[AHKForwarder alloc] initWithTargetObject:self];
    builderBlock(forwarder);
  }
  return self;
}
```

所有的消息转发操作都在这个`AHKForwarder`类中进行，他具体的作用是：

* 1️⃣根据setter方法整理出来getter方法，并为自己增加这个对应的方法签名
* 2️⃣根据NSInvocation获取对应的方法以及参数，对原类的对象进行方法调用

源码如下：

```objective-c
@interface AHKForwarder : NSObject

@property (nonatomic, strong) id targetObject;

@end

@implementation AHKForwarder

- (instancetype)initWithTargetObject:(id)object
{
  NSParameterAssert(object);
  self = [super init];
  if (self) {
    self.targetObject = object;
  }

  return self;
}
// 这里是对AHKForwarder 类进行原有类(MMPerson)的一些属性设置，可以说都是setter方法(比如，setName、setAge)
- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel
{
  if (isSelectorASetter(sel)) {
    NSString *getterName = getterNameFromSetterSelector(sel);// 通过方法名获取对应的getter方法
    Method method = class_getInstanceMethod([self.targetObject class], NSSelectorFromString(getterName));// 通过runtime，在原类(这里是MMPerson类)中getter方法对应的Method

    const NSInteger stringLength = 255;
    char dst[stringLength];
    method_getReturnType(method, dst, stringLength);// 获取返回类型

    NSString *returnType = @(dst);
    NSString *objCTypes = [@"v@:" stringByAppendingString:returnType];

    return [NSMethodSignature signatureWithObjCTypes:[objCTypes UTF8String]];// 返回对应的getter方法的方法签名，也就是为AHKForwarder类添加一个getter方法签名，用于下面的 forwardInvocation: 方法
  } else {// 如果不是setter方法(有可能是协议中的一个方法，比如 -logInfo 函数、-crash函数)，通过target对象在它的方法签名列表中根据sel查找是否有这个方法，如果没有，会crash；如果有，返回对应的方法签名
    return [self.targetObject methodSignatureForSelector:sel];
  }
}

// 上面返回的方法签名会将里面的内容赋给 NSInvocation ,invocation 里面有对应的方法名、目标对象、参数等信息
/**
<NSInvocation: 0x604000078a80>
    return value: {v} void
    target: {@} 0x60800001bfe0  (这里是AHKForwarder类的实例)
    selector: {:} setName:
    argument 2: {@} 0x107ba62a8
*/
- (void)forwardInvocation:(NSInvocation *)invocation
{
  if (isSelectorASetter(invocation.selector)) {
    NSString *getterName = getterNameFromSetterSelector(invocation.selector);
    id argument = [invocation ahk_argumentAtIndex:2];// 获取第二个参数，参考源码
    [self.targetObject setValue:argument forKey:getterName];// 使用KVC，对目标对象进行设置，将值设置给对应的属性
  } else {// 如果不是setter方法，上一步也返回了，说明targetObject中有对应的方法，就交给targetObject来处理这个方法
    invocation.target = self.targetObject;
    [invocation invoke];
  }
}

@end

static  NSString *getterNameFromSetterSelector(SEL selector) {
  const NSString *setterName = NSStringFromSelector(selector);
  NSString *getterName = [setterName substringFromIndex:3];
  getterName = [getterName stringByReplacingCharactersInRange:NSMakeRange(0, 1) withString:[[getterName substringToIndex:1] lowercaseString]];
  getterName = [getterName substringToIndex:getterName.length - 1];

  return getterName;
}

static BOOL isSelectorASetter(SEL selector) {
  // simple implementation, won't work with custom setter names
  NSString *selectorString = NSStringFromSelector(selector);
  return [selectorString hasPrefix:@"set"];
}
```

改进后的Builder模式，不需要添加对应的builder类，只需要添加对应的协议即可:

```objective-c
MMPerson * p6 = [[MMPerson alloc] initWithBuilder_ahk:^(id<MMPersonBuilder>builder) {
    builder.name = @"name";
    builder.age = 20;
    builder.address = @"shanghai";
    [builder logInfo];
//  [builder crash];// 打开这个代码， 运行会崩溃，因为 MMPerson 类没有实现对应的方法
}];
```

这个方法目前有两个问题：不能用于block类型的属性、不能用于自定义setter方法的属性。



### 关于Builder模式的拓展

在上面Builder模式部分，还可以使用另外一种方式进行改善，使得Builder模式具备链式调用的功能。还是要新增一个builder类，这个类的作用是对类的创建：

```objective-c
// MMPersonBuilder 类内部
- (MMPersonBuilder *(^)(NSString *name))configName{
    return ^MMPersonBuilder *(NSString *name ){
        self.name = name;
        return self;
    };
}
- (MMPersonBuilder *(^)(NSInteger age))configAge{
    return ^MMPersonBuilder *(NSInteger age){
        self.age = age;
        return self;
    };
}
- (MMPersonBuilder *(^)(NSString *address))configAddress{
    return ^MMPersonBuilder *(NSString *address ){
        self.address = address;
        return self;
    };
}
/// 创建MMPerson可以这样
MMPerson * p5 = [MMPersonBuilder new].configName(@"name").configAge(20).configAddress(@"shanghai").build;
```



---

http://holko.pl/2015/05/12/immutable-object-initialization/