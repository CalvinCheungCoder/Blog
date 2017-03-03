> 我们在日常开发时，部分页面为了用户更方便的操作，以及实现更好的用户交互体验往往会使用 `UIGestureRecognizer` (手势识别器)


| Eng | Ch |
| --- | --- |
|UITapGestureRecognizer|轻拍|
|UISwipeGestureRecognizer|轻扫|
|UILongPressGestureRecognizer|长按|
|UIPanGestureRecognizer|平移|
|UIPinchGestureRecognizer|捏合/缩放|
|UIRotationGestureRecognizer|旋转|
|UIScreenEdgePanGestureRecognizer|屏幕边缘平移|

#### UITapGestureRecognizer 轻拍

```Objective-C
// 创建手势 使用initWithTarget:action:的方法创建
UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(tapView:)];
// 设置轻拍次数
tap.numberOfTapsRequired = 1;
// 设置手指个数
tap.numberOfTouchesRequired = 1;
// 添加到 View 上
[self.View addGestureRecognizer:tap];
```

实现轻拍事件

```Objective-C
-(void)tapView:(UITapGestureRecognizer *)sender{
    // 设置轻拍事件改变testView的颜色
    self.View.backgroundColor = [UIColor colorWithRed:arc4random()%256/255.0 green:arc4random()%256/255.0 blue:arc4random()%256/255.0 alpha:1];
}
```

#### UISwipeGestureRecognizer 轻扫

```Objective-C
// 创建手势
 UISwipeGestureRecognizer *swipe = [[UISwipeGestureRecognizer alloc]initWithTarget:self action:@selector(swipeView:)];
// 设置属性，swipe也是有两种属性设置手指个数及轻扫方向
swipe.numberOfTouchesRequired = 2;
// 设置轻扫方向(默认是从左往右)
// direction是一个枚举值有四个选项，我们可以设置从左往右，从右往左，从下往上以及从上往下
swipe.direction = UISwipeGestureRecognizerDirectionLeft;
[self.View addGestureRecognizer:swipe];
```

事件实现

```Objective-C
-(void)swipeView:(UISwipeGestureRecognizer *)sender{
    self.View.backgroundColor = [UIColor colorWithRed:arc4random()%256/255.0 green:arc4random()%256/255.0 blue:arc4random()%256/255.0 alpha:1];
}
```

#### UILongPressGestureRecognizer 长按

```Objective-C
UILongPressGestureRecognizer *longPress = [[UILongPressGestureRecognizer alloc]initWithTarget:self action:@selector(longPress:)];
//最小长按时间
longPress.minimumPressDuration = 2;
[self.View addGestureRecognizer:longPress];
```

事件实现

```Objective-C
-(void)longPress:(UILongPressGestureRecognizer *)sender{
    // 进行判断,在什么时候触发事件
    if (sender.state == UIGestureRecognizerStateBegan) {
        NSLog(@"长按状态");
        // 改变View颜色
        self.View.backgroundColor = [UIColor colorWithRed:arc4random()%256/255.0 green:arc4random()%256/255.0 blue:arc4random()%256/255.0 alpha:1];
    }
}
```

#### UIPanGestureRecognizer 平移

```Objective-C
UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(panView:)];
[self.View addGestureRecognizer:pan];
```
事件实现

```Objective-C
-(void)panView:(UIPanGestureRecognizer *)sender{
    CGPoint point = [sender translationInView:_testView];
    // sender.view.transform = CGAffineTransformMake(1, 0, 0, 1, point.x, point.y);

    // 平移一共两种移动方式
    // 第一种移动方法:每次移动都是从原来的位置移动
    // sender.view.transform = CGAffineTransformMakeTranslation(point.x, point.y);

    // 第二种移动方式:以上次的位置为标准(移动方式 第二次移动加上第一次移动量)
    sender.view.transform = CGAffineTransformTranslate(sender.view.transform, point.x, point.y);
    // 增量置为o
    [sender setTranslation:CGPointZero inView:sender.view];
     self.View.backgroundColor = [UIColor colorWithRed:arc4random()%256/255.0 green:arc4random()%256/255.0 blue:arc4random()%256/255.0 alpha:1];
}
```

#### UIPinchGestureRecognizer 捏合
```Objective-C
UIPinchGestureRecognizer *pinch = [[UIPinchGestureRecognizer alloc] initWithTarget:self action:@selector(pinchView:)];
[self.View addGestureRecognizer:pinch];
```
事件实现

```Objective-C
-(void)pinchView:(UIPinchGestureRecognizer *)sender{
    // scale 缩放比例
    // sender.view.transform = CGAffineTransformMake(sender.scale, 0, 0, sender.scale, 0, 0);
    // 每次缩放以原来位置为标准
    // sender.view.transform = CGAffineTransformMakeScale(sender.scale, sender.scale);

    // 每次缩放以上一次为标准
    sender.view.transform = CGAffineTransformScale(sender.view.transform, sender.scale, sender.scale);
    // 重新设置缩放比例 1是正常缩放.小于1时是缩小(无论何种操作都是缩小),大于1时是放大(无论何种操作都是放大)
    sender.scale = 1;
}
```

#### UIRotationGestureRecognizer 旋转
```Objective-C
 UIRotationGestureRecognizer *rotation = [[UIRotationGestureRecognizer alloc] initWithTarget:self action:@selector(rotationView:)];
 [self.View addGestureRecognizer:rotation];
```
事件实现

```Objective-C
-(void)rotationView:(UIRotationGestureRecognizer *)sender{
    // sender.view.transform = CGAffineTransformMake(cos(M_PI_4), sin(M_PI_4), -sin(M_PI_4), cos(M_PI_4), 0, 0);
    // 捏合手势两种改变方式
    // 以原来的位置为标准
    // sender.view.transform = CGAffineTransformMakeRotation(sender.rotation);//rotation 是旋转角度

    // 两个参数,以上位置为标准
    sender.view.transform = CGAffineTransformRotate(sender.view.transform, sender.rotation);
    // 消除增量
    sender.rotation = 0.0;
}
```
#### UIScreenEdgePanGestureRecognizer 屏幕边缘平移
```Objective-C
UIScreenEdgePanGestureRecognizer *screenEdgePan = [[UIScreenEdgePanGestureRecognizer alloc]initWithTarget:self action:@selector(screenEdgePanView:)];

// 注意:,使用屏幕边界平移手势,需要注意两点
// 1. 视图位置(屏幕边缘)
// 2. 设置edges属性
// 设置屏幕边缘手势支持方法
screenEdgePan.edges = UIRectEdgeLeft;
// 属性设置
[self.View addGestureRecognizer:screenEdgePan];
```
事件实现

```Objective-C
-(void)screenEdgePanView:(UIScreenEdgePanGestureRecognizer *)sender{
    // 计算偏移量
    CGPoint point = [sender translationInView:self.View];
    // 进行平移
    sender.view.transform = CGAffineTransformMakeTranslation(point.x, point.y);
}
```

