iOS App 新版本检测并提示更新『绕过 Apple 审核』

一、封装版本检测方法

```Objective-C
/**
 *  版本验证
 *
 *  @param block 是否为最新版本
 */
+ (void)getVersionInfo:(void (^)(BOOL isLastVersion))block{
    NSDictionary *infoDic = [[NSBundle mainBundle] infoDictionary];
    
    // 先获取 App 当前版本号
    NSString *currentVersion = [infoDic objectForKey:@"CFBundleShortVersionString"];
    MyLog(@"当前版本号为------%@",currentVersion);
    
    // 查询 AppStore 上App的版本号，url 中 YOUAPPID 替换为你要查询的APPID
    NSString *url = @"http://itunes.apple.com/cn/lookup?id=YOUAPPID";
    // 此为封装的 AFNetworking 网络请求方法，可根据自己需要替换
    [Networking requestDataByURL:url Parameters:nil success:^(NSURLSessionDataTask *operation, id responseObject) {
        
        NSArray *infoContent = [responseObject objectForKey:@"results"];
        // Appstore 版本号
        NSString *lastVersion = [[infoContent objectAtIndex:0] objectForKey:@"version"];
        MyLog(@"最新的版本号为--------%@",lastVersion);
        // App更新信息
        NSString *iosUpVersionStr = [[infoContent objectAtIndex:0] objectForKey:@"releaseNotes"];
        [Tools addValue:iosUpVersionStr forKey:iosUpVersionDes];
        
        // 去除App版本号中的 . ，并进行判断是否提示升级，可以绕过 Apple 审核。
        NSString *curr = [currentVersion stringByReplacingOccurrencesOfString:@"." withString:@""];
        NSString *last = [lastVersion stringByReplacingOccurrencesOfString:@"." withString:@""];
        MyLog(@"%@%@",curr,last);
        
        // 本地版本号长度 <= 线上版本号长度
        if (curr.length <= last.length) {
            
            if ([curr intValue] < [last intValue]) {
                block(NO);
            }else{
                block(YES);
            }
        }else{
            
            curr = [curr substringToIndex:last.length];
            if ([curr intValue] < [last intValue]) {
                block(NO);
            }else{
                block(YES);
            }
        }
    } failBlock:^(NSURLSessionDataTask *operation, NSError *error) {
        MyLog(@"检查更新失败");
        MyLog(@"error %@",error.localizedDescription);
    }];
}
```

二、在需要提示更新的页面调用方法并进行对应的操作

```Objective-C

[Tools getVersionInfo:^(BOOL isLastVersion) {
        if (isLastVersion == YES) {
            MyLog(@"最新版");
        }else{
            MyLog(@"去更新");
        }
    }];
    
```

> 此版本验证只会在当前版本号 < 线上版本号时才会提示更新，可以很好的规避 Apple 的审核。


