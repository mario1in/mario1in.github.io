---
title: 一个奇怪的旋转问题
date: 2019-07-30 10:24:32
tags:
- iOS
- Orientation
---

## 现象

最近发现一个很神奇的bug，加入某个会议之后然后退出，重复3次，界面就无法旋转了。

## 猜测

### 猜测1⃣️
无法旋转，在确认旋转锁定关闭的情况下，第一个想法是，看看support orientation 相关的函数返回值是否正确。

```
func application(_ application: UIApplication, supportedInterfaceOrientationsFor window: UIWindow?) -> UIInterfaceOrientationMask

open override var shouldAutorotate: Bool 

open override var supportedInterfaceOrientations: UIInterfaceOrientationMask
```

于是打了三个全局断点，一个一个调试...

果不其然...
都**没什么问题**...

但是这里有个现象，当出现bug（即无法旋转）的时候，这些函数都不会被调用了。

### 猜测2⃣️

如果不是支持方向不对的话，回头看了一下现象：重复三次...
于是继续猜测，会不会跟内存泄漏有关，之前有遇到过内存相关问题的现象也是重复操作n次出现。

于是调试了一下...

果不其然...
还是**没什么问题**...

## 寻找思路

前面的猜测都不对，一下子就懵逼了，毫无头绪。后来仔细想了想，要不试试从系统接口找找问题，既然要旋转，最后肯定会落地到系统层面的某个方法上。

于是打了个断点在 `-[UIViewController shouldAutorotate]`

当设备可以旋转的时候，调用栈如下:

