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





















