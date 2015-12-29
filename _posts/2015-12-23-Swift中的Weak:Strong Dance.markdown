---
layout:     post
title:      "Swift中的Weak Strong Dance"
subtitle:   ""
date:       2015-12-23 12:00:00
author:     "DavidDay"
header-img: "img/post/2015-12-23-bg.jpg"
tags:
    - Swift
    - 循环引用
    - Weak Strong Dance
    - 翻译
---

# Swift中的Weak/Strong Dance

马上又要过年了，诶，再也不能像当初那样无耻地逗利是了(我们广东的方言讨红包的意思)

图1

![](http://daiweilai.github.io/img/post/2015-12-23-pic1.jpg)

图2

![](http://daiweilai.github.io/img/post/2015-12-23-pic2.jpg)

看来今年没利了

谁让哥已经工作了呢。 

公司今年的开发任务算是完结了，苹果又极不负(hǎo)责(yàng)任(de)地放圣诞不审核了，所以这半个月就该清闲下来了。

博主掐指一算，Swift已经养到2.1了，并且也开源了，这样看来Swift也够肥了，语法也绝逼不会再有大改动了，是该再次抓起来了。

为什么说再呢，其实在当初Swift Beta版本的时候，我们项目经理尝试了一下palyground后，一拍手，棒棒哒，"我们用Swift开发接下来的项目吧"，然后就是Swift1.0 … 2.0，每一次升级，每一次语法更迭，每一次XCode打开的那一刻都是满江红，那触目惊心的场面无数次让博主受尽折磨。最后庆幸的是，这个项目死了，哦耶！

好吧，重新捡起Swift吧，今天的这篇文章是一篇激情翻译大作。

好吧回到博文的主题中来，这次我们说说“Weak/Strong Dance”

在block中解决循环引用要追寻的[2011 WWDC Session #322](https://developer.apple.com/videos/play/wwdc2011-322/){:target="_blank"} 当时惊艳的代码是这样的：

``` objectivec
- (void)dealloc
{
  [[NSNotificationCenter defaultCenter] removeObserver:_observer];
}

- (void)loadView
{
  [super loadView];

  __weak TestViewController *wself = self;
  _observer = [[NSNotificationCenter defaultCenter] addObserverForName:@"testKey"
                                                                object:nil
                                                                 queue:nil
                                                            usingBlock:^(NSNotification *note) {
      TestViewController *sself = wself;
      [sself dismissModalViewControllerAnimated:YES];
  }];
}
```

或者可以看看[AFNetWorking](https://github.com/AFNetworking){:target="_blank"} 是这样用block的:

``` objectivec
__weak __typeof(self)weakSelf = self;
AFNetworkReachabilityStatusBlock callback = ^(AFNetworkReachabilityStatus status) {
__strong __typeof(weakSelf)strongSelf = weakSelf;
strongSelf.networkReachabilityStatus = status;
if (strongSelf.networkReachabilityStatusBlock) {
strongSelf.networkReachabilityStatusBlock(status);
}
};
```



我们都很熟悉Objective-C中的“weak/strong dance”，但是寂寞无聊的我突然就很想知道Swift语言中该怎么做呢？是否存在传说中的最佳实践呢？



好啦，翻译开始！

[原文](http://kelan.io/2015/the-weak-strong-dance-in-swift/){:target="_blank"}

首先，我们祭出一个在闭包中没有使用weak的引用导致的循环引用的例子

``` swift
class C {
    let name: String
    var block: (() -> Void)?

    init(name: String) {
        self.name = name
        block = {
            self.doSomething()
        }
    }
    deinit { print("Destroying \(name)") }
    func doSomething() { print("Doing something for \(name)") }
}

var c = C(name: "one")
print(c.name)
c = C(name: "two")
print(c.name)
```

输出

``` swift
one
two
```

这是一个巨基础又明显的循环引用的例子，self -> block ->self

所以，`deinit` 方法是绝逼不会被执行的，即使你把 `c` 重新指向`nil`或者其他的实例，`c`也不会被销毁，这就是顽固又调皮的循环引用了，尤其是当你把`c`指向`nil`之后，这个对象你就再也引用不了了，它就静静的躺在堆内存里面，遗世而独立，然后你就堆内存泄露了，然后你就淡淡的忧伤从下体传来 ~ ~没有然后了

其实Swift中闭包的参数列表([Capture List](http://www.russbishop.net/swift-capture-lists){:target="_blank"}) 已经能够很好的让你获取一个`weak self`来避免循环引用了，但这还达不到我们的要求，只有`weak`是构不成“weak/strong dance”滴。



## 使用闭包参数列表

``` swift
class C {
    let name: String
    var block: (() -> Void)?

    init(name: String) {
        self.name = name
        block = { [weak self] in  // <-- 这里做出一些改变
            self?.doSomething()
        }
    }
    deinit { print("Destroying \(name)") }
    func doSomething() { print("Doing something for \(name)") }
}

var c = C(name: "one")
print(c.name)
c = C(name: "two")
print(c.name)
```

输出

``` swift
one
Destroying one
two
```

这样就没有循环引用啦~



## 在闭包中使用`self`

使用 `[weak self]`有一个细节，就是`self`在闭包中会变成`Optional` 从上面的代码中`self?.doSomething()` 就可以看出来了。

但是如果你在这个闭包中狂轰乱炸的使用`self？` （多次使用`self？`），问题就来了，因为这个`self？`是一个弱引用的，那么你没法确定在这个闭包中所有的`self？`操作都能执行完毕，毕竟若引用的`self`可能随时都挂掉，然后怒举一个栗子：

图4

![](http://daiweilai.github.io/img/post/2015-12-23-pic4.jpg)

``` swift
class C {
    deinit { println("Destroying C") }
    func log(msg: String) { println(msg) }
    func doClosure() {
        dispatch_async(dispatch_get_global_queue(0, 0)) { [weak self] in
            self?.log("before sleep")
            usleep(500)
            self?.log("after sleep")
        }
    }
}

var c: C? = C()  // Optional, so we can set it to nil
c?.doClosure()

dispatch_async(dispatch_get_global_queue(0, 0)) {
    usleep(100)
    c = nil  // This will dealloc c
}

dispatch_main()
```

输出

``` swift
before sleep
Destroying C
```



> 小提示：当然一般来说在`dispatch_async()`中你不必担心会有循环引用，因为self并不会持有`dispatch_async()`的block，所以上述的代码中并不会真的导致循环引用，如果你的闭包并不是很注重结果的，那么`self`为`nil`闭包就不会再执行，这个还是挺有用的。



上述的代码中不会打印`after sleep`，因为`self？`在打印这句话之前已经挂掉了。

通常这种无根之源的bug可以把你整的半死。所以通常遇到这种闭包中多次试用`self？`的操作的时候，一般会把`self？`变为又粗又壮的`strong self`，（博主也是又粗又壮的，捂脸~~）这就是传说中的“weak/strong dance”，这个舞蹈，额，什么鬼，为什么把这个技术叫做dance啊，我觉得叫做`美队解禁奥义技`还不错，妇联里面的美国队长也是由`weak`变成`strong`的嘛~，好吧，扯太远，菊花都扯疼了，我们这是在技术翻译呢！要严肃！要尊重原作者！我们还是叫dance吧，有了这个dance之后呢，我们就能确保一旦闭包被执行，`self`就不会为`nil`。

但是，就像文章开头说的，对于在`Swift`的“weak/strong dance”中变回strong的这部分的最佳实践是什么我也不是很确定的。。。



## 获取强引用的一些想法

### 使用可选绑定`if let`

``` swift
func doClosure() {
    dispatch_async(dispatch_get_global_queue(0, 0)) { [weak self] in
        if let strongSelf = self {  // <-- 这里就是精髓了
            strongSelf.log("before sleep")
            usleep(500)
            strongSelf.log("after sleep")
        }
    }
}

// or in Swift 2, using `guard let`:
dispatch_async(dispatch_get_global_queue(0, 0)) { [weak self] in
    guard let strongSelf = self else { return }  // <-- 这里就是精髓了
    strongSelf.log("before sleep")
    usleep(500)
    strongSelf.log("after sleep")
}
```

输出

``` swift
before sleep
after sleep
Destroying C
```

优点：

* 很明显的看出整个操作的流程
* 在闭包中拿到了非可选的本地变量

缺点：

* 很不幸的是我们不能`if let self = self`，因为`self`是常量，不可变，这样的话我们就只能`if let strongSelf = self` 在闭包的作用域中都要使用丑陋的`strongSelf`了。
* 在swift的闭包中，如果你没有试用`strongSelf`而是使用了`self`，这样编译器会警告！因为这个时候`self`是可选的嘛，相比较OC中，就不会警告了。(这句话哥读了21遍，为什么觉得这个不是缺点呢)



### 使用`withExtendedLifetime`

在Swift的标准库中有一个函数：`withExtendedLifetime()`，感觉就像Apple这个金鱼佬故意诱导我们使用这个函数来实现“weak/strong dance”。

``` swift
/// Evaluate `f()` and return its result, ensuring that `x` is not
/// destroyed before f returns.
func withExtendedLifetime<T, Result>(x: T, @noescape _ f: () -> Result) -> Result
```

那就试试

``` swift
func doClosure() {
    dispatch_async(dispatch_get_global_queue(0, 0)) { [weak self] in
        withExtendedLifetime(self) {
            self!.log("before sleep")
            usleep(500)
            self!.log("after sleep")
        }
    }
}

```

优点：

* 闭包中不再需要使用丑陋的`strongSelf`了

缺点：

* `self`还是他妈可选的，调用方法什么的还是要！？，还是要解包，博主突然想起自己的一个技能：单手解，呵呵



### 自定义一个`withExtendedLifetime()`

这个方法是 [@jtbandes](https://twitter.com/jtbandes){:target="_blank"} 这哥们想的，大概会是这样：

``` swift
 extension Optional {
    func withExtendedLifetime(body: Wrapped -> Void) {
        if let strongSelf = self {
            body(strongSelf)
        }
    }
}

// Then:
func doClosure() {
    dispatch_async(dispatch_get_global_queue(0, 0)) { [weak self] () -> Void in
        self.withExtendedLifetime {
            $0.log("before sleep")
            usleep(500)
            $0.log("after sleep")
        }
        return
    }
}
```

优点：

* Follows naming conventions set by the standard library.(原文) 感觉没优点~

缺点：

* `strongSelf`变成了使用`$0` ，博主认为哦，还是很丑陋，并且可读性更差了
* In this case, I had to add some extra type info to the `dispatch_async()` closure. I’m not totally sure why. 不知道他说什么鬼~



翻译至此结束了



## 后记

关于Swift中 “Weak/Strong Dance”，中的Weak部分，大家可以参阅喵大的这篇文章 [内存管理,*weak* 和 *unowned*](https://www.baidu.com/link?url=4d2OoIpoCqnc98N8TJ4PiMR2qoqCe4qm-9LnTIUU4yzHwwVR6KaO1EOOSYNuAowX&wd=&eqid=fb7afdf00006201300000005567b8a3c){:target="_blank"} 。

用回了一个多礼拜的Swift真是感受颇多，虽然Xcode在写Swift还是像纯文本编辑器一样，但是我还是想说一句：Swift真™安全！想crash都难咯。

收笔，走人。