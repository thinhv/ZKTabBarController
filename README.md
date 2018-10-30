# ZKTabBarController

[![CI Status](https://img.shields.io/travis/ducthinh2410/ZKTabBarController.svg?style=flat)](https://travis-ci.org/ducthinh2410/ZKTabBarController)
[![Version](https://img.shields.io/cocoapods/v/ZKTabBarController.svg?style=flat)](https://cocoapods.org/pods/ZKTabBarController)
[![License](https://img.shields.io/cocoapods/l/ZKTabBarController.svg?style=flat)](https://cocoapods.org/pods/ZKTabBarController)
[![Platform](https://img.shields.io/cocoapods/p/ZKTabBarController.svg?style=flat)](https://cocoapods.org/pods/ZKTabBarController)

![ZeroKeyboard](zerokeyboard.png)

![](demo-1.gif)
![](demo-2.gif)
![](demo-3.gif)
![](demo-4.gif)

NOTE: This pod is being reviewed and not released to CocoaPod yet.

ZKTabBarController is an lightweight and easy-to-use UI component which enhances the one-hand usage of mobile phones when navigating around. The ZKTabBarController container view container was built with the idea of using the combination of UITabBarController and UINavigationController in mind.

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)

To run the example project, clone the repo, and run `pod install` from the Example directory first.

## Features
- [x] Similar to UITabBarController which contains multiple
- [x] Expandable tab bar
- [x] Dismissable tab bar
- [x] Easy-to-reach "UIBarButtonItem" items placed on the bottom (which is call ZKTabBarButtonItem)
- [x] Bottom ZKTabBarButtonItem can contain either a single action or a list of actions
- [x] Easy to customize tab bar / bar button items / layout...
- [x] Easy to customize the transition animation when switch between tabs(view controllers)
- [ ] Orientation-changes support

## Requirements

- iOS 9.0+
- Swift 4

## Installation

ZKTabBarController is available through [CocoaPods](https://cocoapods.org). To install
it, simply add the following line to your Podfile:

```ruby
pod 'ZKTabBarController'
```

## Usage

### Basics
Similar to the usage of `UITabBarController`, you would want to subclass `ZKTabBarController` and make it as the `rootViewController` of the `window`.

```swift
import ZKTabBarController

class MainViewController: ZKTabBarController {
...
}
```

### Setup child view controllers

You can easily setup the child view controllers by assigning an array of `UIViewController` to `viewControllers` property.

```swift
class MainViewController: ZKTabBarController {

override func viewDidLoad() {
super.viewDidLoad()

let firstViewController = UIViewController()
let secondViewController = UIViewController()
let thirdViewController = UINavigationController()
let fourthViewController = UINavigationController()

viewControllers = [firstViewController, secondViewController, thirdViewController, fourthViewController]
}
}
```

### ZKTabBarChildController

The child view controller should conform to `ZKTabBarChildController` protocol in order for the ZKTabBarController object to display its tab.

`ZKTabBarChildController` protocol only requires a single property of `ZKTabBarChildItem` type which holds the `title` and `image` for the tab.

```swift
class FirstViewController: UIViewController, ZKTabBarChildController {
var item: ZKTabBarChildItem? = ZKTabBarChildItem(title: "First", image: UIImage(named: "first_icn"))
}
```

### Child view controller 's properties

- `zkTabBarController`: references to the parent view controller, which is an instance of `ZKTabBarViewController`. Note: if the child view controller is a `UINavigationController`, its child view controllers could also reference to this parent `ZKTabBarViewController` object.
- `leftTabBarButtonItem` and `rightTabBarButtonItem`: as its name imply, these `ZKTabBarButtonItem` buttons are placed on the left and right of the tab bar respectively. The left and right bar button item of the active child view controller(which is being presented) are displayed. No thing will be displayed if it is `nil`. Note: You should setup these property after the view controller has been given to the parent `ZKTabBarController` object(in other words the ZKTabBarController's viewControllers property is setup). Ideally, you should create set this up inside `viewDidLoad()` method. 
```swift
...
lazy var lefItem: ZKTabBarButtonItem = {
let item = ZKTabBarButtonItem()
item.setImage(UIImage.appShortcutIcon(color: color), for: .normal)
item.setTitle("SHORTCUTS", for: .normal)
item.setTitleColor(color, for: .normal)
item.titleLabel?.font = UserFonts.caption2()
item.itemSize = CGSize(width: 120, height: 60)
item.edgeInsets = UIEdgeInsets(top: -2.0, left: 0.0, bottom: -2.0, right: 0.0)

return item
}()

override func viewDidLoad() {
super.viewDidLoad()

leftTabBarButtonItem = lefItem
}
...
```
- `leftSelectionMenuView` and `rightSelectionMenuView`: when the left or right tab bar button item is tapped the tab bar expands to show display the view which contains a list of other actions or trigger the action right away. These ZKTabBarSelectionMenuView instances are available once child view controller is attached to parent, which is similar to the above bar button items. More at [ZKTabBarSelectionMenuView](#ZKTabBarSelectionMenuView).

### Customize the transition

In order to customize the transition animation when switch between child view controllers, you need to have an transitioning delegate which conforms to `ZKTabBarControllerTransitioningDelegate`.

The only method required in `ZKTabBarControllerTransitioningDelegate` conformance returns an animator which conforms `ZKTabBarControllerContextTransitioning` protocol. You have to create this animator object and return it from the delegate method.

```swift
func animationController(forTransitionFromViewController fromViewController: UIViewController, toViewController: UIViewController) -> ZKTabBarControllerAnimatedTransitioning {
return MyAnimator()
}
```

```swift
class MyAnimator: NSObject, ZKTabBarControllerAnimatedTransitioning {

func animationOptions() -> UIViewAnimationOptions {
return .curveEaseIn
}

func duration() -> TimeInterval {
return 0.6
}

func animateTransition(with context: ZKTabBarControllerContextTransitioning) {
let fromViewController = context.fromViewController
let toViewController = context.toViewController
let containerView = context.containerView

let fromViewFromFrame = fromViewController.view.frame
let fromViewToFrame = CGRect(x: fromViewFromFrame.origin.x,
y: fromViewFromFrame.origin.y + 400,
width: fromViewFromFrame.size.width,
height: fromViewFromFrame.size.height)

let toViewFromFrame = fromViewToFrame
let toViewToFrame = containerView.bounds

toViewController.view.alpha = 0.0
toViewController.view.frame = toViewFromFrame

containerView.addSubview(toViewController.view)

UIView.animateKeyframes(
withDuration: duration(),
delay: 0.0,
options: UIViewKeyframeAnimationOptions.calculationModeLinear, animations: {
...
},
completion: { _ in
context.finishTransition()
})
}
}
```

Note: You have to call `finishTransition()` on the context when the animation finishes.

### Other customization

- Hide the  tab bar when the child navigation controller push a new view controller onto its stack. Override `shouldHideTabBarWhenPush()` method and return `true`
```swift
override func shouldHideTabBarWhenPush() -> Bool {
return true
}
```

- Hide the back button when push. In the view controller to which your child navigation controller going to push override `shouldHideBackButtonItem` and return `true`

```swift
public func shouldHideBackButtonItem() -> Bool {
return false
}
```

## Author

thinh.vo@zerokeyboard.com, ducthinh2410@gmail.com

## License

ZKTabBarController is available under the MIT license. See the LICENSE file for more info.
