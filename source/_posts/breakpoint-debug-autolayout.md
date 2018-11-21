---
title: 断点调试 autolayout
date: 2016-09-16 13:59:15
tags:
- iOS
---


**对于 iOS 和 OS X 开发者来说，Autolayout已经逐渐变成一个至关重要的开发工具。它让多屏幕适配变得小菜一碟(peasy)，但是有些时候它还是会把我们搞疯掉，因为它总是会出现那些啰嗦又没啥用处的错误警告。**

就像这样:

```objc
Unable to simultaneously satisfy constraints.
Probably at least one of the constraints in the following list is one you don't want.
Try this:

(1) look at each constraint and try to figure out which you don't expect;
(2) find the code that added the unwanted constraint or constraints and fix it.
(Note: If you're seeing NSAutoresizingMaskLayoutConstraints that you don't understand, refer to the documentation for the UIView property translatesAutoresizingMaskIntoConstraints)

(...........)


Make a symbolic breakpoint at UIViewAlertForUnsatisfiableConstraints to catch this in the debugger.
The methods in the UIConstraintBasedLayoutDebugging category on UIView listed in <UIKit/UIView.h> may also be helpful.
```

**这么长的错误日志！！！！尼玛让谁看！！！**  
但是我们仔细看一下，观察 ```NSLayoutConstraint``` 部分。发现它倒数第二行还是有给我们点希望去解决这个错误的。在 ``` UIViewAlertForUnsatisfiableConstraints``` 添加一个Symbolic breakpoint 断点。  
既然这样，我们就去试一下...  

![](http://upload-images.jianshu.io/upload_images/56030-030886b7b8cca75d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

尼玛！！ 毛用都没有啊！！它停在了线程堆栈上，然而LLDB依旧一片黑暗...  

**于是这里就有一个小技巧(Trick)，**来让你的symbolic breakpoint变得更加有用。为你的ObjC项目添加 ```po [[UIWindow keyWindow] _autolayoutTrace]``` 
   
或者为你的Swift项目添加 ``` expr -l objc++ -O -- [[UIWindow keyWindow] _autolayoutTrace]```   

![](http://upload-images.jianshu.io/upload_images/56030-b6505c77c0923c08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在你就会在命令控制台看到你的UIView 的层次结构还有出现错误的地方。
  
```objc
UIWindow:0x7f9481c93360
|   •UIView:0x7f9481c9d680
|   |   *UIView:0x7f9481c9d990- AMBIGUOUS LAYOUT for UIView:0x7f9481c9d990.minX{id: 13}, UIView:0x7f9481c9d990.minY{id: 16}
|   |   *_UILayoutGuide:0x7f9481c9e160- AMBIGUOUS LAYOUT for _UILayoutGuide:0x7f9481c9e160.minY{id: 17}
|   |   *_UILayoutGuide:0x7f9481c9ebb0- AMBIGUOUS LAYOUT for _UILayoutGuide:0x7f9481c9ebb0.minY{id: 27}

```

**而当你在这个地方继续向下执行，它就会停在下一个你可能出现错误的地方。但是如果这样子你还是很难发现你的错误的话，你可以执行下面这个语句**

```objc
(lldb) e id $myView = (id) 0x7f9ea3d43410
(lldb) e (void)[$myView setBackgroundColor:[UIColor blueColor]]
```  

**先获取UIView，然后改变它的背景色**

你就会看到出错误的视图主动显示出来了~

**ps: 如果是手写Autolayout，推荐 [Mansory](https://github.com/SnapKit/Masonry)，实在是好用！！**

**关键是它的错误提示**  

```objc
Unable to simultaneously satisfy constraints......blah blah blah....
(
    "<NSAutoresizingMaskLayoutConstraint:0x8887740 MASExampleDebuggingView:superview.height == 416>",
    "<MASLayoutConstraint:ConstantConstraint UILabel:messageLabel.height >= 5000>",
    "<MASLayoutConstraint:BottomConstraint UILabel:messageLabel.bottom == MASExampleDebuggingView:superview.bottom - 10>",
    "<MASLayoutConstraint:ConflictingConstraint[0] UILabel:messageLabel.top == MASExampleDebuggingView:superview.top + 1>"
)

Will attempt to recover by breaking constraint
<MASLayoutConstraint:ConstantConstraint UILabel:messageLabel.height >= 5000>
```

已经把出现错误的约束显示出来了，完全不需要我们去茫茫去偶遇~




