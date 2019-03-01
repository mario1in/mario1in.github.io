---
title: 贝塞尔曲线指北
date: 2016-11-03 17:37:01
tags:
- 贝塞尔
- iOS
- 动画
---

最近在做项目的时候，需要用到一个动画，非常简单的动画，简单到就是直接对一个View做平移... 然而虽然动画简单，但是却很不自然，尝试了UIView Animation提供的各类参数，都无法达到想要的动画效果。这时候，我的脑子里突然想起一个词... “贝塞尔曲线”.... 这个词经常看到，但却从没有去了解过，这次就趁着有求于它的雅兴，好好做个入门了解好了。
## 什么是贝塞尔曲线？  
  显而易见的是，贝塞尔曲线，应该就是是一个叫贝塞尔的人发明的曲线吧，然而历史剧本却不是这么写的。贝塞尔曲线所依据的最原始的数学公式，是早在1912年就广为人知的伯恩斯坦多项式。OK，now，What is boensitan duoxiangshi？！简单来说，伯恩斯坦多项式可以用来证明，在\[ a, b ] 区间上所有的连续函数都可以用多项式来逼近，并且收敛性很强，也就是一致收敛。再简单点，就是一个连续函数，你可以将它写成若干个伯恩斯坦多项式相加的形式，并且，随着 n→∞，这个多项式将一致收敛到原函数，这个就是伯恩斯坦斯的逼近性质。  
  不知道在说什么鬼？没关系，接着说..  
  到了1959年，当时就职于雪铁龙的法国数学家 Paul de Casteljau 开始对伯恩斯坦多项式进行了图形化的尝试，并且提供了一种数值稳定的德卡斯特里奥（de Casteljau） 算法。根据这个算法，就可以只通过很少的控制点，去生成复杂的平滑曲线，也就是贝塞尔曲线。     
  而贝塞尔曲线的得名，得归功于1962年就职于雷诺的法国工程师皮埃尔·贝塞尔（Pierre Bézier），他使用这种方法来辅助汽车的车体工业设计，并且广泛宣传，因此大家才都称他为贝塞尔曲线  。
## 贝塞尔曲线是怎么画出来的？
首先，我们在平面内选3个不同线的点并且依次用线段连接。如下所示..  
![](56030-48977fcfcd8cd57e.png)
接着，我们在AB和BC线段上找出点D和点E，使得AD/AB = BE/BC。
![](56030-5d3e252f34e657c9.jpg)
再接着，连接DE，并在DE上找出一点F，使得DF/DE =  AD/AB = BE/BC。
![](56030-5175f6c03d4990b2.jpg)
然后，根据我们高中所学的极限的知识，让选取的点D在第一条线段上从起点A，移动到终点B，找出所有点F，并将它们连起来。最后你会发现，你得到了一条非常光滑的曲线，这条就是传说中的，贝塞尔曲线。  
看这里…  

![](56030-bb6b8c6a46f12135.gif)
 
这是二阶贝塞尔曲线。  

下面是三阶四阶和五阶。  

![](56030-f3e69b487f4e37c8.gif)  

![](56030-2d2fb8989e10f177.gif)  

![](56030-390b7b874ddd5d3d.gif)  

最后是... 一阶....  

![](56030-b65e3dd8196f4da5.gif)  

