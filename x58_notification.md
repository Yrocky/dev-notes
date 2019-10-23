> 通知中心模式
>
> 如何封装通知中心模式
>
> RAC中的做法

---

### 通知中心设计模式





### [GLPubBus](https://github.com/Glow-Inc/GLPubSub/blob/master/README.md)

用作者的话来说，这仅仅是对通知中心的一个封装，简化了通知的订阅以及发布，只有对NSObject的一个分类。

内部通过runtime添加一个字典属性，字典以通知名为key，这个通知下的所有订阅者的集合为value，这样做并没有解决对同一个通知订阅多次产生的问题，不过通过block可以将每次的订阅回调分别处理。

使用一个`GLEvent类`封装通知相关的数据，封装订阅通知的核心代码如下：

```objective-c
- (id)subscribe:(NSString *)eventName obj:(id)obj handler:(GLEventHandler)handler {
    id observer =  [[NSNotificationCenter defaultCenter] addObserverForName:eventName object:obj queue:_pubSubQueue usingBlock:^(NSNotification *note) {
        GLEvent *event = [[GLEvent alloc] initWithName:eventName
                          obj:note.object
                          data:[note.userInfo objectForKey:kGLPubSubDataKey]];
        handler(event);
    }];
    NSMutableDictionary *subscriptions = (NSMutableDictionary *)objc_getAssociatedObject(self, &kGLPubSubSubscriptionsKey);
    if (!subscriptions) {
        subscriptions = [[NSMutableDictionary alloc] init];
        objc_setAssociatedObject(self, &kGLPubSubSubscriptionsKey, subscriptions, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    }
    NSMutableSet *observers = [subscriptions objectForKey:eventName];
    if (!observers) {
        observers = [[NSMutableSet alloc] init];
        [subscriptions setObject:observers forKey:eventName];
    }
    [observers addObject:observer];
    return observer;
}
```

由于没有提供自动取消监听的功能，所以需要手动在需要的时候执行`-unsubscribe`方法来移除监听。



### [MCNotificationController](https://github.com/macoscope/NotificationController)

由于通知中心在使用中有过多的不便，这个库使用`MCSNotificationController`可以完成通知的订阅以及自动取消。在添加“订阅者”的时候，会弱引用“订阅者”，内部使用一个字典根据通知的名以及发送者为key，通知中心返回的订阅者为value进行保存，这样在dealloc的时候就可以遍历字典内部的数据，从而达到“自动取消订阅”，另外还可以防止对同一个通知以及发送者进行多次的订阅。

添加一个通知的观察者的代码如下：

```objective-c
- (BOOL)addObserverForName:(nullable NSString *)name
                    sender:(nullable id)sender
                     queue:(nullable NSOperationQueue *)queue
                usingBlock:(void (^)(NSNotification *note))block{

  dispatch_sync(self.mapAccessQueue, ^{
    id<NSCopying> key = [[MCSNotificationKey alloc] initWithNotificationName:name sender:sender];// 根据sender以及notiName保证唯一性

    if (!self.mapNotificationKeyToToken[key]) {
      __weak typeof(self) weakSelf = self;

      id<NSObject> token = [self.notificationCenter addObserverForName:name object:sender queue:queue usingBlock:^(NSNotification *note) {
        if (weakSelf.observer) {
          block(note);
        }
      }];
      
      self.mapNotificationKeyToToken[key] = token;
    }
  });
}
```

这里面的`MCSNotificationKey`是一个私有类，根据通知名称以及订阅者生成一个唯一的key。

另外，为NSObject提供一个属性，使得每一个NSObject的实例都可以使用这个NoticeController。

上面说了，NoticeController和”订阅者“之间是弱引用关系，这样”订阅者“销毁的时候就会释放NoticeController，而在NoticeController的`dealloc`方法中，作者用来移除通知中心中的订阅者，使用一个中间类弱引用这样就可以达到自动释放订阅：

```objective-c
// in MCSNotificationController.m
- (void)dealloc{
  for (MCSNotificationKey *key in _mapNotificationKeyToToken) {
    id<NSObject> token = _mapNotificationKeyToToken[key];
    [self.notificationCenter removeObserver:token];
  }
}
```



### QTEventBus

上面两个库针对于通知中心的不友好分别完成了接口友好、自动取消订阅的需求，由他们内部的实现可以发现，要对通知中心改造，需要使用一个集合来保存订阅者，使用中间类来完成自动取消订阅。



### ReactiveCocoa



---

https://github.com/Glow-Inc/GLPubSub/blob/master/GLPubSub/NSObject%2BGLPubSub.m

https://github.com/LeoMobileDeveloper/QTEventBus