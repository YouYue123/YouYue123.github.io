---
layout: post
section-type: post
title: iOS MVVM 和潜在的内存泄漏问题 in Swift 3
category: tech
tags: [ 'MVVM', 'iOS', 'Memory leak']
---

## 背景

最近在做一个iOS项目，客户要求交付的时候测试覆盖率，可维护性。这让我想到了之前开发的时候View Controller非常难测试，而且到了项目后期庞大的数千行代码的View Controller非常难维护，即使是用了不同的extension来分割代码，由于基本上90%的逻辑都会写在VC中，导致代码非常混乱。 可测试和维护性有很多解决方案，[VIPER](https://objccn.io/issue-13-5/)，借鉴于Redux的[单向数据流动函数式](https://onevcat.com/2017/07/state-based-viewcontroller/)，MVVM等等。从对已有模式的兼容性和维护测试性的两个方面考虑，最终我们还是选择MVVM作为最后选择。

## MVC vs MVVM in iOS

# MVC简介

由于iOS项目创建时会默认使用Model-View-Controller的标准模式，开发者一般都会将他用作自己的默认开发模式。我们来看一张经典的MVC的架构图。

![image](https://i.stack.imgur.com/BYaCl.png)

在MVC中，Model代表数据，View代表用户界面，Controller充当二者之间的媒介。在实际应用中，View和ViewController会经常深度耦合在一起，以至于在Xcode中当你选择新建一个COCOAPods类时，他会询问你是否要新建一个与之相对应的xib view，如下图

![image](https://preview.ibb.co/bVGJc5/Screen_Shot_2017_08_02_at_12_47_37_AM.png)

很多人会称MVC为 Massive View Controller, 因为在实现一个App的过程中很多开发者会把网络，数据，状态的处理方法都写在ViewController里，而造成厚VC薄Model的情况。也正是由于这种厚VC的情况，会让Unit test非常难写，因为VC做了太多的事情，混杂了对UI组件的处理，对View状态的维护和对数据的处理。但是也不能说这是一种错误，因为VC本来就是View和Model之间的胶水代码。时间长了这里就变成了意大利面代码，永远没人想知道里面发生了什么。

# MVVM简介

MVVM本质上是一种MVC的变种.，相当于把以前的VC拆分成View Controller和View Model两部分。 View Controller只关心View的信息比如IB Outlet，view的生命周期等等，View Model只关心现在的数据和状态信息。

这样的改变会让代码很好被测试，利用Unit test很容易测试View Model的数据和状态信息，而Xcode的Automatic UI testing又可以很好的帮我们处理VC这一层的测试情况。而且由于我们只是分离了View Controller为两层，这不会对我们现有的代码有太多的侵入。如果想分离一些逻辑代码给其他团队复用的时候，这种方式很方便提取和移植代码。

# 总结

总结一下MVVM相对MVC的三大优势:
1. 可测试性的提高
2. 方便代码的移植
3. 现有MVC的兼容性很好

当然世界上没有绝对完美的东西，MVVM也有他自己的缺点：
1. 学习成本高，尤其是团队里有新成员加入的时候，会比较难上手
2. MVVM需要绑定机制来触发VM到VC的响应，iOS没有很好的绑定机制可以用，这也造成了我现在团队里的内存泄漏问题
3. 数据和网络相关的代码会使得VM这层变得很臃肿，大量的Async代码会让程序难维护，所以我们采用了DataManager layer和Async Await in Swift来解决这个问题


## IBM团队MVVM在iOS的实现方法

我们团队在具体实现上面参照了IBM奥斯丁团队的[Bluepic项目](https://github.com/IBM/BluePic)，这个项目曾经在WWDC演讲过[Going Server-side with Swift Open Source](https://developer.apple.com/videos/play/wwdc2016/415/)。 其中也介绍了Swift在server side的可能性，值得一看哦。

# 基本架构
我们受到React组件思维的影响，对Bluepic的架构做了调整，用组件来组织不同的UI item，这一点也确实省了我们切换VC，VM，View folder的时间。 而且我们团队在sprint分任务的时候也是根据组件负责来分的，这也让团队成员更专注于他们的任务

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

以Category Component为例子，他有自己的VC，VM和Xib view。 这里Xib就不过多解释了，里面有autoLayout和基本的UI信息。

VC的实现如下,其中的CategoryViewModelNotification是View Model给View Controller传信息的interface. 在VC中会有一个viewModel的reference，以及一个用于初始化ViewModel的notifyVC方法

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

VM的实现如下,VM会直接与Data Manager联系来更新当前这个component的状态或者数据，然后成功后会通过初始化时ViewController传来的notifyVC来通知View Controller更新UI

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

但是由于我们


# 解决方法
