iOS 文字上下翻转(跑马灯效果)

一、封装基于 UIView 的 HU_ScycleScrollView类

HU_ScycleScrollView.h

```Objective-C

#import <UIKit/UIKit.h>

typedef NS_ENUM(NSInteger, pageControlAligment){
  
    pageControlAligmentCenter = 0,
    pageControlAligmentLeft
};

@class HU_ScycleScrollView;
@protocol ScyleScrollViewDelegate <NSObject>

- (void)scyleScrollView:(HU_ScycleScrollView *)scyleView index:(NSInteger)index;

@end


@interface HU_ScycleScrollView : UIView

/** 标题数组 */
@property (nonatomic,strong)NSArray *titles;

/** 协议 */
@property (nonatomic,assign)id<ScyleScrollViewDelegate> delegate;

@end

```

HU_ScycleScrollView.m

```Objective-C

#import "HU_ScycleScrollView.h"
#define SCYLE_WIDTH CGRectGetWidth(self.frame)
#define SCYLE_HEIGHT CGRectGetHeight(self.frame)

#define TitleColor [UIColor whiteColor]
#define TitleFont [UIFont systemFontOfSize:12]

@interface HU_ScycleScrollView()<UIScrollViewDelegate>

/** 延迟时间 */
@property (nonatomic,assign)NSTimeInterval intervalTime;
/** 滑动视图 */
@property (nonatomic,strong)UIScrollView *scrollView;
/** 延时器 */
@property (nonatomic,strong)NSTimer *delayTimer;
/** 目前标题 */
@property (nonatomic,strong)UILabel *currentTitleView;
/** 目前标题位置 */
@property (nonatomic,assign)NSInteger currentTitIndex;
/** 下一个标题 */
@property (nonatomic,strong)UILabel *nextTitleView;
/** 下一个标题的位置 */
@property (nonatomic,assign)NSInteger nextTitIndex;

@end

@implementation HU_ScycleScrollView

#pragma mark --
#pragma mark -- 初始化
- (instancetype)initWithFrame:(CGRect)frame{
 
    if (self = [super initWithFrame:frame]) {
        self.intervalTime = 5;
        [self setupScycleView];
    }
    return self;
}

#pragma mark --
#pragma mark -- 创建视图
- (void)setupScycleView{
 
    // 添加scrollView
    UIScrollView *scrollView = [[UIScrollView alloc]initWithFrame:self.bounds];
    scrollView.pagingEnabled = YES;
    scrollView.scrollEnabled = YES;
    scrollView.userInteractionEnabled = NO;
    scrollView.bounces = NO;
    scrollView.delegate = self;
    scrollView.contentSize = CGSizeMake(SCYLE_WIDTH, SCYLE_HEIGHT * 2);
    scrollView.contentOffset = CGPointMake(0, 0);
    scrollView.showsHorizontalScrollIndicator = NO;
    scrollView.showsVerticalScrollIndicator = NO;
    [self addSubview:scrollView];
    self.scrollView = scrollView;
    
    // 创建3个UILabel
    [self setupThreeTitleView];
    
}

- (void)setupThreeTitleView{
    
    // 目前标题
    UILabel *currentTitleView = [[UILabel alloc]initWithFrame:CGRectMake(5, 0, SCYLE_WIDTH-10, SCYLE_HEIGHT)];
    currentTitleView.userInteractionEnabled = YES;
    currentTitleView.font = TitleFont;
    currentTitleView.textColor = TitleColor;
    [self.scrollView addSubview:currentTitleView];
    self.currentTitleView = currentTitleView;
    // 给目前标题添加手势
    UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(clickTheCurrentImgAction:)];
    [self addGestureRecognizer:tap];
    
    // 下一个标题
    UILabel *nextTitleView = [[UILabel alloc]initWithFrame:CGRectMake(5, SCYLE_HEIGHT, SCYLE_WIDTH-10, SCYLE_HEIGHT)];
    nextTitleView.font = TitleFont;
    nextTitleView.textColor = TitleColor;
    [self.scrollView addSubview:nextTitleView];
    self.nextTitleView = nextTitleView;
}

- (void)setTitles:(NSArray *)titles{

    _titles = titles;
    // 创建延时器
    [self renewSetDelayTimer];
    // 更新图片位置
    [self updateScycelScrollViewTitleIndex];
}

#pragma mark --
#pragma mark -- 更新图片位置
- (void)updateScycelScrollViewTitleIndex{
 
    if (self.titles.count > 0) {
        [self addTheTitleUrlStr:self.titles[self.currentTitIndex] titleView:_currentTitleView];
        [self addTheTitleUrlStr:self.titles[self.nextTitIndex] titleView:_nextTitleView];
    }
}

#pragma mark --
#pragma mark -- 解析图片并添加到imageView上
- (void)addTheTitleUrlStr:(NSString *)title titleView:(UILabel *)titleView{
    
    titleView.text = title;
}

#pragma mark --
#pragma mark -- 延时器执行方法
- (void)useTimerIntervalUpdateScrollViewContentOffSet:(NSTimer *)timer{
    [_scrollView setContentOffset:CGPointMake(0, SCYLE_HEIGHT) animated:YES];
}

#pragma mark --
#pragma mark -- 点击图片执行方法
- (void)clickTheCurrentImgAction:(UITapGestureRecognizer *)tap{
    if ([_delegate respondsToSelector:@selector(scyleScrollView:index:)]) {
        [self.delegate scyleScrollView:self index:_currentTitIndex];
    }
}

#pragma mark --
#pragma mark -- 滑动结束时停止动画
- (void)scrollViewDidEndScrollingAnimation:(UIScrollView *)scrollView{
    [self scrollViewDidEndDecelerating:scrollView];
}

#pragma mark --
#pragma mark -- 减速滑动时
- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView{
    
    int offSet = floor(scrollView.contentOffset.y);
    if (offSet == SCYLE_HEIGHT){
        self.currentTitIndex = self.nextTitIndex;
    }
    
    // 更新标题位置
    [self updateScycelScrollViewTitleIndex];
    // 设置偏移量
    scrollView.contentOffset = CGPointMake(0, 0);
}

#pragma mark --
#pragma mark -- 重新设置延时器
- (void)renewSetDelayTimer{
    // 添加延迟器
    self.delayTimer = [NSTimer scheduledTimerWithTimeInterval:self.intervalTime target:self selector:@selector(useTimerIntervalUpdateScrollViewContentOffSet:) userInfo:nil repeats:YES];
    // 加入事件循环中
    [[NSRunLoop mainRunLoop] addTimer:self.delayTimer forMode:NSRunLoopCommonModes];
}

// 上一个标题位置
- (NSUInteger)beforeTitIndex{
    
    if (self.currentTitIndex == 0) {
        return self.titles.count - 1;
    }else{
        return self.currentTitIndex - 1;
    }
}

// 下一个标题的位置
- (NSInteger)nextTitIndex{
    
    if (self.currentTitIndex < (self.titles.count - 1)) {
        return self.currentTitIndex + 1;
    }else{
        return 0;
    }
}

@end

```

二、调用

```Objective-C

NSArray *images = @[@"测试标题一，这是一条跑马灯的文字效果",@"测试标题2，这条标题很长，测试一下长度过长的时候显示效果",@"测试标题3，这是一条跑马灯的文字效果",@"测试标题4，这是一条跑马灯的文字效果，还行吧"];
    
    _scyleSV = [[HU_ScycleScrollView alloc]initWithFrame:CGRectMake(10, 80, self.view.bounds.size.width-20, 25)];
    _scyleSV.backgroundColor = [UIColor lightGrayColor];
    _scyleSV.layer.cornerRadius = 3;
    _scyleSV.titles = images;
    _scyleSV.userInteractionEnabled = YES;
    _scyleSV.delegate = self;
    [self.view addSubview:_scyleSV];
    
```

实现代理方法

```Objective-C

#pragma mark --
#pragma mark -- ScyleScrollViewDelegate
- (void)scyleScrollView:(HU_ScycleScrollView *)scyleView index:(NSInteger)index{
    NSLog(@"----- %ld",index);
}

```

[源代码地址：XLsn0wTextCarousel 作者：XLsn0w](https://github.com/XLsn0w/XLsn0wTextCarousel)

