> iOS - Notification 通知

本文目录

* 1、Notification
* 2、系统发送 Notification 的使用
* 3、自定义发送 Notification 的使用
* 4、异步发送 Notification 的使用
* 5、系统通知的使用

> **1、Notification**

* 通知中心实际上是在程序内部提供了消息广播的一种机制，它允许我们在低程度耦合的情况下，满足控制器与一个任意的对象进行通信的目的。每一个 iOS 程序（即每一个进程）都有一个自己的通知中心，即 NSNotificationCenter 对象，该对象采用单例设计模式，可以通过类方法 defaultCenter 获得当前进程唯一的通知中心对象。一个 NSNotificationCenter 可以有许多的通知消息 NSNotification，对于每一个 NSNotification 可以有很多的观察者 Observer 来接收通知。NSNotificationCenter 是 iOS 中通知中心的灵魂，由该类实现了观察者模式，并给开发者提供了诸如注册、删除观察者的接口。

* 通知中心以同步的方法将消息转发到所有的观察者中，换言之 NSNotificationCenter 在发送消息后，会一直等待被调用的方法执行完毕，然后返回控制权到主函数中，再接着执行后面的功能，即这是一个同步阻塞的操作。如果我们需要异步的处理消息，直接返回控制权，则应该使用通知队列 NSNotificationQueue，在子线程中将通知加入到通知队列中，在多线程程序中，通知会被分发到每一个发送消息的线程中，这可能与观察者注册时所在的线程已经不是同一个线程。 
* 任何一个对象都可以向通知中心发布通知（NSNotification），描述自己在做什么。其他感兴趣的对象（Observer）可以申请在某个特定通知发布时（或在某个特定的对象发布通知时）收到这个通知。 
![](https://github.com/CalvinCheungCoder/Blog/blob/master/Objective-C/Notification/1.png)

* 使用 [NSNotificationCenter defaultCenter] 发送的通知无论是在主线程还是子线程中被注册，观察者注册的选择器方法都会在主线程中被执行。执行顺序为：main runloop -> 发送通知 －> 观察者选 择器方法（按照观察者注册的顺序执行）-> 通知发送者方法中其它的操作 -> main runloop。

* 在子线程中使用 [NSNotificationQueue defaultQueue] 将通知加入到通知队列中，观察者选择器方法就会在子线程中被执行。子线程执行顺序为：发送通知 -> 观察者选择器方法（按照观察者注册的顺序同 步执行）-> 通知发送者方法中其它的操作。 
* 如果要在同一台机器上进行进程间的通信，需要使用 NSDistributedNOtificationCenter。

#### 优势：
* 1、不需要编写多少代码，实现比较简单；

* 2、对于一个发出的通知，多个对象能够做出反应，即 1 对多的方式实现简单；

* 3、controller 能够传递 context 对象（dictionary），context 对象携带了关于发送通知的自定义的信息。

#### 缺点：
* 1、在编译期不会检查通知是否能够被观察者正确的处理；

* 2、在释放注册的对象时，需要在通知中心取消注册；

* 3、在调试的时候应用的工作以及控制过程难跟踪；

* 4、需要第三方来管理 controller 与观察者对象之间的联系；

* 5、controller 和观察者需要提前知道通知名称、UserInfo dictionary keys。如果这些没有在工作区间定义，那么会出现不同步的情况；

* 6、通知发出后，controller 不能从观察者获得任何的反馈信息。
	
#### 目的：
* 降低两个子系统之间的偶合度。
	
#### 方式：
* 一个对象发送通知给通知中心，通知中心以广播的形式通知所有的监听者。

	
#### 通知和代理的区别：
##### 共同点：
* 利用通知和代理都能完成对象之间的通信（比如 A 对象告诉 D 对象发生了什么事情, A 对象传递数据给 D 对象）
	
##### 不同点：

* 代理 : 1 个对象只能告诉另 1 个对象发生了什么事情。

* 通知 :

	* 1 个对象能告诉 N 个对象发生了什么事情, 1 个对象能得知 N 个对象发生了什么事情。
	
	* 任何数量的对象都可以接收同一个消息，而不仅限定于委托对象。
	
	* 通知系统不接受返回值。
	
	* 对象不需要预先定义协议方法，就可以接收来自通知中心的消息。
发布通知的对象只负责发布通知，不需要关心观察者是否存在。

##### 通知中心是同步的，还是异步的 ？
* 同步的。当发生通知时，通知中心广播，有可能有多个监听者，设计上使用同步的方式，能够保证所有的监听者都对通知作出响应，不会产生遗漏。

> **2、系统发送 Notification 的使用**

* 系统发送 Notification，用户不需要手动发送通知，设置的事件触发时，系统自动发送通知。

* 通知中心不会保留(retain)监听器对象，在通知中心注册过的对象，必须在该对象释放前取消注册。否则，当相应的通知再次出现时，通知中心仍然会向该监听器发送消息。因为相应的监听器对象已经被释放了，所以可能会导致应用崩溃。一般在监听器销毁之前取消注册（如在监听器中加入下列代码）： 
``` Objective-C
- (void)dealloc {
        // [super dealloc];  // 非 ARC 中需要调用此句
        [[NSNotificationCenter defaultCenter] removeObserver:self];
    }
```

* 在注册、移除通知时，通知名称标示（aName）使用系统定义的标示。
* 注册通知（观察者）
	* Objective-C

``` Objective-C
[[NSNotificationCenter defaultCenter] addObserver:self 
                                             selector:@selector(playFinished) 
                                                 name:AVPlayerItemDidPlayToEndTimeNotification 
                                               object:nil];
```

* Swift

``` Swift
NSNotificationCenter.defaultCenter().addObserver(self, 
                                           selector: #selector(ViewController.playFinished), 
                                               name:AVPlayerItemDidPlayToEndTimeNotification, 
                                             object: nil)
```

* 移除通知（观察者）
	* Objective-C 

``` Objective-C
[[NSNotificationCenter defaultCenter] removeObserver:self 
                                                    name:AVPlayerItemDidPlayToEndTimeNotification 
                                                  object:nil];
```

* Swift

``` Swift
NSNotificationCenter.defaultCenter().removeObserver(self, 
                                                   name:AVPlayerItemDidPlayToEndTimeNotification, 
                                                 object:nil)
```

>**3、自定义发送 Notification 的使用**

* 使用 [NSNotificationCenter defaultCenter] 发送的通知无论是在主线程还是子线程中被注册，观察者注册的选择器方法都会在主线程中被执行。

* 执行顺序：main runloop -> 发送通知 －> 观察者选择器方法（按照观察者注册的顺序执行）-> 通知发送者方法中其它的操作 －> main runloop

* 通知（消息）的创建 

``` Objective-C
+ (instancetype)notificationWithName:(NSString *)aName object:(nullable id)anObject;

+ (instancetype)notificationWithName:(NSString *)aName object:(nullable id)anObject 
                                                         userInfo:(nullable NSDictionary *)aUserInfo;

    public convenience init(name aName: String, object anObject: AnyObject?)
    public init(name: String, object: AnyObject?, userInfo: [NSObject : AnyObject]?)

    参数说明：
        aName    ：通知名称
        anObject ：传递给观察者的任意对象，通知发布者(是谁要发布通知)
        aUserInfo：传递的消息内容，自定义字典，可以传递更多附加信息，一些额外的信息(通知发布者传递给通知接收者的信息内容)
```

* Objective-C


```Objective-C
// 不带消息内容
    NSNotification *notification1 = [NSNotification notificationWithName:@"notification1" 
                                                                  object:self];                                         

// 带消息内容
	NSNotification *notification2 = [NSNotification notificationWithName:@"notification2" 
                                                                  object:self 
                                                                userInfo:@{@"name":_name, @"age":_age}]; 
```

* Swift

```Swift
// 不带消息内容
    let notification1 = NSNotification(name: "notification1", 
                                     object: self)                                                                      
// 带消息内容
let notification2 = NSNotification(name: "notification2", 
                                     object: self, 
                                   userInfo: ["name":name, "age":age])  
```

* 发送通知


```Objective-C
- (void)postNotification:(NSNotification *)notification;

- (void)postNotificationName:(NSString *)aName object:(nullable id)anObject;

- (void)postNotificationName:(NSString *)aName object:(nullable id)anObject 
                                                 userInfo:(nullable NSDictionary *)aUserInfo;

    public func postNotification(notification: NSNotification)
    public func postNotificationName(aName: String, object anObject: AnyObject?)
    public func postNotificationName(aName: String, object anObject: AnyObject?, 
                                                 userInfo aUserInfo: [NSObject : AnyObject]?)

    参数说明：
        notification：发送的通知（消息）

        aName       ：通知名称
        anObject    ：传递给观察者的任意对象，通知发布者
        aUserInfo   ：传递的消息内容，自定义字典，可以传递更多附加信息
```

* Objective-C

    
```Objective-C
// 发送创建好的消息
    [[NSNotificationCenter defaultCenter] postNotification:notification1];

// 直接发送消息，不带消息内容
[[NSNotificationCenter defaultCenter] postNotificationName:@"notification3" 
                                                        object:self];                                           
    
// 直接发送消息，带消息内容
[[NSNotificationCenter defaultCenter] postNotificationName:@"notification4" 
                                                        object:self 
                                                      userInfo:@{@"name":_name, @"age":_age}];
```

* Swift


```Swift
// 发送创建好的消息
    NSNotificationCenter.defaultCenter().postNotification(notification1)

// 直接发送消息，不带消息内容
    NSNotificationCenter.defaultCenter().postNotificationName("notification3", 
                                                       object: self)                                            

// 直接发送消息，带消息内容
    NSNotificationCenter.defaultCenter().postNotificationName("notification4", 
                                                       object: self, 
                                                     userInfo: ["name":name, "age":age])
```

* 注册通知（观察者）


```Objective-C
- (void)addObserver:(id)observer 
               selector:(SEL)aSelector 
                   name:(nullable NSString *)aName 
                 object:(nullable id)anObject;

 public func addObserver(observer: AnyObject, 
                  selector aSelector: Selector, 
                          name aName: String?, 
                     object anObject: AnyObject?)

    参数说明：
        observer ：观察者，即谁要接收这个通知;
        aSelector：收到通知后调用何种方法，即回调函数，并且把通知对象当做参数传入;
        aName     ：通知的名字，也是通知的唯一标示，编译器就通过这个找到通知的。
                   为 nil 时，表示注册所有通知，那么无论通知的名称是什么，监听器都能收到这个通知；
        anObject ：通知发送者，为 nil 时，表示监听所有发送者的通知。如果 anObject 和 aName 都为 nil，监听器都收到所有的通知。


- (id)addObserverForName:(NSString *)name 
                      object:(id)obj 
                       queue:(NSOperationQueue *)queue 
                  usingBlock:(void (^)(NSNotification *note))block;

    参数说明：
        name ：通知的名称
        obj  ：通知发布者
        queue：决定了 block 在哪个操作队列中执行，如果传 nil，默认在当前操作队列中同步执行
        block：收到对应的通知时，会回调这个 block
```

* Objective-C


```Objective-C
[[NSNotificationCenter defaultCenter] addObserver:self 
                                             selector:@selector(notification1Sel) 
                                                 name:@"notification1" 
                                               object:nil];

[[NSNotificationCenter defaultCenter] addObserver:self 
                                             selector:@selector(notification2Sel:) 
                                                 name:@"notification2" 
                                               object:nil];

    // 通知触发方法，通知无内容
    - (void)notification1Sel {

    }

    // 通知触发方法，通知有内容
    - (void)notification2Sel:(NSNotification *)notification {

        // 接收用户消息内容
        NSDictionary *userInfo = notification.userInfo;
    }
```

* Swift

    
```Swift
NSNotificationCenter.defaultCenter().addObserver(self, 
                                            selector: #selector(ViewController.notification1Sel), 
                                                name: "notification1", 
                                              object: nil)


NSNotificationCenter.defaultCenter().addObserver(self, 
                                            selector: #selector(ViewController.notification2Sel(_:)), 
                                                name: "notification2", 
                                              object: nil)

    // 通知触发方法，通知无内容
    func notification1Sel() {

    }

    // 通知触发方法，通知有内容
    func notification2Sel(notification:NSNotification) {

        // 接收用户消息内容
        let userInfo = notification.userInfo
    }
```


* 移除通知（观察者）


```Objective-C
- (void)removeObserver:(id)observer;

- (void)removeObserver:(id)observer name:(nullable NSString *)aName object:(nullable id)anObject;

    public func removeObserver(observer: AnyObject)
    public func removeObserver(observer: AnyObject, name aName: String?, object anObject: AnyObject?)

    参数说明：
        observer：观察者，即在什么地方接收通知;
        aName   ：通知的名字，也是通知的唯一标示，编译器就通过这个找到通知的。
        anObject：通知发送者，为 nil 时，表示移除满足条件的所有发送者的通知。
```

* Objective-C

	
```Objective-C
// 移除此观察者的所有通知
[[NSNotificationCenter defaultCenter] removeObserver:self];

// 移除指定名字的通知
[[NSNotificationCenter defaultCenter] removeObserver:self name:@"notification1" object:nil];	
```

* Swift


```Swift
    // 移除此观察者的所有通知
    NSNotificationCenter.defaultCenter().removeObserver(self)

    // 移除指定名字的通知
    NSNotificationCenter.defaultCenter().removeObserver(self, name:"notification1", object:nil)
```

> **4、异步发送 Notification 的使用**

* 在子线程中使用 [NSNotificationQueue defaultQueue] 将通知加入到通知队列中，观察者选择器方法就会在子线程中被执行。


```
| ->  ->  ->  ->  ->  ->  ->  -> main runloop ->  ->  ->  ->  ->  ->  ->  ->  ->  ->   |
    执行顺序：main runloop -> |                                                                                      | -> main runloop
                             | -> 发送通知 -> 观察者选择器方法（按照观察者注册的顺序同步执行）-> 通知发送者方法中其它的操作 |
```

* 发送异步通知

```Objective-C
- (void)enqueueNotification:(NSNotification *)notification 
                   postingStyle:(NSPostingStyle)postingStyle;

- (void)enqueueNotification:(NSNotification *)notification 
                   postingStyle:(NSPostingStyle)postingStyle 
                   coalesceMask:(NSNotificationCoalescing)coalesceMask 
                       forModes:(nullable NSArray<NSString *> *)modes;

    参数说明：
        notification：通知
        postingStyle：发布方式
        coalesceMask：合并方式
        modes       ：运行循环模式，nil 表示 NSDefaultRunLoopMode

        NSPostingStyle                              ：发布方式

            NSPostWhenIdle = 1,                     ：空闲时发布
            NSPostASAP = 2,                         ：尽快发布
            NSPostNow = 3                           ：立即发布

        NSNotificationCoalescing                    ：合并方式

            NSNotificationNoCoalescing = 0,         ：不合并
            NSNotificationCoalescingOnName = 1,     ：按名称合并
            NSNotificationCoalescingOnSender = 2    ：按发布者合并
```

* Objective-C
```Objective-C
	// 创建通知
    NSNotification *asyncNotification = [NSNotification notificationWithName:@"asyncNotification" object:self];

    dispatch_async(dispatch_get_global_queue(0, 0), ^{

        // 将通知添加到发送队列中，发送通知
        [[NSNotificationQueue defaultQueue] enqueueNotification:asyncNotification postingStyle:NSPostWhenIdle];
    });
```

* 移除异步通知

```Objective-C
- (void)dequeueNotificationsMatching:(NSNotification *)notification coalesceMask:(NSUInteger)coalesceMask;

    参数说明：
        notification：通知
        coalesceMask：合并方式
```

* Objective-C

```Objective-C
 	// 移除通知，不是立即发布的通知可以被移除
    [[NSNotificationQueue defaultQueue] dequeueNotificationsMatching:asyncNotification coalesceMask:0];
```

> **5、系统通知的使用**

#### 5.1 UIDevice 通知

* UIDevice 类提供了一个单例对象，它代表着设备，通过它可以获得一些设备相关的信息，比如电池电量值（batteryLevel）、电池状态（batteryState）、设备的类型（model，比如 iPod、iPhone 等）、设备的系统（systemVersion）。通过 [UIDevice currentDevice] 可以获取这个单例对象。

* UIDevice 对象会不间断地发布一些通知，下列是 UIDevice 对象所发布通知的名称常量： 

```Objective-C
    UIDeviceOrientationDidChangeNotification     // 设备旋转
    UIDeviceBatteryStateDidChangeNotification    // 电池状态改变
    UIDeviceBatteryLevelDidChangeNotification    // 电池电量改变
    UIDeviceProximityStateDidChangeNotification  // 近距离传感器（比如设备贴近了使用者的脸部）
```

#### 5.2 键盘通知

* 我们经常需要在键盘弹出或者隐藏的时候做一些特定的操作，因此需要监听键盘的状态。 
* 键盘状态改变的时候，系统会发出一些特定的通知： 

```Objective-C
    UIKeyboardWillShowNotification         // 键盘即将显示
    UIKeyboardDidShowNotification          // 键盘显示完毕
    UIKeyboardWillHideNotification         // 键盘即将隐藏
    UIKeyboardDidHideNotification          // 键盘隐藏完毕
    UIKeyboardWillChangeFrameNotification  // 键盘的位置尺寸即将发生改变
    UIKeyboardDidChangeFrameNotification   // 键盘的位置尺寸改变完毕
```

* 系统发出键盘通知时，会附带一下跟键盘有关的额外信息(字典)，字典常见的 key 如下:


```Objective-C
    UIKeyboardFrameBeginUserInfoKey         // 键盘刚开始的 frame
    UIKeyboardFrameEndUserInfoKey           // 键盘最终的 frame（动画执行完毕后）
    UIKeyboardAnimationDurationUserInfoKey  // 键盘动画的时间
    UIKeyboardAnimationCurveUserInfoKey     // 键盘动画的执行节奏（快慢）
```

[原文链接：QianChia 的博客](http://www.cnblogs.com/QianChia/p/5771055.html#_label0)

