> 由于系统进度条 UIProgressView 使用的局限性，在很多 APP 需要显示进度时，系统进度条达不到想要的样式，只能自己重新封装，达到自己想要的样式

> 实现思路
1、创建基于 UIView 的 CustomProgressView 类
2、声明 UIProgressView 的底色、进度条颜色、进度
3、在 CustomProgressView.m 实现类似进度条样式以及进度条动画

一、创建基于 UIView 的 CustomProgressView

二、在 CustomProgressView.h 文件中声明进度条进度、进度条底部颜色、进度条颜色等
```Objective-C
@property (nonatomic, strong) UIColor *trackColor;      // 进度条背景颜色
@property (nonatomic, strong) UIColor *progressColor;   // 进度条颜色
@property (nonatomic, assign) CGFloat percent;          // 0 - 100

- (instancetype)initWithFrame:(CGRect)frame
```

三、在 CustomProgressView.m 文件中进行实现
```Objective-C
@interface CustomProgressView ()

@property (nonatomic, strong) CALayer *bgLayer;
@property (nonatomic, strong) CALayer *maskLayer;
@property (nonatomic, strong) CAGradientLayer *progressLayer;

@end

@implementation CustomProgressView

- (instancetype)initWithFrame:(CGRect)frame{
    
    self = [super initWithFrame:frame];
    if (self) {
        self.backgroundColor = [UIColor whiteColor];
        
        [self getGradientLayer];
    }
    return self;
}

- (void)getGradientLayer {
    
    // 进度条背景
    self.bgLayer = [CALayer layer];
    self.bgLayer.frame = CGRectMake(0, 0, self.bounds.size.width, self.bounds.size.height);
    self.bgLayer.masksToBounds = YES;
    self.bgLayer.cornerRadius = self.bounds.size.height / 2;
    [self.layer addSublayer:self.bgLayer];
    
    self.maskLayer = [CALayer layer];
    self.maskLayer.frame = CGRectMake(0, 0, self.bounds.size.width * self.percent / 100.f, self.bounds.size.height);
    self.maskLayer.borderWidth = self.bounds.size.height / 2;
    
    self.progressLayer =  [CAGradientLayer layer];
    self.progressLayer.frame = CGRectMake(0, 0, self.bounds.size.width, self.bounds.size.height);
    self.progressLayer.masksToBounds = YES;
    self.progressLayer.cornerRadius = self.bounds.size.height / 2;
    [self.progressLayer setStartPoint:CGPointMake(0, 0)];
    [self.progressLayer setEndPoint:CGPointMake(1, 0)];
    [self.progressLayer setMask:self.maskLayer];
    [self.layer addSublayer:self.progressLayer];
}

- (void)setPercent:(CGFloat)percent {
    [self setPercent:percent animated:YES];
}

- (void)setPercent:(CGFloat)percent animated:(BOOL)animated {
    
    _percent = percent;
    [NSTimer scheduledTimerWithTimeInterval:0 target:self selector:@selector(circleAnimation) userInfo:nil repeats:NO];
}

- (void)circleAnimation {
    
    // 进度条动画
    self.bgLayer.backgroundColor = self.trackColor.CGColor;
    self.progressLayer.backgroundColor = self.progressColor.CGColor;
    [CATransaction begin];
    [CATransaction setDisableActions:NO];
    [CATransaction setAnimationTimingFunction:[CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut]];
    [CATransaction setAnimationDuration:2];
    self.maskLayer.frame = CGRectMake(0, 0, self.bounds.size.width * _percent / 100.f, self.bounds.size.height);
    [CATransaction commit];
}
```

四、使用

```Objective-C
	// 进度条
    self.processView = [[CustomProgressView alloc] initWithFrame:CGRectMake(20, 100, 300, 20)];
    self.processView.progressColor = [UIColor colorWithRed:50/255 green:160/255 blue:255/255 alpha:1];
    self.processView.trackColor = [UIColor colorWithRed:50/255 green:160/255 blue:255/255 alpha:0.2];
    self.processView.percent = 34;
    [self.view addSubview:self.processView];
```

