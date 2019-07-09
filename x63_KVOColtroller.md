> kvo是什么以及它的实现原理
>
> YYKit、KVOController中的kvo封装

---

### kvo



### YYKit

YYKit中对KVO仅仅是做了一层胶水封装，为NSObject提供一个分类，使KVO可以结合block进行使用。

```objective-c
- (void)addObserverBlockForKeyPath:(NSString *)keyPath block:(void (^)(__weak id obj, id oldVal, id newVal))block;
- (void)removeObserverBlocksForKeyPath:(NSString *)keyPath;
- (void)removeObserverBlocks;
```

主要实现思路是为NSObject提供一个字典属性，字典的key就为传入的`keyPath`，value为一个数组，数组中装的是一个中间解耦的私有类：`_YYNSObjectKVOBlockTarget`，这个类的作用就是用来将kvo的代理方法使用block进行回调转化。

这个分类的实现很巧妙，可以为使用KVO的时候要编写很多样板代码解放劳动力，因此功能上就没有KVOController那样多了，比如不支持多keyPath的观察，不是线程安全的

### KVOController

和YYKit中的思路类似，这个工具使用一个第三方类`FBKVOController`来作为要改变者的观察类，对每一个要观察的属性使用私有类`_FBKVOInfo`来进行封装，



```objective-c
- (void)observe:(nullable id)object keyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options block:(FBKVONotificationBlock)block
{
  // 创建kvoInfo
  _FBKVOInfo *info = [[_FBKVOInfo alloc] initWithController:self keyPath:keyPath options:options block:block];

  // 加锁，省略代码
  NSMutableSet *infos = [_objectInfosMap objectForKey:object];

  // check for info existence
  _FBKVOInfo *existingInfo = [infos member:info];
  if (nil != existingInfo) {// 如果已经存在，不进行添加，同时开锁，返回
    return;
  }
  
  if (nil == infos) {//这个集合如果都不存在，就初始化
    infos = [NSMutableSet set];
    [_objectInfosMap setObject:infos forKey:object];//将集合-观察的对象成组，加入到字典中
  }
  [infos addObject:info];

  // 开锁，省略代码
	// 加入到单例中
  [[_FBKVOSharedController sharedController] observe:object info:info];
}
```



---

[KVOController的github主页](https://github.com/facebook/KVOController)

[Draveness对KVOController的源码解读](https://github.com/Draveness/Analyze/blob/master/contents/KVOController/KVOController.md)