所以贝塞尔曲线的厉害之处就在这里，从1-n阶的连续函数，他都可以计算得到一条光滑曲线。
## 贝塞尔曲线有什么用？为什么经常会听到这个名称？
由于贝塞尔曲线控制简便，而且它具有很强的描述能力，因此它在工业设计上已经被广泛使用了。不仅如此，在计算机图形学领域（特别是矢量图形学），贝塞尔曲线也有着举足轻重的地位。而作为程序猿，我们经常会用贝塞尔曲线来绘图（由贝塞尔曲线画出来的图很光滑\~），来做动画（很自然的动画）等等。也就是由于它可以发挥的作用领域太广了，因此我们时不时都会听到这个名字。
## 如何使用贝塞尔曲线？
首先，要明确的一点是，对于贝塞尔曲线来说，最重要的点是，数据点和控制点。  
数据点： 指一条路径的起始点和终止点。  
控制点：控制点决定了一条路径的弯曲轨迹，根据控制点的个数，贝塞尔曲线被分为一阶贝塞尔曲线（0个控制点）、二阶贝塞尔曲线（1个控制点）、三阶贝塞尔曲线（2个控制点）等等。
而系统给我们提供了一个叫做UIBezierPath类，用它可以画简单的圆形，椭圆，矩形，圆角矩形，也可以通过添加点去生成任意的图形，还可以简单的创建一条二阶贝塞尔曲线和三阶贝塞尔曲线。
### 用法1：简单地画图形
这里的简单用法就不细讲，虽然类名叫UIBezierPath，但画圆形啥的跟贝塞尔也没啥关系，直接贴代码。  
  
* 画圆形      

```
	 UIBezierPath *bPath = [UIBezierPath bezierPathWithArcCenter:CGPointMake(300, 300) radius:50
                                              startAngle: DEGREES_TO_RADIANS(135) endAngle:M_PI*2 clockwise:YES];
     [bPath setLineWidth:5];
     //绘制
     [bPath stroke];
```

*  画椭圆

```
	 UIBezierPath *ovalPath = [UIBezierPath bezierPathWithOvalInRect:CGRectMake(200, 150, 100, 200)];
     [ovalPath setLineWidth:5];
     [ovalPath stroke];
```
     
*  画矩形

```
	 UIBezierPath *myBezierPath = [UIBezierPath bezierPathWithRect:CGRectMake(20, 20, 100, 50)];
    
     [[UIColor blackColor]setStroke];
     [myBezierPath setLineWidth:5];
     [myBezierPath stroke];
```

* 画圆角矩形

```
//UIRectCorner可以设置 哪几个角是圆角，其他不变  
	 UIBezierPath *tBPath = [UIBezierPath bezierPathWithRoundedRect:CGRectMake(220, 20, 100, 100)                                                 
	 byRoundingCorners:UIRectCornerTopLeft | UIRectCornerBottomLeft cornerRadii:CGSizeMake(20, 20)];
     
     [[UIColor greenColor]setStroke];
     [tBPath setLineWidth:5];
     [tBPath stroke];
```

*  通过任意点画任意图形

```
	 UIBezierPath* aPath = [UIBezierPath bezierPath];
	 aPath.lineWidth = 15.0;
   
     aPath.lineCapStyle = kCGLineCapButt;  //线条终点
     //round 圆形
     //butt 平的 默认值 把线连接到精准的终点
     //Square 平的，会把线延伸到终点再加上线宽的一半  
     aPath.lineJoinStyle = kCGLineJoinBevel;  //拐点处理
     //bevel 斜角斜面，角的外侧是平的不圆滑
     //miter 斜接 角的外侧是尖的
     //round 圆角
     
     //这是起点  
     [aPath moveToPoint:CGPointMake(100.0, 200.0)];
     
     //添加点  
     [aPath addLineToPoint:CGPointMake(200.0, 240.0)];
     [aPath addLineToPoint:CGPointMake(160, 340)];
     [aPath addLineToPoint:CGPointMake(40.0, 340)];
     [aPath addLineToPoint:CGPointMake(10.0, 240.0)];
     [aPath closePath]; //第五条线通过调用closePath方法得到的
       
     [aPath stroke]; //Draws line 根据坐标点连线
```
* 画二阶贝塞尔

