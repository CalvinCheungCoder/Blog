> iOS 禁止 Textfield 复制粘贴以及选择功能

在很多涉及密码的文本框我们一般都不让用户进行复制和粘贴功能

1、创建基于 UITextfield 的 BaseTextfield

2、在 BaseTextfield.m 中写入如下代码

``` Objective-C
- (BOOL)canPerformAction:(SEL)action withSender:(id)sender
{
    
    UIMenuController *menuController = [UIMenuController sharedMenuController];
    
    if(menuController) {
        
        [UIMenuController sharedMenuController].menuVisible = NO;
        
    }
    
    return NO;
}
```

3、在创建 UITextfield 时基于 BaseTextfield 创建，此时的 Textfield 就没有了粘贴、选择以及全选功能。