```c
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
  * frame #0: 0x00000001d039de80 UIKitCore`-[UIViewController shouldAutorotate]
    frame #1: 0x00000001d039f8b8 UIKitCore`-[UIViewController _updateLastKnownInterfaceOrientationOnPresentionStack:] + 144
    frame #2: 0x00000001d03a0e84 UIKitCore`-[UIViewController window:willAnimateRotationToInterfaceOrientation:duration:newSize:] + 92
    frame #3: 0x00000001d03a8270 UIKitCore`__95-[UIViewController(AdaptiveSizing) _window:viewWillTransitionToSize:withTransitionCoordinator:]_block_invoke.3392 + 48
    frame #4: 0x00000001d03af570 UIKitCore`-[_UIViewControllerTransitionCoordinator _applyBlocks:releaseBlocks:] + 264
    frame #5: 0x00000001d03abb64 UIKitCore`-[_UIViewControllerTransitionContext __runAlongsideAnimations] + 176
    frame #6: 0x00000001d0da7f40 UIKitCore`+[UIView(UIViewAnimationWithBlocks) _setupAnimationWithDuration:delay:view:options:factory:animations:start:animationStateGenerator:completion:] + 608
    frame #7: 0x00000001d0da8480 UIKitCore`+[UIView(UIViewAnimationWithBlocks) animateWithDuration:delay:options:animations:completion:] + 108
    frame #8: 0x000000010710a74c Glip`+[UIView(instrumentation) ADEumAnimateWithDuration:delay:options:animations:completion:] + 300
    frame #9: 0x00000001d03c29e8 UIKitCore`__58-[_UIWindowRotationAnimationController animateTransition:]_block_invoke_2 + 308
    frame #10: 0x00000001d0dac560 UIKitCore`+[UIView(Internal) _performBlockDelayingTriggeringResponderEvents:] + 220
    frame #11: 0x00000001d03c2778 UIKitCore`__58-[_UIWindowRotationAnimationController animateTransition:]_block_invoke + 128
    frame #12: 0x00000001d0da7f40 UIKitCore`+[UIView(UIViewAnimationWithBlocks) _setupAnimationWithDuration:delay:view:options:factory:animations:start:animationStateGenerator:completion:] + 608
    frame #13: 0x00000001d0da8480 UIKitCore`+[UIView(UIViewAnimationWithBlocks) animateWithDuration:delay:options:animations:completion:] + 108
    frame #14: 0x000000010710a74c Glip`+[UIView(instrumentation) ADEumAnimateWithDuration:delay:options:animations:completion:] + 300
    frame #15: 0x00000001d03c2650 UIKitCore`-[_UIWindowRotationAnimationController animateTransition:] + 456
    frame #16: 0x00000001d0968398 UIKitCore`-[UIWindow _rotateToBounds:withAnimator:transitionContext:] + 580
    frame #17: 0x00000001d096aafc UIKitCore`-[UIWindow _rotateWindowToOrientation:updateStatusBar:duration:skipCallbacks:] + 1184
    frame #18: 0x00000001d096b1b0 UIKitCore`-[UIWindow _setRotatableClient:toOrientation:updateStatusBar:duration:force:isRotating:] + 516
    frame #19: 0x00000001d096a5b0 UIKitCore`-[UIWindow _setRotatableViewOrientation:updateStatusBar:duration:force:] + 128
    frame #20: 0x00000001d096925c UIKitCore`__57-[UIWindow _updateToInterfaceOrientation:duration:force:]_block_invoke + 124
    frame #21: 0x00000001d0969160 UIKitCore`-[UIWindow _updateToInterfaceOrientation:duration:force:] + 560
    frame #22: 0x00000001a43595bc CoreFoundation`__CFNOTIFICATIONCENTER_IS_CALLING_OUT_TO_AN_OBSERVER__ + 20
    frame #23: 0x00000001a4359588 CoreFoundation`___CFXRegistrationPost_block_invoke + 64
    frame #24: 0x00000001a4358a7c CoreFoundation`_CFXRegistrationPost + 392
    frame #25: 0x00000001a4358728 CoreFoundation`___CFXNotificationPost_block_invoke + 96
    frame #26: 0x00000001a42d2524 CoreFoundation`-[_CFXNotificationRegistrar find:object:observer:enumerator:] + 1496
    frame #27: 0x00000001a43581d8 CoreFoundation`_CFXNotificationPost + 696
    frame #28: 0x00000001a4d40814 Foundation`-[NSNotificationCenter postNotificationName:object:userInfo:] + 68
    frame #29: 0x00000001d05c2030 UIKitCore`-[UIDevice setOrientation:animated:] + 328
    frame #30: 0x00000001d01f071c UIKitCore`__124-[_UICanvasDeviceOrientationSettingsDiffAction _updateDeviceOrientationWithSettingObserverContext:canvas:transitionContext:]_block_invoke + 88
    frame #31: 0x00000001d01f2304 UIKitCore`_performChangesWithTransitionContext + 836
    frame #32: 0x00000001d01f0688 UIKitCore`-[_UICanvasDeviceOrientationSettingsDiffAction _updateDeviceOrientationWithSettingObserverContext:canvas:transitionContext:] + 236
    frame #33: 0x00000001d01f058c UIKitCore`__133-[_UICanvasDeviceOrientationSettingsDiffAction performActionsForCanvas:withUpdatedScene:settingsDiff:fromSettings:transitionContext:]_block_invoke + 104
    frame #34: 0x00000001d01f1fb0 UIKitCore`_performActionsWithDelayForTransitionContext + 112
    frame #35: 0x00000001d01f04e0 UIKitCore`-[_UICanvasDeviceOrientationSettingsDiffAction performActionsForCanvas:withUpdatedScene:settingsDiff:fromSettings:transitionContext:] + 172
    frame #36: 0x00000001d01f5d84 UIKitCore`-[_UICanvas scene:didUpdateWithDiff:transitionContext:completion:] + 360
    frame #37: 0x00000001d05253b8 UIKitCore`-[UIApplicationSceneClientAgent scene:handleEvent:withCompletion:] + 464
    frame #38: 0x00000001a6d63920 FrontBoardServices`__80-[FBSSceneImpl updater:didUpdateSettings:withDiff:transitionContext:completion:]_block_invoke_3 + 224
    frame #39: 0x00000001356e4c74 libdispatch.dylib`_dispatch_client_callout + 16
    frame #40: 0x00000001356e8840 libdispatch.dylib`_dispatch_block_invoke_direct + 232
    frame #41: 0x00000001a6d9d0bc FrontBoardServices`__FBSSERIALQUEUE_IS_CALLING_OUT_TO_A_BLOCK__ + 40
    frame #42: 0x00000001a6d9cd58 FrontBoardServices`-[FBSSerialQueue _performNext] + 408
    frame #43: 0x00000001a6d9d310 FrontBoardServices`-[FBSSerialQueue _performNextFromRunLoopSource] + 52
    frame #44: 0x00000001a437a2bc CoreFoundation`__CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__ + 24
    frame #45: 0x00000001a437a23c CoreFoundation`__CFRunLoopDoSource0 + 88
    frame #46: 0x00000001a4379b24 CoreFoundation`__CFRunLoopDoSources0 + 176
    frame #47: 0x00000001a4374a60 CoreFoundation`__CFRunLoopRun + 1004
    frame #48: 0x00000001a4374354 CoreFoundation`CFRunLoopRunSpecific + 436
    frame #49: 0x00000001a657479c GraphicsServices`GSEventRunModal + 104
    frame #50: 0x00000001d092bb68 UIKitCore`UIApplicationMain + 212
    frame #51: 0x000000010525d370 Glip`main at AppDelegate.swift:67:7
    frame #52: 0x00000001a3e3a8e0 libdyld.dylib`start + 4