```
	 UIBezierPath* twoPath = [UIBezierPath bezierPath];
     twoPath.lineWidth = 5.0;//宽度
     twoPath.lineCapStyle = kCGLineCapRound;  //线条拐角
     twoPath.lineJoinStyle = kCGLineJoinRound;  //终点处理
     //起始点
     [twoPath moveToPoint:CGPointMake(20, 100)];
     //添加两个控制点
     [twoPath addQuadCurveToPoint:CGPointMake(220, 100) controlPoint:CGPointMake(170, 0)];
     //划线
     [twoPath stroke];
```
* 画三阶贝塞尔
````
	 UIBezierPath* bPath = [UIBezierPath bezierPath];
     
     bPath.lineWidth = 5.0;
     bPath.lineCapStyle = kCGLineCapRound;  //线条拐角
     bPath.lineJoinStyle = kCGLineCapRound;  //终点处理
     //起始点
     [bPath moveToPoint:CGPointMake(20, 250)];
     
     //添加两个控制点
     [bPath addCurveToPoint:CGPointMake(350, 250) controlPoint1:CGPointMake(310, 200) controlPoint2:CGPointMake(210, 400)];
     [bPath stroke];
```
### 用法2：用贝塞尔曲线圆滑绘图
这个用法可以说是处女座的福音。  
假设这么一个场景：产品提了个需求，来吧，咱们来做一个你画我猜的APP。你画我猜？肯定是要先有画了。简单！新建个UIView的子类，然后在它的初始化方法中创建Path和手势。  

```
 // Create a path to connect lines
 path = [UIBezierPath bezierPath];
 // Capture touches
 UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(pan:)];
 pan.maximumNumberOfTouches = pan.minimumNumberOfTouches = 1;
 [self addGestureRecognizer:pan];
```
再将捕获到的pan事件location数据依次加入到path中，并且用直线连接两点。 
 
```
 - (void)pan:(UIPanGestureRecognizer *)pan {
     CGPoint currentPoint = [pan locationInView:self];
     if (pan.state == UIGestureRecognizerStateBegan) {
         [path moveToPoint:currentPoint];
     } else if (pan.state == UIGestureRecognizerStateChanged) {
         [path addLineToPoint:currentPoint];
     }
     [self setNeedsDisplay];
 }
```
最后画出轨迹。

```
- (void)drawRect:(CGRect)rect {
     [[UIColor blackColor] setStroke];
     [path stroke];
 }
```

最后将这个view添加到控制器上，很开心的Command + R，让程序跑起来。

开始画\~  
然后你就会发现，画出来的曲线是这样的。。  
![](56030-c2894a2979862cd1.png)

WHAT THE FXXK!!  
怎么可以有这么多锯齿。。  
所以这个时候，贝塞尔曲线就很有用了。它的定义是可以找到两点之间的光滑曲线，因为我们之前手势移动的时候，两点之间都是使用直线连接，如果我们可以使用贝塞尔曲线连接，那应该就不会出现这个问题了。  
试一下。  
首先写一个计算中点的方法，我们到时会使用这个中点作为控制点。  

```
static CGPoint midpoint(CGPoint p0, CGPoint p1) {
     return (CGPoint) {
         (p0.x + p1.x) / 2.0,
         (p0.y + p1.y) / 2.0
     };
 }
```

最后将手势处理中的连接方式替换成使用贝塞尔曲线。

```
- (void)pan:(UIPanGestureRecognizer *)pan {
     CGPoint currentPoint = [pan locationInView:self];
     CGPoint midPoint = midpoint(previousPoint, currentPoint);
 
     if (pan.state == UIGestureRecognizerStateBegan) {
         [path moveToPoint:currentPoint];
     } else if (pan.state == UIGestureRecognizerStateChanged) {
         [path addQuadCurveToPoint:midPoint controlPoint:previousPoint];
     }
     previousPoint = currentPoint;
     [self setNeedsDisplay];
 }
