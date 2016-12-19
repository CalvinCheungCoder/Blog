在 iOS 常用的按钮控件中，有时候我们需要按钮中图片和文字共同显示，有时候文字在左，图片在右；有时候图片在上，文字在下等等。此时就需要我们自定制按钮并设置图片和文字的位置

``` Objective-C

- (void)createBtnView{
    
    // 图片在左，文字在右
    UIButton *btn = [UIButton buttonWithType:UIButtonTypeCustom];
    btn.frame = CGRectMake(50, 120, 100, 30);
    [btn setTitle:@"测试" forState:(UIControlStateNormal)];
    [btn setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];
    [btn setImage:[UIImage imageNamed:@"UpCrow"] forState:(UIControlStateNormal)];
    [btn setImage:[UIImage imageNamed:@"down"] forState:(UIControlStateHighlighted)];
    btn.adjustsImageWhenHighlighted = NO;
    [btn setContentEdgeInsets:UIEdgeInsetsMake(0, 0, 0, 0)];
    [self.view addSubview:btn];
    
    // 文字在左，图片在右
    // 通过调节其偏移量来解决，image相当于左边偏移了label的宽度距离，右边相应减少偏移label的距离，titleLabel则相反。
    UIButton *rightBtn = [UIButton buttonWithType:UIButtonTypeCustom];
    rightBtn.frame = CGRectMake(50, 160, 100, 30);
    [rightBtn setTitle:@"测试长度" forState:(UIControlStateNormal)];
    [rightBtn setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];
    [rightBtn setImage:[UIImage imageNamed:@"UpCrow"] forState:(UIControlStateNormal)];
    [rightBtn setImage:[UIImage imageNamed:@"down"] forState:(UIControlStateHighlighted)];
    rightBtn.adjustsImageWhenHighlighted = NO;
    [rightBtn setContentEdgeInsets:UIEdgeInsetsMake(0, 0, 0, 0)];
    CGFloat imgWidth = rightBtn.imageView.bounds.size.width;
    CGFloat labWidth = rightBtn.titleLabel.bounds.size.width;
    [rightBtn setImageEdgeInsets:UIEdgeInsetsMake(0, labWidth, 0, -labWidth)];
    [rightBtn setTitleEdgeInsets:UIEdgeInsetsMake(0, -imgWidth, 0, imgWidth)];
    [self.view addSubview:rightBtn];
    
    // 图片在上，文字在下
    UIButton *topBtn = [UIButton buttonWithType:UIButtonTypeCustom];
    [topBtn setImage:[UIImage imageNamed:@"down"] forState:UIControlStateNormal];
    [topBtn setImage:[UIImage imageNamed:@"down"] forState:UIControlStateHighlighted];
    [topBtn setTitle:@"我的主帖" forState:UIControlStateNormal];
    topBtn.titleLabel.font = [UIFont systemFontOfSize:14];
    [topBtn setTitleColor:[UIColor redColor] forState:UIControlStateNormal];
    [topBtn setTitleColor:[UIColor yellowColor] forState:UIControlStateHighlighted];
    [topBtn setAdjustsImageWhenHighlighted:NO];
    topBtn.frame = CGRectMake(0, 300, 100, 49);
    CGFloat totalHeight = (topBtn.imageView.frame.size.height + topBtn.titleLabel.frame.size.height);
    [topBtn setImageEdgeInsets:UIEdgeInsetsMake(-(totalHeight - topBtn.imageView.frame.size.height), 0.0, 0.0, -topBtn.titleLabel.frame.size.width)];
    [topBtn setTitleEdgeInsets:UIEdgeInsetsMake(0.0, -topBtn.imageView.frame.size.width, -(totalHeight - topBtn.titleLabel.frame.size.height),0.0)];
    [self.view addSubview:topBtn];
}

```

