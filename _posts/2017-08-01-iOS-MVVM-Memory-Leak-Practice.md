---
layout: post
section-type: post
title: iOS MVVM 和潜在的内存泄漏问题 in Swift 3
category: engineering
tags: ["frontend"]
---

## 背景

最近在做一个 iOS 项目，客户要求交付的时候要有相对好的测试覆盖率，可维护性。这让我想到了之前开发的时候 View Controller 测试痛苦的经历，而且到了项目后期庞大的数千行代码的 View Controller 非常难维护，即使是用了不同的 extension 来分割代码，由于基本上 90%的逻辑都会写在 ViewController 中，导致代码非常混乱。 通过改变 MVC 模式来达到更好的可测试和维护性有很多解决方案，比如[VIPER](https://objccn.io/issue-13-5/)，借鉴于 Redux 的[单向数据流动函数式](https://onevcat.com/2017/07/state-based-viewcontroller/)，MVVM 等等。我从对已有模式的兼容性和维护测试性的两个方面考虑，选择了 MVVM 作为整个 App 的前端架构模式。

## MVC vs MVVM in iOS

# MVC 简介

由于 iOS 项目创建时会默认使用 Model-View-Controller 的标准模式，开发者一般都会将他用作自己的默认开发模式。我们来看一张经典的 MVC 的架构图。

![image](https://i.stack.imgur.com/BYaCl.png)

在 MVC 中，Model 代表数据，View 代表用户界面，Controller 充当二者之间的媒介。在实际应用中，View 和 ViewController 会经常深度耦合在一起，以至于在 Xcode 中当你选择新建一个 COCOAPods 类时，他会询问你是否要新建一个与之相对应的 xib view，如下图

![image](https://preview.ibb.co/bVGJc5/Screen_Shot_2017_08_02_at_12_47_37_AM.png)

很多人会称 MVC 为 Massive View Controller, 因为在实现一个 App 的过程中很多开发者会把网络，数据，状态的处理方法都写在 ViewController 里，而造成厚 VC 薄 Model 的情况。也正是由于这种厚 VC 的情况，会让 Unit test 非常难写，因为 VC 做了太多的事情，混杂了对 UI 组件的处理，对 View 状态的维护和对数据的处理。但是也不能说这是一种错误，因为 VC 本来就是 View 和 Model 之间的胶水代码。时间长了这里就变成了意大利面代码，永远没人想知道里面发生了什么。

# MVVM 简介

MVVM 本质上是一种 MVC 的变种.，相当于把以前的 VC 拆分成 View Controller 和 View Model 两部分。 View Controller 只关心 View 的信息比如 IB Outlet，View 的生命周期等等，View Model 只关心现在的数据和状态信息。

这样的改变会让代码很好被测试，利用 Unit test 很容易测试 View Model 的数据和状态信息，而 Xcode 的 Automatic UI testing 又可以很好的帮我们处理 VC 这一层的测试。而且由于我们只是分离了 View Controller 为两层，这也不会对我们现有的代码有太多的侵入。尤其是如果我们在分离一部分 VM 层的数据状态处理逻辑代码给其他团队复用的时候，这种方式极大地方便了提取和移植。

# 总结

总结一下 MVVM 相对 MVC 的三大优势:

1. 更好的可测试性
2. 方便的代码移植
3. 良好的 MVC 兼容性

当然世界上没有绝对完美的东西，MVVM 也有他自己的缺点：

1. 学习成本高，尤其是团队里有新成员加入的时候，会比较难上手
2. MVVM 需要绑定机制来触发 ViewModel 到 ViewController 的响应，iOS 没有很好的绑定机制可以用，这也造成了我现在团队里的内存泄漏问题
3. 数据和网络相关的代码会使得 VM 这层变得很臃肿，大量的 Async 代码会让程序难维护，所以我们采用了 DataManager layer 和 Async Await in Swift 来解决这个问题

## IBM 团队 MVVM 在 iOS 的实现方法

我们团队在具体实现上面参照了 IBM 奥斯丁团队的[Bluepic 项目](https://github.com/IBM/BluePic)，这个项目曾经在 WWDC 演讲过[Going Server-side with Swift Open Source](https://developer.apple.com/videos/play/wwdc2016/415/)。 其中也介绍了 Swift 在 server side 的可能性，值得一看哦。

# 基本架构

我们受到 React 组件思维的影响，对 Bluepic 的架构做了调整，用组件来组织不同的 UI item，这一点也确实省了我们切换 ViewController，ViewModel 和 View 文件夹的时间。 而且我们团队在 Sprint 中分任务的时候也是根据组件负责来分的，这也让团队成员更专注于他们的任务

![image](https://image.ibb.co/enH8c5/Screen_Shot_2017_08_01_at_11_59_22_PM.png)

| 文件                     | 解释                                                                   |
| ------------------------ | ---------------------------------------------------------------------- |
| AppDelegate.swift        | 处理 App 生命周期                                                      |
| TestingAppDelegate.swift | 处理测试环境下 App 的生命周期                                          |
| Localizable.strings      | 多语言支持                                                             |
| Assets.xcassets/         | 图片资源                                                               |
| Components/              | UI 组件，每个组件都由一个 View Controller, View Model 和 Xib view 构成 |
| Configurations/          | 存放 info.plist 等配置文件                                             |
| DataManagers/            | 本地数据库,网络相关                                                    |
| Extensions/              | 对 Swift 原生库的扩展方法                                              |
| Models/                  | 数据结构定义                                                           |
| Storyboards/             | 存放不同环境下的 Storyboards                                           |
| Utilities/               | 工具类文件夹                                                           |

# 基本实现

![image](https://image.ibb.co/c6bjH5/Screen_Shot_2017_08_02_at_12_16_02_AM.png)

以 Category Component 为例子，他有自己的 ViewController，ViewModel 和 Xib view。 这里 Xib 就不过多解释了，里面有 autoLayout 和基本的 UI 组件。

ViewController 的实现如下,其中的 CategoryViewModelNotification 是 View Model 给 View Controller 传信息的消息约定. 在 ViewController 中会有一个 ViewModel 的 reference，以及一个用于初始化 ViewModel 的 notifyVC 方法

```swift
class CategoryViewController: UIViewController {
  var viewModel: CategoryViewModel!
  // IB Outlet
  @IBOutlet weak var textA: UILabel!
  @IBOutlet weak var textB: UILabel!

  override func viewDidLoad() {
    self.viewModel = CategoryViewModel(notifyVC: notifyVC)
    self.viewModel.updateData()
  }

  func doA() {
    // Update UI using View Model Data
    debugPrint('doA')
    textA.text = self.viewModel.getText(for: 0)
  }

  func doB() {
    // Update UI using ViewModel Data
    debugPrint('doB')
    textA.text = self.viewModel.getText(for: 1)
  }
}

// ViewModel -> View controller Communication
extension CategoryViewController {
    func notifyVC(_ notification: CategoryViewModelNotification) {
        switch notification {
        case .A:
            self.doA()
            break
        case .B:
            self.doB()
            break
        }
    }
}

```

ViewModel 的实现如下,ViewModel 会直接与 Data Manager 联系来更新当前这个 Component 的状态或者数据，然后成功后会通过初始化时 ViewController 传来的 notifyVC 来通知 View Controller 更新 UI

```swift
enum CategoryViewModelNotification {
  case A
  case B
}

class CategoryViewModel: NSObject {
  fileprivate var notifyVC: ((_ viewModelNotification: CategoryViewModelNotification) -> Void)!
  private var textList = [String]()
  init(notifyVC : @escaping (_ viewModelNotification: CategoryViewModelNotification) -> Void) {
        super.init()
        self.notifyVC = notifyVC
  }

  func setupData() {
    textList = DataManager.getData()
    self.notifyVC(.A)
    self.notifyVC(.B)
  }

  func getText(for index: Int) -> String {
    if(index >= textList.count) {
      return 'Error'
    } else {
      return textList[index]
    }
  }
}

```

# 内存泄漏问题，三角关系泛滥

以上这种模式是 BluePic 里所采用的，使用名为 notifyVC 的@escaping 函数来作为绑定媒介。在初始化的时候把定义在 ViewController 的 notifyVC 绑定到 ViewModel 里面，当 ViewModel 中的数据发生变化时，用这个绑定好的函数来进行对 ViewController 的通知。

但是这种方法，由于在绑定方法的时候，会将这个 notifyVC 函数当成参数传入。由于不像 Javascript，Swift 中没有函数对象，这里的函数会被转换为 closure。 而在这个 closure 中又保持了一个对于 ViewController self 的 strong reference,这里就会产生循环依赖关系。如下图

![image](https://preview.ibb.co/kz1Tc5/Screen_Shot_2017_08_02_at_11_38_41_PM.png)

这样每一个 Component 的 VC，VM，View 的组合都是一个个闭合的三角循环依赖关系。由于 iOS 没有垃圾回收机制而是使用的 ARC，这种循环引用会导致每次初始化一个 Component 都会导致一次内存泄漏，这在我团队的项目中很严重，每次切换语言重新渲染的时候都会有 50MB 左右的内存泄漏。

# 解决方法

我们要解决这个问题只需要打破三角关系中的一环，由于外部初始化 component 会从 ViewController 开始，销毁也一般会调用 VC 的 removeFromParentVC 方法。所以我们选择让他们的关系变成如下这样（虚线代表弱引用）

![image](https://preview.ibb.co/h3RKH5/Screen_Shot_2017_08_02_at_11_47_18_PM.png)

具体解决方法有两种

## 使用 unowned 或者 weak 关键字修饰 notifyVC 里的 self，强制 closure 使用弱引用

在 Viewcontroller 中新加一个变量

```swift
  lazy var weakNotifyVC: (_ viewModelNotification: ProductListingPageViewModelNotification) -> Void ={                         [unowned self] in
         return self.notifyVC
  }()
```

在绑定的时候把这个 weakNotifyVC 传进 ViewModel 来做初始化，这样直接就把 closure 里的 self reference 变成 weak 类型，也就解决了这个问题

## 使用 delegate 模式，把一个变量传给 ViewModel 来初始化而不是传一个函数进去

先定义一个专门用来做 VC 和 VM 绑定的 protocol

```swift
protocol NotifyVCDelegate: class {
    func notifyVC(notification: Any)
}
```

然后 ViewController 实现这个 protocol

```swift
extension CategoryViewController: NotifyVCDelegate {

    func notifyVC(notification: Any) {
        if let notifyVC = notification as? CategoryViewModelNotification {
            self.notifyVC(notifyVC)
        }
    }

    func notifyVC(_ notification: CategoryViewModelNotification) {
        switch notification {
        case .A:
            self.doA()
            break
        case .B:
            self.doB()
            break
        }
    }
}
```

并且修改 ViewModel 的初始化函数,并且在 ViewModel 中介入一个 notifyVCDelegate，这里的 notifyVC 函数复用了之前的实现，就不载多做描述了

```swift

   weak var notifyVCDelegate: NotifyVCDelegate!
   init(notifyVC : NotifyVCProtocol) {
       super.init()
       self.notifyVCDelegate = notifyVC
   }

```

并且改变 ViewController 中对 ViewModel 的初始化代码

```swift
self.viewModel = CategoryViewModel(notifyVC: self)
```

由于 self 是一个 object，Swift 会按引用传值，在 ViewModel 内部我们给他绑定了一个 weak 的引用，这样的话也就解决了之前 closure 到 ViewController 的强引用问题

# 小结

由于没有现成的绑定机制，所以 MVVM 在 iOS 中的实现要稍微复杂些。Swift 中函数是一等公民，可以把函数当成参数传递到另一个函数中，但是他会变成一个 closure，会保持对外部的引用关系，Closure 相关的内存泄漏也是一个[iOS 的经典问题](https://medium.com/@streem/understanding-memory-leaks-in-closures-48207214cba)。Swift 个人认为在不同的类的交互之间更适合于面向协议的编程方式，这种方式也更符合 ARC 的工作机制，更方便也更直观计算每个对象的 reference 数目。所以我个人比较推荐第二种解决方案。第一种解决方案更像是 OC 留下的一个语法糖，这种非 Swift 风格的代码也很有可能在之后的 Swift 更新中被重构掉。
