> 开发中，经常会用到定时执行网络请求、倒计时、计时器等功能，本篇文章介绍在iOS开发中，Swift怎样使用GCD实现这些功能。

执行一次

下面的代码将会在5秒后执行，且只执行一次。

```Swift
let time: NSTimeInterval = 5.0
let delay = dispatch_time(DISPATCH_TIME_NOW, Int64(time * Double(NSEC_PER_SEC)))

dispatch_after(delay, dispatch_get_main_queue()) {
    self.getTaskList(false)
}
```

执行多次

下面的代码是一个60秒倒计时的例子。
```Swift
var _timeout: Int = 60
let _queue: dispatch_queue_t = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
let _timer: dispatch_source_t = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, _queue)

// 每秒执行
dispatch_source_set_timer(_timer, dispatch_walltime(nil, 0), 1 * NSEC_PER_SEC, 0)   

dispatch_source_set_event_handler(_timer) { () -> Void in
    
    if _timeout <= 0 {
        
        // 倒计时结束
        dispatch_source_cancel(_timer)  
        
        dispatch_async(dispatch_get_main_queue(), { () -> Void in
            
            // 如需更新UI 代码请写在这里
        })
        
    } else {
        
        print(_timeout)
        _timeout--
        
        dispatch_async(dispatch_get_main_queue(), { () -> Void in
           
            // 如需更新UI 代码请写在这里
        })
        
    }
}

dispatch_resume(_timer)

```