```

最后通过断点定位，发现问题出在

```c
frame #22: 0x00000001a43595bc CoreFoundation`__CFNOTIFICATIONCENTER_IS_CALLING_OUT_TO_AN_OBSERVER__ + 20
frame #23: 0x00000001a4359588 CoreFoundation`___CFXRegistrationPost_block_invoke + 64
frame #24: 0x00000001a4358a7c CoreFoundation`_CFXRegistrationPost + 392
frame #25: 0x00000001a4358728 CoreFoundation`___CFXNotificationPost_block_invoke + 96
frame #26: 0x00000001a42d2524 CoreFoundation`-[_CFXNotificationRegistrar find:object:observer:enumerator:] + 1496
frame #27: 0x00000001a43581d8 CoreFoundation`_CFXNotificationPost + 696
frame #28: 0x00000001a4d40814 Foundation`-[NSNotificationCenter postNotificationName:object:userInfo:] + 68
frame #29: 0x00000001d05c2030 UIKitCore`-[UIDevice setOrientation:animated:] + 328
```

不论APP是否支持旋转，`[UIDevice setOrientation:animated:]` 都会调用，且参数值也是对的。关键在于下一步，当APP支持旋转的时候，会发出一个`UIDeviceOrientationDidChangeNotification` 通知，不支持旋转的时候就不会。

于是问题就转化成了：什么情况下，系统有可能会不发出 `UIDeviceOrientationDidChangeNotification` 通知呢？

遇事不决，看文档。

在 `UIDevice.h` 中找到这三个玩意儿

```objc
@property(nonatomic,readonly,getter=isGeneratingDeviceOrientationNotifications) BOOL generatesDeviceOrientationNotifications __TVOS_PROHIBITED;
- (void)beginGeneratingDeviceOrientationNotifications __TVOS_PROHIBITED;      // nestable
- (void)endGeneratingDeviceOrientationNotifications __TVOS_PROHIBITED;
```

看着应该就是罪魁祸首了。

## 定位问题

调试发现，在APP支持旋转时 `isGeneratingDeviceOrientationNotifications = true`；
不支持时  `isGeneratingDeviceOrientationNotifications=false`。

查看 `generatesDeviceOrientationNotifications` 文档：
> If the value of this property is YES, the shared UIDevice object posts a UIDeviceOrientationDidChangeNotification notification when the device changes orientation. If the value is NO, it generates no orientation notifications. Device orientation notifications can only be generated between calls to the beginGeneratingDeviceOrientationNotifications and endGeneratingDeviceOrientationNotifications methods.

查看 `beginGeneratingDeviceOrientationNotifications` 文档：
> You must call this method before attempting to get orientation data from the receiver. This method enables the device’s accelerometer hardware and begins the delivery of acceleration events to the receiver. The receiver subsequently uses these events to post UIDeviceOrientationDidChangeNotification notifications when the device orientation changes and to update the orientation property.
You may nest calls to this method safely, but you should always match each call with a corresponding call to the endGeneratingDeviceOrientationNotifications method.

所以 `isGeneratingDeviceOrientationNotifications` 的值是受另外两个方法影响的。

**看来已经找到原因了**

继续调试发现，项目代码内部并没有调用到 `beginGeneratingDeviceOrientationNotifications` 和 `endGeneratingDeviceOrientationNotifications`。

## 结论

当 `-[UIWindow setRootViewController:]`的时候，系统会调用 `beginGeneratingDeviceOrientationNotifications`，当 `-[UIWindow dealloc]` 时，会调用 `endGeneratingDeviceOrientationNotifications`，这么看系统的这个逻辑是没什么问题的。
问题出在于 WebRTC 中，在 start video session 的时候有调用 `beginGeneratingDeviceOrientationNotifications`，stop video session 的时候有调用 `endGeneratingDeviceOrientationNotifications`。但是CoreLib里的代码有点问题，start video session 的时候并没有调用 `beginGeneratingDeviceOrientationNotifications` ，导致这两个没有成对出现，从而出错。


以前没有特别关注过这三者：

```objc
@property(nonatomic,readonly,getter=isGeneratingDeviceOrientationNotifications) BOOL generatesDeviceOrientationNotifications __TVOS_PROHIBITED;
- (void)beginGeneratingDeviceOrientationNotifications __TVOS_PROHIBITED;      // nestable
- (void)endGeneratingDeviceOrientationNotifications __TVOS_PROHIBITED;
```

以后遇到旋转问题，又多了一条思路了..