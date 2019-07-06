> 什么是切片编程：**通过预编译和运行期动态代理实现给程序动态统一添加功能的一种技术**
>
> 使用场景：日志记录、性能统计、安全控制、事务处理、异常处理等
>
> 作用：将一些与业务逻辑无关的功能通过AOP独立出来

---

### AOP

AOP(Aspect Oriented Programming，面向切面编程)，是通过预编译方式和运行期动态代理实现在不修改源代码的情况下给程序动态添加功能的一种技术。

其核心思想是将业务逻辑（核心关注点，系统的主要功能）与公共功能（横切关注点，如日志、事物等）进行分离，降低复杂性，提高软件系统模块化、可维护性和可重用性。其中核心关注点采用 OOP 方式进行代码的编写，横切关注点采用 AOP 方式进行编码，最后将这两种代码进行组合形成系统。



### 应用场景

AOP 被广泛应用在日志记录、性能统计、安全控制、事务处理、异常处理等领域。



### 在iOS开发中的实现方式

在 iOS 中 AOP 的实现是基于 Objective-C 的 **Runtime** 机制，实现 Hook 的三种方式分别为：**Method Swizzling**、**NSProxy** 和 **Fishhook**。



### Aspects

// todo 解读源码，知道内部实现逻辑



---

https://github.com/qhd/ANYMethodLog

https://github.com/steipete/Aspects

https://lision.me/aspects/

https://github.com/aozhimin/iOS-Monitor-Platform