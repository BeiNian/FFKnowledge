# FFKnowledge
#### 1、 改变view的frame，layer的frame是否会变化？改变layer.frame，view的frame是否会变化？请问原因是什么？
答：都改变了。 每个UIView内部都有一个CALayer在提供内容的绘制和显示，并且View的尺寸都是由Layer所提供。一个Layer的frame是由anchorPoint、position、bounds、transform等共同决定的。而一个View的frame只是简单的返回Layer的frame。同样的View的center和bounds也是返回Layer的一些属性。View是Layer的代理。
* 在调用[UIView initWithFrame：]方法时候。调用了私有方法[UIView  _createLayerWithFrame:]去创建CALayer。
* 在常见View的时候，Layer和View的相关方法调用顺序：
```
[UIView _createLayerWithFrame]
[Layer setBounds:bounds]
[UIView setFrame：Frame]
[Layer setFrame:frame]
[Layer setPosition:position]
[Layer setBounds:bounds]
```
* 在修改View的bounds时，调用了
```
[View setBounds：bounds]
[Layer setBounds：bounds]
```
* 在修改Layer的bounds时，只调用了
```
[Layer setBounds：bounds]
```
* 在改变View的bounds时Layer的方法会调用，在改变Layer的bounds时View的方法不会调用。
* 重写方法
```
[UIView drawRect:rect]//UIView

[CALayer display]//CALayer
```
可以看到 UIView 是 CALayer 的CALayerDelegate，我猜测是在代理方法内部[UIView(CALayerDelegate) drawLayer:inContext]


#### 2、 NSDictionary底层的数据结构是什么，根据key找到value的时间复杂度是多少？
* NSDictionary（字典）是使用 hash表来实现key和value之间的映射和存储的， hash函数设计的好坏影响着数据的查找访问效率。数据在hash表中分布的越均匀，其访问效率越高。而在Objective-C中，通常都是利用NSString 来作为键值，其内部使用的hash函数也是通过使用 NSString对象作为键值来保证数据的各个节点在hash表中均匀分布。
* O(1)是理想的复杂度，代表着恒定的时间。


#### 3、 autoreleasepool的释放时机是什么，什么时候需要自己声明一个autoreleasepool ？
* @autoreleasepool是自动释放池，让我们更自由的管理内存。需要自己声明的场景应该是有大量临时变量产生时。避免内存峰值过高，及时的释放内存。


#### 4、什么事ARC？（ARC是为了解决什么技术诞生的？）
* ARC 自动引用计数。这是iOS 5中引用的内存管理机制。由编译器在合适的位置插入retain / release 方法。从而进行内存管理。减少了开发者工作量，能够大幅度提高程序的 流畅性 和 可预测性 。但是在 Core Foundation 框架中，让然需要手动管理内存。


#### 5、keywords 有什么区别: assign vs weak , __block vs __weak ？
* assign 和 weak 是用于在声明属性时，为属性指定内存管理的语句。
* assign 用于简单的赋值，不改变属性的引用计数。用于基本数据类型的修饰。
* weak 用于对象，弱引用，不改变对象的引用计数。当对象释放后置为nil，避免错误的内存访问。它可以用于避免两个强引用产生的循环引用导致无法释放的问题。
* __block 和 __weak 它们都是用于修饰变量的。
* __block 不管ARC 和 MRC 下都可以使用。声明的变量可以在block中改变变量的值。
* __weak 只能再ARC下使用，也只能修饰对象，不能修饰基本数据类型。同时，要避免block的巡管引用  __weak typedof(self)weakSelf = self ；
* 在MRC 中 __block 在block 中使用是不会 retain的。但在 RAC 中 __block 是会被retain 的。


#### 6、+(void)load ; +(void)initialize ；有什么用处？
* 如果父类和子类都调用，会优先调用父类再调用子类。
* +(void)load 在初始化加载时调用。只会被调用一次。
* +(void)initialize 会在当前类第一次主动使用时调用。有可能被调用过多次。如果子类没有实现，父类实现了，子类中会调用父类的+(void)initialize 方法。

#### 7、请简述Responder Chain ？
* 响应者对象：继承自UIResponder 的对象，称为响应者对象。
* 响应者链：由多个响应者组成起来的链条，称为响应者链。（系统把包含这些点击事件的信息包装成UITouch和UIEvent形式的实例，然后找到当前运行的程序，逐级寻找能够响应这个事件的对象，直到没有响应者响应。这一寻找的过程，被称作事件的响应链）
* 在iOS 中，能够响应事件的对象都是继承自UIResponder。UIResponder提供了4个用户点击回调的方法，分别对应用户点击开始、移动、点击结束、取消点击，其中只有程序强制退出过着来电时，取消点击事件才会调用。
* 在捕获UIEvent 后，从AppDelegate-->UIAppcation-->UIWindow-->UIViewController-->UIView-->subView 进行查找。然后从 subView 开始尝试响应这个事件，并传递给 nextResponder，如果父节点不能响应则不继续往下传递。
* 在查找过程中，通过pointInside 判断是否在本视图内，通过htiTest 返回响应事件的对象。

#### 8、 http的post和get啥区别？
* GET 用于向服务器请求数据，POST 用于提交数据。
* GET 请求的数据会依附在URL之后，POST 请求则放在请求体里。
* GET请求提交数据有限制（不超过2KB），POST没有限制。
* POST的安全性要高于GET的安全性。

#### 9、KVO 原理
* KVO 键值监听，基于runtime实现的。
* 键值观察通知依赖于NSObject 的两个方法: willChangeValueForKey: 和 didChangevlueForKey:；在一个被观察属性发生改变之前， willChangeValueForKey:一定会被调用，这就 会记录旧的值。而当改变发生后，didChangeValueForKey:会被调用，继而 observeValueForKey:ofObject:change:context: 也会被调用。
* 在某个类的属性第一次被观察时，系统在运行期间动态的为该类添加一个派生类，并在派生类中重写被观察的属性的setter方法。在派生类的重写方法中真正的实现通知机制。
* 如果原类为Person，那么生成的派生类名为NSKVONotifying_Person
* 每一个类对象都有一个isa指针指向当前类，当一个类的对象第一呗观察时，系统会偷偷的将isa指针指向动态生成的派生类，从而在给被监听属性赋值时执行的是派生类的setter方法。
* 补充：KVO的这套实现机制中苹果还偷偷重写了class方法，让我们误认为还是使用的当前类，从而达到隐藏生成的派生类


#### 10、Runloop
* Runloop有事件时会去处理，没有事件时会休眠。
* Runloop 和线程是绑定在一起的。每个线程（包括主线程）都有一个对应的Runloop 对象。我们不能自己创建Runloop 对象，但是可以获取到系统提供的Runloop 对象。
* 主线程的Runloop 会在程序启动的时候完成启动，其他线程的 Runloop 默认并不会开启，需要我们手动启动。














