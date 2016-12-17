##### 1，通知（NSNotification）介绍
这里所说的通知不是指发给用户看的通知消息，而是系统内部进行消息传递的通知。要介绍通知之前，我们需要先了解什么是观察者模式。

观察者模式 （Observer）：指一个对象在状态变化的时候会通知另一个对象。参与者并不需要知道其他对象的具体是干什么的 。这是一种降低耦合度的设计。常见的使用方法是观察者注册监听，然后在状态改变的时候，所有观察者们都会收到通知。 
在 MVC 里，观察者模式意味着需要允许 Model 对象和 View 对象进行交流，而不能有直接的关联。

Cocoa 使用两种方式实现了观察者模式： 一个是 Key-Value Observing (KVO)，另一个便是本文要讲的Notification。

##### 2，系统通知的注册和响应
比如我们想要在用户按下设备的home键，程序进入后台时执行某些操作。一种办法是在AppDelegate.swift里的applicationDidEnterBackground方法里执行。
除此之外，由于程序进入后台会发送 UIApplicationDidEnterBackgroundNotification 的通知，我们可以事先注册个监听这个通知的“观察者”来处理。


```
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let notificationCenter = NSNotificationCenter.defaultCenter()
         
        let operationQueue = NSOperationQueue.mainQueue()
         
        let applicationDidEnterBackgroundObserver =
            notificationCenter.addObserverForName(UIApplicationDidEnterBackgroundNotification,
                object: nil, queue: operationQueue, usingBlock: {
                (notification: NSNotification!) in
                    print("程序进入到后台了")
        })
         
        //如果不需要的话，记得把相应的通知注册给取消，避免内存浪费或奔溃
        //notificationCenter.removeObserver(applicationDidEnterBackgroundObserver)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

##### 3，使用自定义的通知 
通知类型其实就是一个字符串，所以我们也可以使用自己定义的通知（同时也可以传递用户自定义数据）。
下面创建了两个观察者获取下载图片通知，同时收到通知后的处理函数内部添加了个3秒的等待。

--- ViewController.swift ---

```
import UIKit
 
class ViewController: UIViewController {
     
    let observers = [MyObserver(name: "观察器1"),MyObserver(name: "观察器2")]
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        print("发送通知")
        NSNotificationCenter.defaultCenter().postNotificationName("DownloadImageNotification",
            object: self, userInfo: ["value1":"hangge.com", "value2" : 12345])
        print("通知完毕")
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

--- MyObserver.swift ---


```
import UIKit
 
class MyObserver: NSObject {
     
    var name:String = ""
 
    init(name:String){
        super.init()
         
        self.name = name
        NSNotificationCenter.defaultCenter().addObserver(self, selector:"downloadImage:",
            name: "DownloadImageNotification", object: nil)
    }
     
    func downloadImage(notification: NSNotification) {
        let userInfo = notification.userInfo as! [String: AnyObject]
        let value1 = userInfo["value1"] as! String
        let value2 = userInfo["value2"] as! Int
         
        print("\(name) 获取到通知，用户数据是［\(value1),\(value2)］")
         
        sleep(3)
         
        print("\(name) 执行完毕")
    }
 
    deinit {
        //记得移除通知监听
        NSNotificationCenter.defaultCenter().removeObserver(self)
    }
    
}
```

运行结果如下：
> 发送通知
> 
> 观察器1 获取到通知，用户数据是［hangge.com,12345］
> 
> 观察器1 执行完毕
> 
> 观察器2 获取到通知，用户数据是［hangge.com,12345］
> 
> 观察器2 执行完毕
> 
> 通知完毕