```
再Run一次…  
![](56030-de6fcb447ec49b87.png)

看，光滑多了\~  
**所以很多时候，当我们遇到画出的图形太不自然的时候，就可以试着用贝塞尔曲线解决这些问题，用到越高阶的曲线，画出的图形越光滑。** 

### 用法3：用贝塞尔曲线做变形
网上看到的大多数比较酷炫的动画，都是通过修改曲线的控制点，对曲线进行变形而做的。  
比如，我们要实现如下一个动画。  
![](56030-8fb7008726c62987.gif)

这个动画最难地方就是手势拖拽的时候，直线的变形，可以首先的想到的是使用贝塞尔。通过创建path，添加控制点画出曲线，然后通过更改控制点的位置来达到让曲线进行变形的目的。  
![](56030-9b189370ffbf5aec.gif)

如上图所示，这里添加了7个点，从左到右依次为l3、l2、l1、c、 r1、 r2、 r3。屏幕最左和最右两边的l3和r3没有在图中显示出来，然后我们就可以以l3和l2为控制点，从l3到l1建立一条二阶贝塞尔曲线，再以c和r1为控制点建一条从l3到r1的曲线，最后以r1和r2为控制点建一条从r1到r3的曲线。   主要代码如下：  

```
- (CGPathRef)currentPath {
     CGFloat width = self.view.bounds.size.width;
     UIBezierPath *path = [UIBezierPath bezierPath];
     
     [path moveToPoint:CGPointMake(0, 0)];
     [path addLineToPoint:CGPointMake(0, self.l3ControlPointView.center.y)];
     [path addCurveToPoint:self.l1ControlPointView.center
             controlPoint1:self.l3ControlPointView.center
             controlPoint2:self.l2ControlPointView.center];
     [path addCurveToPoint:self.r1ControlPointView.center
             controlPoint1:self.cControlPointView.center
             controlPoint2:self.r1ControlPointView.center];
     [path addCurveToPoint:self.r3ControlPointView.center
             controlPoint1:self.r1ControlPointView.center
             controlPoint2:self.r2ControlPointView.center];
     [path addLineToPoint:CGPointMake(width, 0)];
     [path closePath];
     return path.CGPath;
 }
```

建立好路径之后，就可以通过手势操作来修改控制点的坐标达到我们的目的了。  
在这里也就是修改l3到r3的中心点坐标。主要代码如下：  

```
- (void)panDidMove:(UIPanGestureRecognizer *)gesture {
     if (gesture.state == UIGestureRecognizerStateEnded ||
         gesture.state == UIGestureRecognizerStateFailed ||
         gesture.state == UIGestureRecognizerStateCancelled) {
         
     } else {
         CGFloat additionalHeight = MAX([gesture translationInView:self.view].y, 0);
         CGFloat waveHeight = MIN(additionalHeight*0.6, kMaxWaveHeight);
         CGFloat baseHeight = kMiniHeight + additionalHeight - waveHeight;
         CGFloat locationX = [gesture locationInView:gesture.view].x;
         
         [self layoutControlPoints:baseHeight waveHeight:waveHeight locationX:locationX];
         [self updateShapeLayer];
     }
 }
 
 - (void)layoutControlPoints:(CGFloat)baseHeight
                  waveHeight:(CGFloat)waveHeight
                   locationX:(CGFloat)locationX {
     CGFloat width = self.view.bounds.size.width;
     CGFloat minLeftX = MIN(locationX-width/2*0.28, 0);
     CGFloat maxRightX = MAX(width+(locationX-width)/2 *0.28, width);
     CGFloat leftPartWidth = locationX - minLeftX;
     CGFloat rightPartWidth = maxRightX - locationX;
     
     self.l3ControlPointView.center = CGPointMake(minLeftX, baseHeight);
     self.l2ControlPointView.center = CGPointMake(minLeftX+leftPartWidth*0.44, baseHeight);
     self.l1ControlPointView.center = CGPointMake(minLeftX+leftPartWidth*0.71, baseHeight+waveHeight*0.64);
     self.cControlPointView.center = CGPointMake(locationX, baseHeight+waveHeight*1.36);
     self.r1ControlPointView.center = CGPointMake(maxRightX-rightPartWidth*0.71, baseHeight+waveHeight*0.64);
     self.r2ControlPointView.center = CGPointMake(maxRightX-(rightPartWidth*0.44), baseHeight);
     self.r3ControlPointView.center = CGPointMake(maxRightX, baseHeight);
 }
 
 - (void)updateShapeLayer {
     self.shapeLayer.path = [self currentPath];
 }
