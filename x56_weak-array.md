> 数组如何对内部元素进行弱引用
>
> 弱引用数组、字典、集合的实际意义是什么

---

### 弱引用

当对一个数组放入一些对象之后，他会对这些元素进行强引用，这也是在使用数组的时候大多数情况下的需求，但是如果需要一个数组对内部的元素进行弱引用又该如何实现呢？在Funaction中就提供了这样的具备弱引用的集合类：`NSPointerArray`、`NSMapTable`、`NSHashTable`。

这三个类分别对应的是NSArray、NSDictionary、NSSet，除了可以选择对内部对象弱引用，其他用法基本上类似。需要注意的是，NSPointerArray也可用于纯指针（指针不一定是Objective-C的类），但`NSHashTable`和`NSMutableArray`类都需要它们的内容是Objective-C的对象。



### NSMapTable

NSDictionary提供了key-object的映射，从本质上讲，NSDictionary中存储的value位置是通过key来索引的，NSDictionary中要求key的值不能改变（否则value的位置会突然错误）。为了保证这一点，NSDictionary中会对key进行`copy`操作，对value进行`retain`操作。由于会复制key值，所以key最好是小巧高效的，这样复制的时候开销才不会太大，这也是为什么我们一直都是使用NSString来作为key值，另外一点是要作为key还必须准守`NSCopying协议`，NSString类就准守这个协议。

除了可以弱引用对象，NSMapTable还更适合于一般意义的映射，不仅可以实现NSDictionary这种key-object，还可以实现object-object形式的映射：

```objective-c
// key-object，类似于NSMutableDictionary
NSMapTable *keyToObjectMapping = [NSMapTable mapTableWithKeyOptions:NSMapTableCopyIn
												   valueOptions:NSMapTableStrongMemory];

// object-object
NSMapTable *objectToObjectMapping = [NSMapTable mapTableWithStrongToStrongObjects];
```



### NSDictionary

上面说了，NSDictionary会对key进行copy，而对value进行retain，如果要使用一个自定义的类来作为key，除了使其遵守NSCopying协议之外，还可以使用Core Foundation中的CFDictionary来实现，这样实现的“字典”可以决定key可以不执行copy操作而执行retain操作，结合着桥接技术，可以实现CF对象和Objective-C对象之前的自由转换。

诚然使用以上很多方法可以实现很多有独特特性的array、dictionary、set这些集合类，但是需要注意的一点是，Foundation提供的NSArray、NSDictionary、NSSet是有很大的普用性的，任何特殊化的集合类，都需要考虑到不限于多线程、内存管理等方面的问题，种种的这些考虑如果不周全就会导致崩溃等问题发生。



---

http://www.cnblogs.com/YouXianMing/p/4803290.html

http://www.cocoawithlove.com/2008/07/nsmaptable-more-than-nsdictionary-for.html?spm=a2c4e.11153940.blogcont30720.11.1fe257e2SllJTb [翻译](https://yq.aliyun.com/articles/30720?spm=a2c4e.11163080.searchblog.34.76e92ec15hwRqv)