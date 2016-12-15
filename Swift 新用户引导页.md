##### 1、如何检测应用是第一次登陆启动
我们可以使用NSUserDefaults类来解决这个问题。其特点是不会因应用的关闭、系统的重启而丢失。所以可以用来标记是否启动过。

##### 2、新手引导视图控制器我们使用UIScrollView
比如我们设置了一套新手引导图共三张，都添加到UIScrollView里，这时UIScrollView的内容宽度是3倍于照片或者屏幕的宽度。

##### 3、为适应不同分辨率，需要设计几套不同尺寸的图
iOS图片资源的命名规则是：basename + screen size modifier + urischeme + orientation + scale + device + .ext

- basename：文件名
- screen size modifier：屏幕尺寸修饰符（iPhone5出现后才有，如 -568h）
- urischeme：标识URI方案的字符串（一般情况不需要关心）
- orientation：屏幕方向（横屏为-Landscape，竖屏为-Portrait）
- scale：缩放尺寸（普通屏不需要，Retina屏为@2x，iPhone6后多了个@3x）
- device：设备类型（~ipad表示供iPad使用）
- .ext：文件扩展名（可以是png或其他格式）

尽管文件很复杂，但调用却很简单，只要写上basename.ext即可。

##### 6、入口类：AppDelegate.swift

```
import UIKit
 
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
     
    var window: UIWindow?
     
    func application(application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
        //增加标识，用于判断是否是第一次启动应用...
        if (!(NSUserDefaults.standardUserDefaults().boolForKey("everLaunched"))) {
            NSUserDefaults.standardUserDefaults().setBool(true, forKey:"everLaunched")
            let guideViewController = GuideViewController()
            self.window!.rootViewController=guideViewController;
            print("guideview launched!")
        }
        return true
    }
     
    func applicationWillResignActive(application: UIApplication) {
    }
     
    func applicationDidEnterBackground(application: UIApplication) {
    }
     
    func applicationWillEnterForeground(application: UIApplication) {
    }
     
    func applicationDidBecomeActive(application: UIApplication) {
    }
     
    func applicationWillTerminate(application: UIApplication) {
    }
}
```

##### 7、向导页面：GuideViewController.swift


```
import UIKit
 
class GuideViewController:UIViewController,UIScrollViewDelegate
{
    //页面数量
    var numOfPages = 3
     
    override func viewDidLoad()
    {
        let frame = self.view.bounds
        //scrollView的初始化
        let scrollView = UIScrollView()
        scrollView.frame = self.view.bounds
        scrollView.delegate = self
        //为了能让内容横向滚动，设置横向内容宽度为3个页面的宽度总和
        scrollView.contentSize = CGSizeMake(frame.size.width * CGFloat(numOfPages),
                                            frame.size.height)
        print("\(frame.size.width*CGFloat(numOfPages)),\(frame.size.height)")
        scrollView.pagingEnabled = true
        scrollView.showsHorizontalScrollIndicator = false
        scrollView.showsVerticalScrollIndicator = false
        scrollView.scrollsToTop = false
        for i in 0..<numOfPages{
            let imgfile = "jianjie\(Int(i+1)).png"
            print(imgfile)
            let image = UIImage(named:"\(imgfile)")
            let imgView = UIImageView(image: image)
            imgView.frame = CGRectMake(frame.size.width*CGFloat(i),CGFloat(0),
                                     frame.size.width,frame.size.height)
            scrollView.addSubview(imgView)
        }
        scrollView.contentOffset = CGPointZero
        self.view.addSubview(scrollView)
    }
     
    //scrollview滚动的时候就会调用
    func scrollViewDidScroll(scrollView: UIScrollView)
    {
        print("scrolled:\(scrollView.contentOffset)")
        let twidth = CGFloat(numOfPages-1) * self.view.bounds.size.width
        //如果在最后一个页面继续滑动的话就会跳转到主页面
        if(scrollView.contentOffset.x > twidth)
        {
            let mainStoryboard = UIStoryboard(name:"Main", bundle:nil)
            let viewController = mainStoryboard.instantiateInitialViewController()
            self.presentViewController(viewController!, animated: true, completion:nil)
        }
    }
}
```
