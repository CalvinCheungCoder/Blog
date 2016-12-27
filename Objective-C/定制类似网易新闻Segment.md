在类似新闻页面时，常需要多个 ViewController 展示不同的信息，此时可以自定制类似Segment的控件，加入不同的控制器，在不同的控制器里展示处理数据和用户交互逻辑。

一、创建基于 UIView 的视图并命名为 CustomSegment

CustomSegment.h 代码
``` Objective-C
#import <UIKit/UIKit.h>

@interface CustomSegment : UIView <UIScrollViewDelegate>

@property (nonatomic, strong) NSArray *nameArray; // Title数组
@property (nonatomic, strong) NSArray *controllers; // 子控制器数组
@property (nonatomic, strong) UIView *segmentView; // Segment视图
@property (nonatomic, strong) UIScrollView *segmentScrollV; // 增加视图滚动
@property (nonatomic, strong) UILabel *line; // Title底部横线
@property (nonatomic , strong) UIButton *seleBtn; // 选中的按钮
@property (nonatomic, strong) UILabel *down;


- (instancetype)initWithFrame:(CGRect)frame  controllers:(NSArray*)controllers titleArray:(NSArray*)titleArray ParentController:(UIViewController*)parentC click:(int)Click;

@end
```

CustomSegment.m 实现封装的方法
``` Objective-C
#import "CustomSegment.h"

#define RGBA(r,g,b,a) [UIColor colorWithRed:r/255.0 green:g/255.0 blue:b/255.0 alpha:a]

#define RGB(r,g,b) RGBA(r,g,b,1.f)

@implementation CustomSegment

- (instancetype)initWithFrame:(CGRect)frame controllers:(NSArray *)controllers titleArray:(NSArray *)titleArray ParentController:(UIViewController *)parentC click:(int)Click
{
    if (self  =  [super initWithFrame:frame])
    {

        self.controllers = controllers;
        self.nameArray = titleArray;
        
        // Segment视图
        self.segmentView = [[UIView alloc]initWithFrame:CGRectMake(0, 0, frame.size.width, 40)];
        self.segmentView.backgroundColor = [UIColor whiteColor];
        self.segmentView.tag = 50;
        [self addSubview:self.segmentView];
        
        // Segment 底部视图增加手势滑动
        self.segmentScrollV = [[UIScrollView alloc]initWithFrame:CGRectMake(0, 40, frame.size.width, frame.size.height -40)];
        self.segmentScrollV.contentSize = CGSizeMake(frame.size.width*self.controllers.count, 0);
        self.segmentScrollV.delegate = self;
        self.segmentScrollV.showsHorizontalScrollIndicator = NO;
        self.segmentScrollV.pagingEnabled = YES;
        self.segmentScrollV.bounces = NO;
        [self addSubview:self.segmentScrollV];
        
        for (int i = 0;i<self.controllers.count;i++)
        {
            UIViewController *contr = self.controllers[i];
            [self.segmentScrollV addSubview:contr.view];
            contr.view.frame = CGRectMake(i*frame.size.width, 0, frame.size.width,frame.size.height);
            [parentC addChildViewController:contr];
            [contr didMoveToParentViewController:parentC];
        }
        
        for (int i = 0;i<self.controllers.count;i++)
        {
            UIButton *btn = [UIButton buttonWithType:UIButtonTypeCustom];
            btn.frame = CGRectMake(i*(frame.size.width/self.controllers.count), 0, frame.size.width/self.controllers.count, 40);
            btn.tag = i;
            [btn setTitle:self.nameArray[i] forState:(UIControlStateNormal)];
            [btn setTitleColor:[UIColor grayColor] forState:(UIControlStateNormal)];
            [btn setTitleColor:RGB(50, 160, 255) forState:(UIControlStateSelected)];
            [btn addTarget:self action:@selector(Click:) forControlEvents:(UIControlEventTouchUpInside)];
            btn.titleLabel.font = [UIFont systemFontOfSize:15];
            if (i == 0)
            {btn.selected = YES ;self.seleBtn = btn;
                btn.titleLabel.font = [UIFont systemFontOfSize:15];
            } else { btn.selected = NO; }
            
            [self.segmentView addSubview:btn];
        }
        self.line = [[UILabel alloc]initWithFrame:CGRectMake(0,38, frame.size.width/self.controllers.count, 2)];
        self.line.backgroundColor = RGB(50, 160, 255);
        self.line.tag = 100;
        [self.segmentView addSubview:self.line];
        
        self.down = [[UILabel alloc]initWithFrame:CGRectMake(0, 39.5, frame.size.width, 0.5)];
        self.down.backgroundColor = RGB(246, 246, 246);
        [self.segmentView addSubview:self.down];
        
        UIButton *btn = [[UIButton alloc]init];
        btn.tag  =  Click;
        [self Click:btn];
    }
    return self;
}


- (void)Click:(UIButton*)sender
{
    self.seleBtn.titleLabel.font = [UIFont systemFontOfSize:15];
    self.seleBtn.selected = NO;
    self.seleBtn = sender;
    self.seleBtn.selected = YES;
    self.seleBtn.titleLabel.font = [UIFont systemFontOfSize:15];
    [UIView animateWithDuration:0.2 animations:^{
        CGPoint frame = self.line.center;
        frame.x = self.frame.size.width/(self.controllers.count*2) +(self.frame.size.width/self.controllers.count)* (sender.tag);
        self.line.center = frame;
    }];
    [self.segmentScrollV setContentOffset:CGPointMake((sender.tag)*self.frame.size.width, 0) animated:YES ];
    
    [[NSNotificationCenter defaultCenter] postNotificationName:@"SelectVC" object:sender userInfo:nil];
}

- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView
{
    [UIView animateWithDuration:0.2 animations:^{
        CGPoint frame = self.line.center;
        frame.x = self.frame.size.width/(self.controllers.count*2) +(self.frame.size.width/self.controllers.count)*(self.segmentScrollV.contentOffset.x/self.frame.size.width);
        self.line.center = frame;
    }];
    UIButton *btn = (UIButton*)[self.segmentView viewWithTag:(self.segmentScrollV.contentOffset.x/self.frame.size.width)];
    self.seleBtn.selected = NO;
    self.seleBtn = btn;
    self.seleBtn.selected = YES;
}
```

至此，Segment 封装已经完成，主要功能有：
* 手动设置默认展示的ViewController，即Btn的Tag值
* 在点击按钮 ViewController 切换时添加移动动画

调用方法如下：
* 声明3个视图控制器
* 设置视图控制器数组和Title数组
* 设置CustomSegment的Frame和默认展示的控制器

```Objective-C
	OneViewController *one = [[OneViewController alloc]init];
    TwoViewController *two = [[TwoViewController alloc]init];
    ThreeViewController *three = [[ThreeViewController alloc]init];
    NSArray *controllers=@[one,two,three];
    
    NSArray *titleArray =@[@"One",@"Two",@"Three"];
    CustomSegment *rcs = [[CustomSegment alloc]initWithFrame:CGRectMake(0, 20, [UIScreen mainScreen].bounds.size.width, [UIScreen mainScreen].bounds.size.height-20) controllers:controllers titleArray:titleArray ParentController:self click:0];
    [self.view addSubview:rcs];
```



