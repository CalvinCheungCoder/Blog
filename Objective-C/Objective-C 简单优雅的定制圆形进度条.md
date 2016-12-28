Objective-C 简单优雅的定制圆形进度条

> 在某些需要显示进度的App中，有些时候进度条是一个圆形的形式显示的，圆形底部颜色教浅，圆形进度条颜色教深，然后根据进度在圆形上显示对应的角度，有时在圆形显示进度时还需要加上动画的加载效果。

现在就这一需求，对圆形进度条进行简单的定制

一、创建基于 UIView 的 CircleProgressView 类

二、CircleProgressView.h 中声明圆的各个属性和动画方法

```Objective-C
@interface CircleProgressView : UIView
{
    CAShapeLayer *_trackLayer;
    UIBezierPath *_trackPath;
    CAShapeLayer *_progressLayer;
    UIBezierPath *_progressPath;
}

@property (nonatomic, strong) UIColor *trackColor;      // 进度条背景颜色
@property (nonatomic, strong) UIColor *progressColor;   // 进度条颜色
@property (nonatomic) float progress;                   // 进度，0~1之间
@property (nonatomic) float progressWidth;              // 进度条宽度

// 带有动画效果
- (void)setProgress:(float)progress animated:(BOOL)animated;
```

三、在 CircleProgressView.m 中实现

* 画出圆形底部背景

```Objective-C
	// bezierPathWithOvalInRect
    _trackPath = [UIBezierPath bezierPathWithArcCenter:CGPointMake(CGRectGetWidth(self.bounds) / 2, CGRectGetHeight(self.bounds) / 2) radius:(self.bounds.size.width)/ 2 startAngle:0 endAngle:M_PI * 2 clockwise:YES];;
    _trackLayer.path = _trackPath.CGPath;
```

* 根据传入的进度确定进度条的起始点

```Objective-C
	_progressPath = [UIBezierPath bezierPathWithArcCenter:CGPointMake(CGRectGetWidth(self.bounds) / 2, CGRectGetHeight(self.bounds) / 2) radius:self.bounds.size.width/ 2 startAngle:- M_PI_2 endAngle:(M_PI * 2) * _progress - M_PI_2 clockwise:YES];
    _progressLayer.path = _progressPath.CGPath;
```

* 实现带有动画的进度条方法

```Objective-C
// 设置带有动画的进度
- (void)setProgress:(float)progress animated:(BOOL)animated
{
    [self setProgress:progress];
    // 给这个layer添加动画效果
    CABasicAnimation *pathAnimation = [CABasicAnimation animationWithKeyPath:@"strokeEnd"];
    pathAnimation.duration = 1.5*progress;
    pathAnimation.fromValue = [NSNumber numberWithFloat:0.0f];
    pathAnimation.toValue = [NSNumber numberWithFloat:1.0f];
    [_progressLayer addAnimation:pathAnimation forKey:nil];
}
```

完整的 CircleProgressView.m 代码

```Objective-C
- (id)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        
        // Initialization code
        _trackLayer = [CAShapeLayer new];
        _trackLayer.fillColor = nil;
        _trackLayer.frame = self.bounds;
        [self.layer addSublayer:_trackLayer];
        
        _progressLayer = [CAShapeLayer new];
        [self.layer addSublayer:_progressLayer];
        _progressLayer.fillColor = nil;
        _progressLayer.lineCap = kCALineCapRound;
        _progressLayer.frame = self.bounds;
        
        // 默认5
        self.progressWidth = 5;
    }
    return self;
}

- (void)setTrack
{
    // bezierPathWithOvalInRect
    _trackPath = [UIBezierPath bezierPathWithArcCenter:CGPointMake(CGRectGetWidth(self.bounds) / 2, CGRectGetHeight(self.bounds) / 2) radius:(self.bounds.size.width)/ 2 startAngle:0 endAngle:M_PI * 2 clockwise:YES];;
    _trackLayer.path = _trackPath.CGPath;
}

- (void)setProgress
{
    _progressPath = [UIBezierPath bezierPathWithArcCenter:CGPointMake(CGRectGetWidth(self.bounds) / 2, CGRectGetHeight(self.bounds) / 2) radius:self.bounds.size.width/ 2 startAngle:- M_PI_2 endAngle:(M_PI * 2) * _progress - M_PI_2 clockwise:YES];
    _progressLayer.path = _progressPath.CGPath;
}

// 设置进度条的宽度
- (void)setProgressWidth:(float)progressWidth
{
    _progressWidth = progressWidth;
    _trackLayer.lineWidth = _progressWidth;
    _progressLayer.lineWidth = _progressWidth;
    
    [self setTrack];
    [self setProgress];
}

- (void)setTrackColor:(UIColor *)trackColor
{
    _trackLayer.strokeColor = trackColor.CGColor;
}

- (void)setProgressColor:(UIColor *)progressColor
{
    _progressLayer.strokeColor = progressColor.CGColor;
}

// 设置进度
- (void)setProgress:(float)progress
{
    _progress = progress;
    
    [self setProgress];
}

// 设置带有动画的进度
- (void)setProgress:(float)progress animated:(BOOL)animated
{
    [self setProgress:progress];
    // 给这个layer添加动画效果
    CABasicAnimation *pathAnimation = [CABasicAnimation animationWithKeyPath:@"strokeEnd"];
    pathAnimation.duration = 1.5*progress;
    pathAnimation.fromValue = [NSNumber numberWithFloat:0.0f];
    pathAnimation.toValue = [NSNumber numberWithFloat:1.0f];
    [_progressLayer addAnimation:pathAnimation forKey:nil];
}
```

四、方法调用

* 导入 #import "CircleProgressView.h"

```
	CircleProgressView *progressView = [[CircleProgressView alloc]initWithFrame:CGRectMake(ScreenWidth/2-70,70, 140, 140)];
    progressView.progressWidth = 15;
    progressView.progressColor = RGBA(160, 160, 160,1);
    progressView.trackColor = RGBA(160, 160, 160,0.2);
    [progressView setProgress:0.6 animated:YES];
    [self.topView addSubview:progressView];
```


