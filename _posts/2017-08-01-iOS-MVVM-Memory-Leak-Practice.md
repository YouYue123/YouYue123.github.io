---
layout: post
section-type: post
title: iOS MVVM 和潜在的内存泄漏问题 in Swift 3
category: tech
tags: [ 'frontend']
---

## 背景

最近在做一个iOS项目，客户要求交付的时候要有相对好的测试覆盖率，可维护性。这让我想到了之前开发的时候View Controller测试痛苦的经历，而且到了项目后期庞大的数千行代码的View Controller非常难维护，即使是用了不同的extension来分割代码，由于基本上90%的逻辑都会写在ViewController中，导致代码非常混乱。 通过改变MVC模式来达到更好的可测试和维护性有很多解决方案，比如[VIPER](https://objccn.io/issue-13-5/)，借鉴于Redux的[单向数据流动函数式](https://onevcat.com/2017/07/state-based-viewcontroller/)，MVVM等等。我从对已有模式的兼容性和维护测试性的两个方面考虑，选择了MVVM作为整个App的前端架构模式。

## MVC vs MVVM in iOS

# MVC简介

由于iOS项目创建时会默认使用Model-View-Controller的标准模式，开发者一般都会将他用作自己的默认开发模式。我们来看一张经典的MVC的架构图。

![image](https://i.stack.imgur.com/BYaCl.png)

在MVC中，Model代表数据，View代表用户界面，Controller充当二者之间的媒介。在实际应用中，View和ViewController会经常深度耦合在一起，以至于在Xcode中当你选择新建一个COCOAPods类时，他会询问你是否要新建一个与之相对应的xib view，如下图

![image](https://preview.ibb.co/bVGJc5/Screen_Shot_2017_08_02_at_12_47_37_AM.png)

很多人会称MVC为 Massive View Controller, 因为在实现一个App的过程中很多开发者会把网络，数据，状态的处理方法都写在ViewController里，而造成厚VC薄Model的情况。也正是由于这种厚VC的情况，会让Unit test非常难写，因为VC做了太多的事情，混杂了对UI组件的处理，对View状态的维护和对数据的处理。但是也不能说这是一种错误，因为VC本来就是View和Model之间的胶水代码。时间长了这里就变成了意大利面代码，永远没人想知道里面发生了什么。

# MVVM简介

MVVM本质上是一种MVC的变种.，相当于把以前的VC拆分成View Controller和View Model两部分。 View Controller只关心View的信息比如IB Outlet，View的生命周期等等，View Model只关心现在的数据和状态信息。

这样的改变会让代码很好被测试，利用Unit test很容易测试View Model的数据和状态信息，而Xcode的Automatic UI testing又可以很好的帮我们处理VC这一层的测试。而且由于我们只是分离了View Controller为两层，这也不会对我们现有的代码有太多的侵入。尤其是如果我们在分离一部分VM层的数据状态处理逻辑代码给其他团队复用的时候，这种方式极大地方便了提取和移植。

# 总结

总结一下MVVM相对MVC的三大优势:
1. 更好的可测试性
2. 方便的代码移植
3. 良好的MVC兼容性

当然世界上没有绝对完美的东西，MVVM也有他自己的缺点：
1. 学习成本高，尤其是团队里有新成员加入的时候，会比较难上手
2. MVVM需要绑定机制来触发ViewModel到ViewController的响应，iOS没有很好的绑定机制可以用，这也造成了我现在团队里的内存泄漏问题
3. 数据和网络相关的代码会使得VM这层变得很臃肿，大量的Async代码会让程序难维护，所以我们采用了DataManager layer和Async Await in Swift来解决这个问题


## IBM团队MVVM在iOS的实现方法

我们团队在具体实现上面参照了IBM奥斯丁团队的[Bluepic项目](https://github.com/IBM/BluePic)，这个项目曾经在WWDC演讲过[Going Server-side with Swift Open Source](https://developer.apple.com/videos/play/wwdc2016/415/)。 其中也介绍了Swift在server side的可能性，值得一看哦。

# 基本架构
我们受到React组件思维的影响，对Bluepic的架构做了调整，用组件来组织不同的UI item，这一点也确实省了我们切换ViewController，ViewModel和View文件夹的时间。 而且我们团队在Sprint中分任务的时候也是根据组件负责来分的，这也让团队成员更专注于他们的任务

![image](https://image.ibb.co/enH8c5/Screen_Shot_2017_08_01_at_11_59_22_PM.png)

文件               | 解释
----------------------------|-----------------------------
AppDelegate.swift | 处理App生命周期
TestingAppDelegate.swift | 处理测试环境下App的生命周期
Localizable.strings | 多语言支持
Assets.xcassets/  | 图片资源
Components/ | UI组件，每个组件都由一个View Controller, View Model和Xib view构成
Configurations/ | 存放info.plist等配置文件
DataManagers/ | 本地数据库,网络相关
Extensions/ | 对Swift原生库的扩展方法
Models/ | 数据结构定义
Storyboards/ | 存放不同环境下的Storyboards
Utilities/ | 工具类文件夹

# 基本实现

![image](https://image.ibb.co/c6bjH5/Screen_Shot_2017_08_02_at_12_16_02_AM.png)

以Category Component为例子，他有自己的ViewController，ViewModel和Xib view。 这里Xib就不过多解释了，里面有autoLayout和基本的UI组件。

ViewController的实现如下,其中的CategoryViewModelNotification是View Model给View Controller传信息的消息约定. 在ViewController中会有一个ViewModel的reference，以及一个用于初始化ViewModel的notifyVC方法

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

ViewModel的实现如下,ViewModel会直接与Data Manager联系来更新当前这个Component的状态或者数据，然后成功后会通过初始化时ViewController传来的notifyVC来通知View Controller更新UI

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

以上这种模式是BluePic里所采用的，使用名为notifyVC的@escaping函数来作为绑定媒介。在初始化的时候把定义在ViewController的notifyVC绑定到ViewModel里面，当ViewModel中的数据发生变化时，用这个绑定好的函数来进行对ViewController的通知。

但是这种方法，由于在绑定方法的时候，会将这个notifyVC函数当成参数传入。由于不像Javascript，Swift中没有函数对象，这里的函数会被转换为closure。 而在这个closure中又保持了一个对于ViewController self的strong reference,这里就会产生循环依赖关系。如下图

![image](https://preview.ibb.co/kz1Tc5/Screen_Shot_2017_08_02_at_11_38_41_PM.png)

这样每一个Component的VC，VM，View的组合都是一个个闭合的三角循环依赖关系。由于iOS没有垃圾回收机制而是使用的ARC，这种循环引用会导致每次初始化一个Component都会导致一次内存泄漏，这在我团队的项目中很严重，每次切换语言重新渲染的时候都会有50MB左右的内存泄漏。

# 解决方法

我们要解决这个问题只需要打破三角关系中的一环，由于外部初始化component会从ViewController开始，销毁也一般会调用VC的removeFromParentVC方法。所以我们选择让他们的关系变成如下这样（虚线代表弱引用）

![image](https://preview.ibb.co/h3RKH5/Screen_Shot_2017_08_02_at_11_47_18_PM.png)

具体解决方法有两种

## 使用unowned或者weak关键字修饰notifyVC里的self，强制closure使用弱引用

在Viewcontroller中新加一个变量

```swift
  lazy var weakNotifyVC: (_ viewModelNotification: ProductListingPageViewModelNotification) -> Void ={                         [unowned self] in
         return self.notifyVC
  }()
```

在绑定的时候把这个weakNotifyVC传进ViewModel来做初始化，这样直接就把closure里的self reference变成weak类型，也就解决了这个问题


## 使用delegate模式，把一个变量传给ViewModel来初始化而不是传一个函数进去

先定义一个专门用来做VC和VM绑定的protocol

```swift
protocol NotifyVCDelegate: class {
    func notifyVC(notification: Any)
}
```

然后ViewController实现这个protocol

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

并且修改ViewModel的初始化函数,并且在ViewModel中介入一个notifyVCDelegate，这里的notifyVC函数复用了之前的实现，就不载多做描述了

```swift

   weak var notifyVCDelegate: NotifyVCDelegate!
   init(notifyVC : NotifyVCProtocol) {
       super.init()
       self.notifyVCDelegate = notifyVC
   }

```

并且改变ViewController中对ViewModel的初始化代码

```swift
self.viewModel = CategoryViewModel(notifyVC: self)
```

由于self是一个object，Swift会按引用传值，在ViewModel内部我们给他绑定了一个weak的引用，这样的话也就解决了之前closure到ViewController的强引用问题

# 小结

由于没有现成的绑定机制，所以MVVM在iOS中的实现要稍微复杂些。Swift中函数是一等公民，可以把函数当成参数传递到另一个函数中，但是他会变成一个closure，会保持对外部的引用关系，Closure相关的内存泄漏也是一个[iOS的经典问题](https://medium.com/@streem/understanding-memory-leaks-in-closures-48207214cba)。Swift个人认为在不同的类的交互之间更适合于面向协议的编程方式，这种方式也更符合ARC的工作机制，更方便也更直观计算每个对象的reference数目。所以我个人比较推荐第二种解决方案。第一种解决方案更像是OC留下的一个语法糖，这种非Swift风格的代码也很有可能在之后的Swift更新中被重构掉。
