#### 如何使用 KeyChain 保存和获取 UDID
 
##### 一、iOS 不同版本获取 UDID 的方法比较
　　
1、iOS 5.0


　　iOS 2.0版本以后`UIDevice`提供一个获取设备唯一标识符的方法`uniqueIdentifier`，通过该方法我们可以获取设备的序列号，这个也是目前为止唯一可以确认唯一的标示符。好景不长，因为该唯一标识符与手机一一对应，苹果觉得可能会泄露用户隐私，所以在 iOS 5.0之后该方法就被废弃掉了。


2、iOS 6.0


iOS 6.0系统新增了两个用于替换`uniqueIdentifier`的接口，分别是：`identifierForVendor`，`advertisingIdentifier`。


`identifierForVendor`接口的官方文档介绍如下:


> The value of this property is the same for apps that come from the same vendor running on the same device. A different value is returned for apps on the same device that come from different vendors, and for apps on different devices regardless of vendor.
>
> The value of this property may be nil if the app is running in the background, before the user has unlocked the device the first time after the device has been restarted. If the value is nil, wait and get the value again later.
>
> The value in this property remains the same while the app (or another app from the same vendor) is installed on the iOS device. The value changes when the user deletes all of that vendor’s apps from the device and subsequently reinstalls one or more of them. Therefore, if your app stores the value of this property anywhere, you should gracefully handle situations where the identifier changes.


大概意思就是“同一开发商的APP在指定机器上都会获得同一个ID。当我们删除了某一个设备上某个开发商的所有APP之后，下次获取将会获取到不同的ID。” 也就是说我们通过该接口不能获取用来唯一标识设备的ID。


