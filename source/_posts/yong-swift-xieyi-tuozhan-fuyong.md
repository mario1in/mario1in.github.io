---
title: 用 Swift 协议扩展和泛型来实现复用[译]
date: 2016-11-11 20:56:15
tags:
- iOS
- Swift
---


作为一个iOS开发者，最常用的任务就是通过自定义cell的子类，来实现UITableView或者UICollectionView的自定义。并且 `UITableView` 和 `UICollectionView` 在注册自定义cell子类这一块都有非常类似的API:

```swift
public func registerClass(cellClass: AnyClass?, forCellWithReuseIdentifier identifier: String)
public func registerNib(nib: UINib?, forCellWithReuseIdentifier identifier: String)
```

对于注册cell的自定义最常用的解决办法就是，声明一个reuseIdentifier的常量，像下面这样: 

```swift
private let reuseIdentifier = "BookCell"

class BookListViewController: UIViewController, UICollectionViewDataSource {

    @IBOutlet private weak var collectionView: UICollectionView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let nib = UINib(nibName: "BookCell", bundle: nil)
        self.collectionView.registerNib(nib, forCellWithReuseIdentifier: reuseIdentifier)
    }

    func collectionView(collectionView: UICollectionView, cellForItemAtIndexPath indexPath: NSIndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCellWithReuseIdentifier(reuseIdentifier, forIndexPath: indexPath)
    
        if let bookCell = cell as? BookCell {
            // TODO: configure cell
        }
    
        return cell
    }
}
```

接下来让我们尝试着使用泛型来让它简单化和安全化。  
  
首先，如果在我们的代码当中能不需要到处声明reuse identifier常量，那就再好不过了。而事实上，我们可以直接使用自定义cell的类名来当做   **默认的reuseIdentifier**。  
我们可以通过创建一个Reuseable Views的协议并且创建默认的声明方法给 `UIView` 的子类们。

```swift
protocol ReusableView: class {
    static var defaultReuseIdentifier: String { get }
}

extension ReusableView where Self: UIView {
    static var defaultReuseIdentifier: String {
        return NSStringFromClass(self)
    }
}

extension UICollectionViewCell: ReusableView {
}
```

通过让 `UICollectionViewCell` 遵循 `ReusableView` 协议，我们可以得到每个cell子类的一个唯一的重用标识。

```swift
let identifier = BookCell.defaultReuseIdentifier
// identifier = "MyModule.BookCell"
```

接下来，我们通过同样的方法，将注册Nib步骤中的一些脏代码给去除掉。  

我们创建一个 **Nib Loadable Views** 的协议并通过协议拓展添加一个默认方法实现。

```swift
protocol NibLoadableView: class {
    static var nibName: String { get }
}

extension NibLoadableView where Self: UIView {
    static var nibName: String {
        return NSStringFromClass(self).componentsSeparatedByString(".").last!
    }
}

extension BookCell: NibLoadableView {
}
```

通过让我们的 `BookCell` 类遵循 `NibLoadableView` 协议，现在我们就有了一个更安全和方便的方法去获得到Nib的名称。

```swift
let nibName = BookCell.nibName
// nibName = "BookCell"
```

有这两个协议，我们可以通过使用 **Swift的泛型** 并且通过拓展 `UICollectionView` 来简化cell的注册和使用。

```swift
extension UICollectionView {
    
    func register<T: UICollectionViewCell where T: ReusableView>(_: T.Type) {
        registerClass(T.self, forCellWithReuseIdentifier: T.defaultReuseIdentifier)
    }
    
    func register<T: UICollectionViewCell where T: ReusableView, T: NibLoadableView>(_: T.Type) {
        let bundle = NSBundle(forClass: T.self)
        let nib = UINib(nibName: T.nibName, bundle: bundle)
        
        registerNib(nib, forCellWithReuseIdentifier: T.defaultReuseIdentifier)
    }
    
    func dequeueReusableCell<T: UICollectionViewCell where T: ReusableView>(forIndexPath indexPath: NSIndexPath) -> T {
        guard let cell = dequeueReusableCellWithReuseIdentifier(T.defaultReuseIdentifier, forIndexPath: indexPath) as? T else {
            fatalError("Could not dequeue cell with identifier: \(T.defaultReuseIdentifier)")
        }
        
        return cell
    }    
}
```

注意这里，我们创建了两个版本的注册方法，一个是用来注册 `ReusableView` 子类用的，一个是用来注册 `ReusableView` 和 `NibLoadableView`的子类。这很好的将view controller的特定的注册方法分离出来。

另一个比较棒的细节就是 `dequeueReusableCell` 方法不再需要给他任何重用标识字符串而且可以直接使用cell的子类作为返回值。

现在cell的注册和使用代码看起来棒极了 :) 。

```swift
class BookListViewController: UIViewController, UICollectionViewDataSource {

    @IBOutlet private weak var collectionView: UICollectionView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.collectionView.register(BookCell.self)
    }

    func collectionView(collectionView: UICollectionView, cellForItemAtIndexPath indexPath: NSIndexPath) -> UICollectionViewCell {
        
        let cell: BookCell = collectionView.dequeueReusableCell(forIndexPath: indexPath)
        
        // TODO: configure cell
    
        return cell
    }
    ...
}
```

## 总结  
如果你是从 Objective-C 转到 Swift 的话，研究Swift强大的新特性比如 **协议拓展**、 **泛型**， 从而找到更优雅的实现方式和替代方法是非常值得的。