```

通过这个思路，我们可以做出很多有意思而且有生命力的动画，这里一般还会经常和 `CADisplayLink` 一起用，先留个坑。 
 
### 用法4：用贝塞尔曲线做缓冲动画
做动画最主要的一点，就是要让动画达到很自然的效果。这就要涉及到一些现实中的物理知识，比如重力弹力和速度等等，所以有时候，我们需要对动画的速度进行控制，有时候需要先快再慢，有时候需要先慢再快然后再慢，有时候又需要快慢超慢非常慢...  
这个时候就不得不提到 `CAMediaTimingFunction` 。  
`CAMediaTimingFunction ` 的主要用法可以理解为我们在一个二维坐标系上建议一条或曲线或直线的函数，这个函数的斜率就是动画的速度，斜率的改变量也就是导数则为加速度。理论上来说，这个坐标系上的任何曲线都可以用来当做加速动画。然而`CAMediaTimingFunction ` 只给我们提供了一个三次贝塞尔曲线的函数，它可以生成三次贝塞尔曲线所能生成的所有缓冲函数。  
这里刚好可以介绍 ~~一个~~ 两个好用的网站：  [http://www.roblaplaca.com/examples/bezierBuilder](http://www.roblaplaca.com/examples/bezierBuilder)   
这个网站可以做到可视化的修改两个控制点，来达到生成一条三阶贝塞尔曲线，并且还会给出两个控制点的具体坐标，以及右边还可以看到这条曲线产生的动画会做怎样的速度改变。也就是说，只要我们能拿到两个控制点的坐标，就可以用来控制动画了。  
[http://easings.net](http://easings.net/#)   
这个网站提供了丰富的曲线类型可供选择，图表旁还有一个小动画预览，非常直观。
比如下面这段代码，就可以让我们把相册从4：3 切换到1：1 的时候，展示一个先快后慢的过渡效果，这个效果跟系统相机的还是蛮接近的。  

```
     CABasicAnimation *animation = [CABasicAnimation animation];
     animation.keyPath = @"borderWidth";
     animation.repeatCount = 1;
     animation.duration = 0.4;
     animation.removedOnCompletion = NO;
     animation.timingFunction = [CAMediaTimingFunction functionWithControlPoints:0 :1 :1 :1];
     animation.fillMode = kCAFillModeForwards;
     animation.fromValue = 0.f;
     animation.toValue = 40.f;
     [self.previewMask addAnimation:animation forKey:@"changeBorderWith"];
