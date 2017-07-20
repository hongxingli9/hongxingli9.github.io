---
layout: post
title:  "Current ViewController"
date:   2017-07-20
categories: ios
---

在今天的集成支持工作中，开发者反馈一个问题:他通过UINavigationController presentViewController一个ViewController_1,然后在这个ViewController_1上调用我们出视频广告的接口(就是再present一个ViewController)。接口是通过[UIApplication getUIViewController]来呈现视频的，先来看看这个方法是如何实现的:

```obj-c
+(UIViewController *)getUIViewController{
    UIViewController *controller = nil;
    UIWindow *window = [[UIApplication sharedApplication]keyWindow];
    if (window.windowLevel!=UIWindowLevelNormal) {
        NSArray *windows = [[UIApplication sharedApplication]windows];
        for (UIWindow *tmpWin in windows) {
            if (tmpWin.windowLevel == UIWindowLevelNormal) {
                window = tmpWin;
                break;
            }
        }
    }
    UIView *frontView = [[window subviews]objectAtIndex:0];
    id nextResponder = [frontView nextResponder];
    if ([nextResponder isKindOfClass:[UIViewController class]]) {
        controller = nextResponder;
    }else{
        controller = window.rootViewController;
    }
   
    return controller;
}
```

所以return回来的是UINavigationController, 它再去presentViewController时候就会失败，出现error: Attempt to present viewcontroller on UINavigationController whose view is not in the window hierarchy。

那我们要做的就是要让方法返回ViewController_1, 看下面如何实现:

```obj-c
+(UIViewController *)getUIViewController{
    UIViewController *controller = nil;
    UIWindow *window = [[UIApplication sharedApplication]keyWindow];
    if (window.windowLevel!=UIWindowLevelNormal) {
        NSArray *windows = [[UIApplication sharedApplication]windows];
        for (UIWindow *tmpWin in windows) {
            if (tmpWin.windowLevel == UIWindowLevelNormal) {
                window = tmpWin;
                break;
            }
        }
    }
    UIView *frontView = [[window subviews]objectAtIndex:0];
    id nextResponder = [frontView nextResponder];
    if ([nextResponder isKindOfClass:[UIViewController class]]) {
        controller = nextResponder;
    }else{
        controller = window.rootViewController;
    }
    return [self topViewController:controller];
}

+ (UIViewController *)topViewController:(UIViewController *)rootViewController
{
    if (rootViewController.presentedViewController == nil) {
        return rootViewController;
    }
    
    if ([rootViewController.presentedViewController isKindOfClass:[UINavigationController class]]) {
        UINavigationController *navigationController = (UINavigationController *)rootViewController.presentedViewController;
        UIViewController *lastViewController = [[navigationController viewControllers] lastObject];
        return [self topViewController:lastViewController];
    }
    
    UIViewController *presentedViewController = (UIViewController *)rootViewController.presentedViewController;
    return [self topViewController:presentedViewController];
}
```

最后可以成功返回ViewController_1,视频广告也可以正常呈现。

以前在自己测试的demo上是把viewcontroller push进navigationController的，所以改进前的方法也是可以正常呈现广告的, 因为navigationController此时没有presentViewController.