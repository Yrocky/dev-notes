> 如何获取当前的ViewController
>
> 需要考虑的点
---

### UIWindow

在做工程中会开发一些UI组件，这些组件的特点是需要不依赖某一个控制器来在屏幕前进行显示，虽说是不依赖控制器，其实是内部需要去获取当前活跃在屏幕前的控制器。

在iOS的工程中，会有一个keyWindow用来承载视图控制器，

### currentViewController




```objective-c
- (UIViewController *)currentViewController {
    __block UIViewController *currentVC = nil;
    if ([NSThread isMainThread]) {
        @try {
            UIViewController *rootViewController = [UIApplication sharedApplication].delegate.window.rootViewController;
            if (rootViewController != nil) {
                currentVC = [self getCurrentVCFrom:rootViewController];
            }
        } @catch (NSException *exception) {
            SAError(@"%@ error: %@", self, exception);
        }
        return currentVC;
    } else {
        dispatch_sync(dispatch_get_main_queue(), ^{
            @try {
                UIViewController *rootViewController = [UIApplication sharedApplication].delegate.window.rootViewController;
                if (rootViewController != nil) {
                    currentVC = [self getCurrentVCFrom:rootViewController];
                }
            } @catch (NSException *exception) {
                SAError(@"%@ error: %@", self, exception);
            }
        });
        return currentVC;
    }
}

- (UIViewController *)getCurrentVCFrom:(UIViewController *)rootVC {
    @try {
        UIViewController *currentVC;
        if ([rootVC presentedViewController]) {
            // 视图是被presented出来的
            rootVC = [self getCurrentVCFrom:rootVC.presentedViewController];
        }
        
        if ([rootVC isKindOfClass:[UITabBarController class]]) {
            // 根视图为UITabBarController
            currentVC = [self getCurrentVCFrom:[(UITabBarController *)rootVC selectedViewController]];
        } else if ([rootVC isKindOfClass:[UINavigationController class]]){
            // 根视图为UINavigationController
            currentVC = [self getCurrentVCFrom:[(UINavigationController *)rootVC visibleViewController]];
        } else {
            // 根视图为非导航类
            if ([rootVC respondsToSelector:NSSelectorFromString(@"contentViewController")]) {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
                UIViewController *tempViewController = [rootVC performSelector:NSSelectorFromString(@"contentViewController")];
#pragma clang diagnostic pop
                if (tempViewController) {
                    currentVC = [self getCurrentVCFrom:tempViewController];
                }
            } else {
                currentVC = rootVC;
            }
        }
        
        return currentVC;
    } @catch (NSException *exception) {
        SAError(@"%@ error: %@", self, exception);
    }
}
```