```

效果如下：
![](56030-61f16deba712d8ac.gif)

### 用法5：用贝塞尔曲线做拟合计算  
贝塞尔曲线有个非常常用的动画效果，叫MetaBall算法。什么是MetaBall？就是我们平时看到的QQ的小红点消除啦\~ 像下面这样。  
  
![](56030-88174468ef9870e3.png)

这个是怎么实现的？

#### 矩形拟合
首先我们需要了解一下简单的矩形拟合原理  

![](56030-c2aa8094cd0a6004.png)

如图所示的两个圆，我们通过给它添加一个矩形（绿色部分），矩形较短的两边分别顶住两个圆各自的一条直径上，然后通过改变矩形较长的两边的弧度（红色部分），达到拟合的效果。  

![](56030-9e2c8e268d676528.png)

这种做法当两个圆较小的时候，几乎是没有问题的。但是当圆稍微大点的时候，就会出现很明显的相交区域，拟合效果非常不好。  

![](56030-4ad6b01ea2c03caa.png)

所以这种简单的矩形拟合在圆较大的时候是很不严格的。这个时候就需要更严谨的切线拟合。  
#### 切线拟合
我们知道，之前的矩形拟合之所以才圆大的时候会出现拟合不严谨的情况。为什么？

![](56030-86bc9a9afdb15e48.png)

正如上图所示，两条曲线的画法都是由A1和B1为起点和终点，C点为控制点和A1、B2为起点和终点，C为控制点画出的二阶贝塞尔曲线。
而要做到完美的拟合，必须达到的一点要求就是，贝塞尔曲线与圆的连接点，也就是A1、B1、A2、B2，他们与控制点C的连线，一定要是圆的切线。这样就不管圆大小怎么变，都不会出现明显的相交区域了。

![](56030-8c56a1e98f432000.jpg)  
> 图片引用: http://www.jianshu.com/p/55c721887568

于是，现在解决问题的关键就转变成了：如何计算这些拟合的关键点？

![](56030-6700ecb2811dcac4.jpg)  
> 图片引用: http://pandara.xyz/2015/10/27/ios_slime

 
我们现在要做的，就是求出点ABCDMN这六个点的坐标，就可以实现完美拟合了。   
结合上面两张图，通过三角函数的各种计算，我们最终可以得到如下代码：

```
- (void)reloadBezierPath {
     CGFloat r1 = self.trailDot.frame.size.width / 2.0f;
     CGFloat r2 = self.headDot.frame.size.width / 2.0f;
     
     CGFloat x1 = self.trailDot.center.x;
     CGFloat y1 = self.trailDot.center.y;
     CGFloat x2 = self.headDot.center.x;
     CGFloat y2 = self.headDot.center.y;
     
     CGFloat distance = sqrt((x2 - x1) * (x2 - x1) + (y2 - y1) * (y2 - y1));
     
     CGFloat sinDegree = (x2 - x1) / distance;
     CGFloat cosDegree = (y2 - y1) / distance;
     
     CGPoint pointA = CGPointMake(x1 - r1 * cosDegree, y1 + r1 * sinDegree);
     CGPoint pointB = CGPointMake(x1 + r1 * cosDegree, y1 - r1 * sinDegree);
     CGPoint pointC = CGPointMake(x2 + r2 * cosDegree, y2 - r2 * sinDegree);
     CGPoint pointD = CGPointMake(x2 - r2 * cosDegree, y2 + r2 * sinDegree);
     CGPoint pointN = CGPointMake(pointB.x + (distance / 2) * sinDegree, pointB.y + (distance / 2) * cosDegree);
     CGPoint pointM = CGPointMake(pointA.x + (distance / 2) * sinDegree, pointA.y + (distance / 2) * cosDegree);
     
     UIBezierPath *path = [UIBezierPath bezierPath];
     [path moveToPoint:pointA];
     [path addLineToPoint:pointB];
     [path addQuadCurveToPoint:pointC controlPoint:pointN];
     [path addLineToPoint:pointD];
     [path addQuadCurveToPoint:pointA controlPoint:pointM];
     
     self.shapeLayer.path = path.CGPath;
 }
```

现在我们已经可以做到非常完美拟合的时候了，这时候再结合前面的通过修改控制点来实现图形曲线变换，我们就可以做到类似QQ小红点消除一样的效果了，具体做法不再赘述。

## Ending
至此，我们已基本了解了贝塞尔曲线的历史出处公式性质及各种用法。在不断学习的过程中，我发现一些比较牛逼的实现方法，都涉及到了较多较复杂的数学公式，奈何大学高数没有好好学，导致需要回头去看很多东西，这也是这篇博客耗费了较多时间的原因之一。不过在掌握了这些基础和基本用法之后，就可以再去研究一下比较高级和酷炫的用法了，也留下了很多坑，会在以后慢慢填补的…   
如果以后还想补的话....  
文中如果有什么不足之处欢迎指正，这也是Share的目的之一。  
Have fun \~  

## 参考链接
[贝塞尔曲线维基百科](https://zh.wikipedia.org/wiki/%E8%B2%9D%E8%8C%B2%E6%9B%B2%E7%B7%9A)  
[UIBezierPath Class Reference](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIBezierPath_class/)  
[贝塞尔曲线扫盲](http://www.html-js.com/article/1628)  
[自定义缓冲函数](https://zsisme.gitbooks.io/ios-/content/chapter10/custom-easing-functions.html)  
[iOS-UI进阶13 - 贝塞尔曲线和帧动画结合](http://www.jianshu.com/p/5dbdd1ee47aa)  
[贝塞尔曲线开发的艺术](http://www.jianshu.com/p/55c721887568)