问题总是难不倒聪明的程序员，于是大家想到了使用WiFi的mac地址来取代已经废弃了的`uniqueIdentifier`方法。具体的方法晚上有很多，大家感兴趣的可以自己找找，这儿提供一个网址: [http://stackoverflow.com/questions/677530/how-can-i-programmatically-get-the-mac-address-of-an-iphone](http://stackoverflow.com/questions/677530/how-can-i-programmatically-get-the-mac-address-of-an-iphone)


3、iOS 7.0


　　iOS 7 中苹果再一次无情的封杀`mac`地址，但我们可以从`KeyChain`中读取信息。


##### 二、KeyChain介绍


我们搞 iOS 开发，一定都知道 OS X 里面的`KeyChain`(钥匙串)，通常要调试的话，都得安装证书之类的，这些证书就是保存在`KeyChain`中，还有我们平时浏览网页记录的账号密码也都是记录在`KeyChain`中。iOS中的`KeyChain`相比OS X比较简单，整个系统只有一个`KeyChain`，每个程序都可以往`KeyChain`中记录数据，而且只能读取到自己程序记录在`KeyChain`中的数据。iOS 中`Security.framework`框架提供了四个主要的方法来操作`KeyChain`:



```Objective-C
// 查询
OSStatus SecItemCopyMatching(CFDictionaryRef query, CFTypeRef *result);

// 添加
OSStatus SecItemAdd(CFDictionaryRef attributes, CFTypeRef *result);

// 更新KeyChain中的Item
OSStatus SecItemUpdate(CFDictionaryRef query, CFDictionaryRef attributesToUpdate);

// 删除KeyChain中的Item
OSStatus SecItemDelete(CFDictionaryRef query)
```


这四个方法参数比较复杂，一旦传错就会导致操作`KeyChain`失败，这块儿文档中介绍的比较详细，大家可以查查官方文档[Keychain Services Reference](https://developer.apple.com/reference/security/1658642-keychain_services#//apple_ref/doc/uid/TP30000898)。


##### 三、使用KeyChain保存和获取UDID

```Objective-C

+ (BOOL)settUDIDToKeyChain:(NSString*)udid
{
    NSMutableDictionary *dictForAdd = [[NSMutableDictionary alloc] init];
    
    [dictForAdd setValue:(id)kSecClassGenericPassword forKey:(id)kSecClass];
    [dictForAdd setValue:[NSString stringWithUTF8String:kKeychainUDIDItemIdentifier] forKey:kSecAttrDescription];
    
    [dictForAdd setValue:@"UUID" forKey:(id)kSecAttrGeneric];
    
    // Default attributes for keychain item.
    [dictForAdd setObject:@"" forKey:(id)kSecAttrAccount];
    [dictForAdd setObject:@"" forKey:(id)kSecAttrLabel];
    
    // The keychain access group attribute determines if this item can be shared
    // amongst multiple apps whose code signing entitlements contain the same keychain access group.
    NSString *accessGroup = [NSString stringWithUTF8String:kKeyChainUDIDAccessGroup];
    if (accessGroup != nil)
    {
#if TARGET_IPHONE_SIMULATOR
        // Ignore the access group if running on the iPhone simulator.
        //
        // Apps that are built for the simulator aren't signed, so there's no keychain access group
        // for the simulator to check. This means that all apps can see all keychain items when run
        // on the simulator.
        //
        // If a SecItem contains an access group attribute, SecItemAdd and SecItemUpdate on the
        // simulator will return -25243 (errSecNoAccessForItem).
#else
        [dictForAdd setObject:accessGroup forKey:(id)kSecAttrAccessGroup];
#endif
    }

    const char *udidStr = [udid UTF8String];
    NSData *keyChainItemValue = [NSData dataWithBytes:udidStr length:strlen(udidStr)];
    [dictForAdd setValue:keyChainItemValue forKey:(id)kSecValueData];
    
    OSStatus writeErr = noErr;
    if ([SvUDIDTools getUDIDFromKeyChain]) {        // there is item in keychain
        [SvUDIDTools updateUDIDInKeyChain:udid];
        [dictForAdd release];
        return YES;
    }
    else {          // add item to keychain
        writeErr = SecItemAdd((CFDictionaryRef)dictForAdd, NULL);
        if (writeErr != errSecSuccess) {
            NSLog(@"Add KeyChain Item Error!!! Error Code:%ld", writeErr);
            
            [dictForAdd release];
            return NO;
        }
        else {
            NSLog(@"Add KeyChain Item Success!!!");
            [dictForAdd release];
            return YES;
        }
    }
    
    [dictForAdd release];
    return NO;
}

```


上面代码中，首先构建一个要添加到`KeyChain`中数据的`Dictionary`，包含一些基本的`KeyChain Item`的数据类型，描述，访问分组以及最重要的数据等信息，最后通过调用`SecItemAdd`方法将我们需要保存的UUID保存到`KeyChain`中。


获取`KeyChain`中相应数据的代码如下:



```Objective-C

+ (NSString*)getUDIDFromKeyChain
{
    NSMutableDictionary *dictForQuery = [[NSMutableDictionary alloc] init];
    [dictForQuery setValue:(id)kSecClassGenericPassword forKey:(id)kSecClass];
    
    // set Attr Description for query
    [dictForQuery setValue:[NSString stringWithUTF8String:kKeychainUDIDItemIdentifier]
                    forKey:kSecAttrDescription];
    
    // set Attr Identity for query
    NSData *keychainItemID = [NSData dataWithBytes:kKeychainUDIDItemIdentifier
                                            length:strlen(kKeychainUDIDItemIdentifier)];
    [dictForQuery setObject:keychainItemID forKey:(id)kSecAttrGeneric];
    
    // The keychain access group attribute determines if this item can be shared
    // amongst multiple apps whose code signing entitlements contain the same keychain access group.
    NSString *accessGroup = [NSString stringWithUTF8String:kKeyChainUDIDAccessGroup];
    if (accessGroup != nil)
    {
#if TARGET_IPHONE_SIMULATOR
        // Ignore the access group if running on the iPhone simulator.
        //
        // Apps that are built for the simulator aren't signed, so there's no keychain access group
        // for the simulator to check. This means that all apps can see all keychain items when run
        // on the simulator.
        //
        // If a SecItem contains an access group attribute, SecItemAdd and SecItemUpdate on the
        // simulator will return -25243 (errSecNoAccessForItem).
#else
        [dictForQuery setObject:accessGroup forKey:(id)kSecAttrAccessGroup];
#endif
    }
    
    [dictForQuery setValue:(id)kCFBooleanTrue forKey:(id)kSecMatchCaseInsensitive];
    [dictForQuery setValue:(id)kSecMatchLimitOne forKey:(id)kSecMatchLimit];
    [dictForQuery setValue:(id)kCFBooleanTrue forKey:(id)kSecReturnData];
    
    OSStatus queryErr   = noErr;
    NSData   *udidValue = nil;
    NSString *udid      = nil;
    queryErr = SecItemCopyMatching((CFDictionaryRef)dictForQuery, (CFTypeRef*)&udidValue);
    
    NSMutableDictionary *dict = nil;
    [dictForQuery setValue:(id)kCFBooleanTrue forKey:(id)kSecReturnAttributes];
    queryErr = SecItemCopyMatching((CFDictionaryRef)dictForQuery, (CFTypeRef*)&dict);
    
    if (queryErr == errSecItemNotFound) {
        NSLog(@"KeyChain Item: %@ not found!!!", [NSString stringWithUTF8String:kKeychainUDIDItemIdentifier]);
    }
    else if (queryErr != errSecSuccess) {
        NSLog(@"KeyChain Item query Error!!! Error code:%ld", queryErr);
    }
    if (queryErr == errSecSuccess) {
        NSLog(@"KeyChain Item: %@", udidValue);
        
        if (udidValue) {
            udid = [NSString stringWithUTF8String:udidValue.bytes];
        }
    }
    
    [dictForQuery release];
    return udid;
}

```


上面代码的流程也差不多一样，首先创建一个`Dictionary`，其中设置一下查找条件，然后通过`SecItemCopyMatching`方法获取到我们之前保存到`KeyChain`中的数据。


##### 四、总结


　　本文介绍了使用`KeyChain`实现APP删除后依然可以获取到相同的`UDID`信息的解决方法。


　　你可能有疑问，如果系统升级以后，是否仍然可以获取到之前记录的UDID数据？


　　答案是肯定的，这一点我专门做了测试。就算我们程序删除掉，系统经过升级以后再安装回来，依旧可以获取到与之前一致的UDID。但是当我们把整个系统还原以后是否还能获取到之前记录的UDID，这一点我觉得应该不行，不过手机里面数据太多，没有测试，如果大家有兴趣可以测试一下，验证一下我的猜想。


　　完整代码地址: [https://github.com/smileEvday/SvUDID](https://github.com/smileEvday/SvUDID)


　　大家如果要在真机运行时，需要替换两个地方:


　　第一个地方是plist文件中的accessGroup中的APPIdentifier。


　　第二个地方是SvUDIDTools.m中的kKeyChainUDIDAccessGroup的APPIdentity为你所使用的profile的APPIdentifier。


　　文章和代码中如果有什么不对的地方，欢迎指正，在这儿先谢过了。


原文地址：[http://www.cnblogs.com/smileEvday/p/UDID.html](http://www.cnblogs.com/smileEvday/p/UDID.html)


作者：[一片枫叶](http://www.cnblogs.com/smileEvday/)


