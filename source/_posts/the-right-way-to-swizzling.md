---
title: 探究使用Method Swizzling的正确姿势
date: 2019-03-01 11:07:43
tags: 
- Runtime
- iOS
- Objective-C
- Method Swizzling
---

## 背景

初次接触到OC runtime机制的时候，应该都会被其黑魔法所折服。特别是在使用了Method Swizzling来hook某一个方法，改变一个已经存在的 selector 的实现的时候，实现AOP统计打点、APM检测、... 都成为了可能。

## 危害

然而，**越是强大的力量，背后往往会隐藏着更大的危险。**

在Stackoverflow上，有一篇文章：[《What are the Dangers of Method Swizzling in Objective C?》](https://stackoverflow.com/questions/5339276/what-are-the-dangers-of-method-swizzling-in-objective-c?answertab=active#tab-top)，里面已经把OC中方法交换的危险描述得非常清楚了，这里做个简要的概述。

### 1. Method swizzling并非是原子操作

其实95%的场景下，使用 Method Swizzling 都是安全的。因为我们一般希望在整个APP的生命周期里方法的替换是有效的，所以我们会在 `+(void)load` 方法里执行一系列的操作， 这个情况下是不会遇到并发问题的。但是如果不小心将代码写到了 `+(void)initialize` 中，就会有可能出现非常诡异的情况。
> 其实也应该尽可能少的在 `+(void)initialize` 中做操作，否则会影响启动速度。

### 2. 会更改到并非是我们自身代码的实现

其实这个也是想一下就能明白的问题。如果你在没搞清楚状况的情况下去进行方法替换，可能会影响到其他人的代码。特别是，如果重写了一个类的方法时，却没有调用父类的方法，可能就会出现问题。因此为了避免可能出现的未知情况，我们最好还是要在替换方法实现里调用一下原始实现。

### 3. 可能会存在命名冲突

在 Method Swizzling 的时候，我们一般会在新的方法前加上一个前缀。

如:

```objc
- (void)my_setFrame:(NSRect)frame {  
    // do custom work  
    [self my_setFrame:frame];  
}  
```

但是这样有一个问题，就是万一有某一个地方，也定义了 `- (void)my_setFrame:(NSRect)frame` 这个名字的方法，就可能出现问题。

因此最好的解决方式还是在于使用函数指针来解决这个问题（不过这样代码看起来就不那么OC了）。

```objc
@implementation NSView (MyViewAdditions)  
  
  
static void MySetFrame(id self, SEL _cmd, NSRect frame);  
static void (*SetFrameIMP)(id self, SEL _cmd, NSRect frame);  
  
  
static void MySetFrame(id self, SEL _cmd, NSRect frame) {  
    // do custom work  
    SetFrameIMP(self, _cmd, frame);  
}  
  
  
+ (void)load {  
    [self swizzle:@selector(setFrame:) with:(IMP)MySetFrame store:(IMP *)&SetFrameIMP];  
}  
  
  
@end  
```

作者也给出了一个比较完美的swizzle方法的定义：

```objc
typedef IMP *IMPPointer;  
  
  
BOOL class_swizzleMethodAndStore(Class class, SEL original, IMP replacement, IMPPointer store) {  
    IMP imp = NULL;  
    Method method = class_getInstanceMethod(class, original);  
    if (method) {  
        const char *type = method_getTypeEncoding(method);  
        imp = class_replaceMethod(class, original, replacement, type);  
        if (!imp) {  
            imp = method_getImplementation(method);  
        }  
    }  
    if (imp && store) { *store = imp; }  
    return (imp != NULL);  
}  
  
  
@implementation NSObject (FRRuntimeAdditions)  
+ (BOOL)swizzle:(SEL)original with:(IMP)replacement store:(IMPPointer)store {  
    return class_swizzleMethodAndStore(self, original, replacement, store);  
}  
@end  
```

### 4. 会改变方法的参数

作者认为这是一个最大的问题。当你替换了一个方法之后，其实你也替换了传入原始方法实现的参数。

```objc
[self my_setFrame:frame];
```

这一行做的事情是：

```objc
objc_msgSend(self, @selector(my_setFrame:), frame);
```

runtime会去寻找 `my_setFrame:` 的实现，一旦找到了，就会把 `my_setFrame` 和 `frame` 传入，但其实这个时候找到的方法应该是原始的 `setFrame:`，于是当它被调用的时候，`_cmd` 这个参数并不是预期 `setFrame:`，而是 `my_setFrame`，这样就接收了一个意料之外的参数。

最好的方式还是使用如上的定义。

### 5. 方法交换带来的顺序问题

当对多个类进行方法交换的时候，要注意顺序，特别是有父子类关系的时候。
比如：

```objc
[NSButton swizzle:@selector(setFrame:) with:@selector(my_buttonSetFrame:)];
[NSControl swizzle:@selector(setFrame:) with:@selector(my_controlSetFrame:)];
[NSView swizzle:@selector(setFrame:) with:@selector(my_viewSetFrame:)];
```

上述的实现，其实最终当你调用 `NSButton` 的 `setFrame` 的时候，会调用你替换的 `my_buttonSetFrame` 方法和 `NSView` 的原始的 `setFrame` 的方法。

相反的，如果顺序是这样的话：

```objc
[NSView swizzle:@selector(setFrame:) with:@selector(my_viewSetFrame:)];
[NSControl swizzle:@selector(setFrame:) with:@selector(my_controlSetFrame:)];
[NSButton swizzle:@selector(setFrame:) with:@selector(my_buttonSetFrame:)];
```

就会分别调用 `NSButton` 、 `NSControl` 和 `NSView` 的交换后的方法，这个顺序应该来说才是正确的。

所以其实这边还是建议在 `+(void)load` 方法里做方法交换，它可以保证父类的load方法在子类的方法调用前先调用，不会出错。

### 6. 会带来很多理解和调试上的不便

这个就不用多说了，特别是没有文档的时候。有时候要是遇到同事写在某一个角落里的runtime操作但是没人知道的话，搞出了一些无法预见的问题，调试起来就很是蛋疼。

## 正确姿势

所以正确的 Method Swizzling 的姿势是什么呢？

### ①

 如上面的作者一直强调的，**在load里进行方法替换**

### ② 

其实上面作者给出的 `swizzle完美定义` 已经是比较正确的姿势了。但是这里也记录下另一个问题。

网上一部分的文章，都会讲到，通过 Category 实现 Method Swizzling 的例子如下：

```objc
#import <objc/runtime.h>

@implementation UIViewController (Tracking)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];

        SEL originalSelector = @selector(viewWillAppear:);
        SEL swizzledSelector = @selector(xxx_viewWillAppear:);

        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);

        // When swizzling a class method, use the following:
        // Class class = object_getClass((id)self);
        // ...
        // Method originalMethod = class_getClassMethod(class, originalSelector);
        // Method swizzledMethod = class_getClassMethod(class, swizzledSelector);

        BOOL didAddMethod =
            class_addMethod(class,
                originalSelector,
                method_getImplementation(swizzledMethod),
                method_getTypeEncoding(swizzledMethod));

        if (didAddMethod) {
            class_replaceMethod(class,
                swizzledSelector,
                method_getImplementation(originalMethod),
                method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}

#pragma mark - Method Swizzling

- (void)xxx_viewWillAppear:(BOOL)animated {
    [self xxx_viewWillAppear:animated];
    NSLog(@"viewWillAppear: %@", self);
}

@end
```

然而这里有一点不够严谨，也是之前有提到的危险性所在，origin_imp 如果使用了 _cmd 参数，hook之后的_cmd 是不符合预期的。

假设我们现在要hook `- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event;` 

那么如果如上那么实现的话，在 `[self xxx_touchesBegan:touches withEvent:event];` 就会崩溃。原因是这个函数里有 `forwardTouchMethod`, 反汇编后实现类似：

```objc
static void forwardTouchMethod(id self, SEL _cmd, NSSet *touches, UIEvent *event) {
  // The responder chain is used to figure out where to send the next touch
    UIResponder *nextResponder = [self nextResponder];
    if (nextResponder && nextResponder != self) {
      // Not all touches are forwarded - so we filter here.
        NSMutableSet *filteredTouches = [NSMutableSet set];
        [touches enumerateObjectsUsingBlock:^(UITouch *touch, BOOL *stop) {
          // Checks every touch for forwarding requirements.
            if ([touch _wantsForwardingFromResponder:self toNextResponder:nextResponder withEvent:event]) {
                [filteredTouches addObject:touch];
            }else {
              // This is interesting legacy behavior. Before iOS 5, all touches are forwarded (and this is logged)
                if (!_UIApplicationLinkedOnOrAfter(12)) {
                    [filteredTouches addObject:touch];
                    // Log old behavior
                    static BOOL didLog = 0;
                    if (!didLog) {
                        NSLog(@"Pre-iOS 5.0 touch delivery method forwarding relied upon. Forwarding -%@ to %@.", NSStringFromSelector(_cmd), nextResponder);
                    }
                }
            }
        }];
        // here we basically call [nextResponder touchesBegan:filteredTouches event:event];
        [nextResponder performSelector:_cmd withObject:filteredTouches withObject:event];
    }
}
```

如果我们exchange了 IMP, `[nextResponder performSelector:_cmd withObject:filteredTouches withObject:event];` 是没有相应的实现的，_cmd 就变成了 我们替换的 SEL。 显然，nextResponder没有实现相应的方法，就会crash。

那么这里可以这么写：

```objc
static IMP __original_TouchesBegan_Method_Imp;


@implementation UIView (Debug)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];
        
        SEL originalSelector = @selector(touchesBegan:withEvent:);
        SEL swizzledSelector = @selector(dae_touchesBegan:withEvent:);
        
        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
        
        __original_TouchesBegan_Method_Imp = method_getImplementation(originalMethod);
        
        BOOL didAddMethod =
        class_addMethod(class,
                        originalSelector,
                        method_getImplementation(swizzledMethod),
                        method_getTypeEncoding(swizzledMethod));
        
        if (didAddMethod) {
            class_replaceMethod(class,
                                swizzledSelector,
                                method_getImplementation(originalMethod),
                                method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}

- (void)dae_touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    // custom 
    
     void (*functionPointer)(id, SEL, NSSet<UITouch *> *, UIEvent *) = (void(*)(id, SEL, NSSet<UITouch *> *, UIEvent*))__original_TouchesBegan_Method_Imp;
        
    functionPointer(self, _cmd, touches, event);
}

```

这样就能找到正确的IMP了。


## 参考链接

[Method Swizzling 的正确途径](https://junyixie.github.io/2017/12/04/safeSwizzleRSSwizzleAnalyze/#%E9%87%87%E7%94%A8Block%E6%B7%BB%E5%8A%A0%E5%AE%9E%E7%8E%B0%EF%BC%8C%E6%B2%A1%E6%9C%89%E5%91%BD%E5%90%8D%E5%86%B2%E7%AA%81%E9%97%AE%E9%A2%98)
[The Right Way to Swizzle in Objective-C](https://blog.newrelic.com/engineering/right-way-to-swizzle/)
[What are the Dangers of Method Swizzling in Objective C?](https://stackoverflow.com/questions/5339276/what-are-the-dangers-of-method-swizzling-in-objective-c?answertab=active#tab-top)


