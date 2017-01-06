### iOS 应用间跳转『一篇就够了』

本文分三部分进行解释并分析应用间跳转的原理和实现

* App 应用跳转的原理解析
* 如何实现两个 App 应用之间的跳转
* 如何实现两个 App 之间跳转到指定界面

#### 一、App 应用跳转的原理解析

相信从一个应用跳转到另一个应用大家并不陌生，最常见的莫过于第三方登录，支付宝支付等等。这些东西大家都耳熟能详，集成进来也很简单，跟着第三方sdk集成文档一步步走下来就是了，通常sdk集成文档都需要你在工程中配置一堆堆的东西，但是配置的这些东西，你真的明白了吗？比如下面这个，第三方登录或分享需要你配置的URL Schemes：

![](https://github.com/CalvinCheungCoder/Blog/blob/master/Objective-C/JumpBetweenApplications/urlshare.jpeg)

不明白呢没关系，开始我也不明白，但是这篇博文看完后，相信你会明白的，下面正式进入主题：

1、一些概念的补充

* 协议：双方互相遵守的一种规范，只有遵守共同的协议规范才能进行彼此的通信。比如我们最熟悉的网络协议——http协议。
* URL:资源的路径或地址。在IOS中有一个专门用于包装资源路径的类——NSURL。
* 一个完整URL的组成

```Objective-C
例如：http://123.0.0.1/path?page=100
“http://”：协议类型
“123.0.0.1”：服务器ip地址
“/path”：资源存放的是路径
“page=100”：请求的参数
```
* NSURL包装一个完整地址

```Objective-C
  NSURL *url = [NSURL URLWithString:@"http://123.0.0.1/path?page=100"];
  NSLog(@"scheme(协议):%@",url.scheme);
  NSLog(@"host（域名）:%@",url.host);
  NSLog(@"path（路径）:%@",url.path);
  NSLog(@"query（参数）:%@",url.query);
```

打印结果如下：

```Objective-C
2016-12-02 14:50:38.442 TestDemo[5632:406869] scheme(协议):http
2016-12-02 14:50:38.442 TestDemo[5632:406869] host（域名）:123.0.0.1
2016-12-02 14:50:38.442 TestDemo[5632:406869] path（路径）:/path
2016-12-02 14:50:38.442 TestDemo[5632:406869] query（参数）:page=100
```

2、跳转的原理

在 iOS 中，从一个 app 打开另一个 app ，这必然牵扯到两个 app 之间的交互和通信，像这种涉及到整个应用程序层面的事情，苹果有一个专门的类来管理 *UIApplication*。在ios中*UIApplication* 其实就是代表着应用程序，这点从它的命名就可以窥之。而我们要打开另一个应用程序，如何实现呢？

很简单，其实就是 *UIApplication* 下面这个 的API

```Objective-C
/**
 通过应用程序打开一个资源路径
@param url 资源路径的地址
@return 返回成功失败的信息
 */
- (BOOL)openURL:(NSURL*)url;
```

它的一些我们非常熟悉的用法：

```Objective-C
//拨打系统电话
 NSURL *url = [NSURL URLWithString:@"tel://10086"];
 [[UIApplication sharedApplication] openURL:url];
```

```Objective-C
//发送系统短信
 NSURL *url = [NSURL URLWithString:@"sms://1383838438"];
 [[UIApplication sharedApplication] openURL:url];
```

看到这里也许有人会有疑问：拨打系统电话、发送系统短信跟我本篇要讲的应用间的跳转有什么关系呢？

呵呵，不要着急，重点来了：你难道不觉得拨打系统电话、发送系统短信其实就是应用间的跳转吗？只要一执行以上两个方法就会从你当前的应用跳转到系统的拨打电话界面、发送短信界面，这难道还不够应用间的跳转吗？其实你也可以这么理解：拨打系统电话、发送短信它俩就是手机本身自带的两个app应用。

写到这里答案已经呼之欲出，上面打电话和发短信的实现代码大同小异，唯一的区别是传递的NSURL参数不一样，导致他们跳转到不同的应用场景。我们再仔细分析下传给它们的NSURL参数，就会发现NSURL的scheme（协议）不一样，打电话时“tel://”协议，发短信是“sms://”协议。（对协议有疑问的童鞋可以拉上去看）

一个总结：一个应用能打开另一个应用的必然条件是，另一个应用必须配置一个scheme（协议），这样应用程序才能根据协议找到需要打开的应用.

#### 二、实现两个app间的跳转

创建两个示例Demo，TestDemo和Test2Demo，现在需要实现从Test2Demo跳转到TestDemo中

1、在被跳转的TestDemo配置一个协议scheme，这里命名为test（名字可随意配置，当然最好是英文并且跟你项目相关）

targets -> info -> URL Types ->URL Scheme ->填写协议
![](https://github.com/CalvinCheungCoder/Blog/blob/master/Objective-C/JumpBetweenApplications/testDemo.jpeg)
注意：不需要填写成“test://”

2、在Test2Demo执行跳转的方法中实现下面方法

```Objective-C
-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    NSURL *url = [NSURL URLWithString:@"test://"];

    if ([[UIApplication sharedApplication] canOpenURL:url]) {

        [[UIApplication sharedApplication] openURL:url];

    }else{
        NSLog(@"没有安装应用");
    }
}
```

ok，到这里如果你的系统是ios9.0以下，已经大大功告成了。但是，如果是9.0以后，请看下一步。

3、配置协议白名单

在Test2Demo的info.plist文件中增加一个*LSApplicationQueriesSchemes*字段，把它设置为数组类型，并配置需要跳转的协议名单

![](https://github.com/CalvinCheungCoder/Blog/blob/master/Objective-C/JumpBetweenApplications/testDemo2.jpeg)

到此，两个应用间的跳转已经完全实现，其实说穿了就三步，so easy！但是，很多时候，我不仅要跳转到一个应用上，而且还需要跳转到应用的指定界面，想知道怎么处理请接着往下看。

#### 三、跳转到指定界面

想要跳转到指定界面，必然是上一个app告诉下一个app（被跳转的app）需要跳转到哪个界面，而如何告诉它这里便涉及到两个app的通信。我们从上面可以知道，两个app之间的跳转只需要配置一个scheme，然后通过UIApplication调用它的对象方法openURL:即可实现，除此之外再也没有实现任何代码了。而这之间是如何通信的呢？
答案依然是协议，请看下面步骤：

1、在”test://”协议后面的域名加上一些字段用来标记需要跳转的界面

```Objective-C
//进入更多界面
- (IBAction)intoMore:(id)sender {
    NSURL *url = [NSURL URLWithString:@"test://more"];

    if ([[UIApplication sharedApplication] canOpenURL:url]) {

        [[UIApplication sharedApplication] openURL:url];
    }else{
        NSLog(@"没有安装应用");
    }

}

//进入设置界面
- (IBAction)intoSet:(id)sender {

    NSURL *url = [NSURL URLWithString:@"test://set"];

    if ([[UIApplication sharedApplication] canOpenURL:url]) {

        [[UIApplication sharedApplication] openURL:url];
    }else{
        NSLog(@"没有安装应用");
    }

}
```

2、来到被跳转的应用TestDemo的AppDelegate类的.m文件中，监听其代理方法application:handleOpenURL:

```Objective-C
// 当应用程序将要被其他程序打开时，会先执行此方法，并传递url过来
// 注：下面这个方法9.0后就过期了，请注意适配，9.0后用这个方法：application:openURL:options:
-(BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url
{
    NSLog(@"url:%@",url.absoluteString);
    NSLog(@"host:%@",url.host);
    if ([url.host isEqualToString:@"more"]) {
        NSLog(@"进入更多界面");
        // 到此做界面的跳转
    }

    if ([url.host isEqualToString:@"set"]) {
        NSLog(@"进入设置界面");
        // 到此做界面的跳转
    }

    return YES;
}
```

当Test2Demo点击进入更多界面打印如下：

```Objective-C
2016-12-02 17:11:17.680 TestDemo[6507:495044] url:test://more
2016-12-02 17:11:17.681 TestDemo[6507:495044] host:more
2016-12-02 17:11:17.681 TestDemo[6507:495044] 进入更多界面
```

当Test2Demo点击进入设置界面打印如下：

```Objective-C
2016-12-02 17:10:38.745 TestDemo[6507:495044] url:test://set
2016-12-02 17:10:38.745 TestDemo[6507:495044] host:set
2016-12-02 17:10:38.745 TestDemo[6507:495044] 进入设置界面
```

[原文链接:iOS应用之间的跳转，看这篇就够了
](http://ios.jobbole.com/91129/)


