封装自定义打印宏并在预编译文件中引用

```Objective-C
#ifdef DEBUG
#define MyLog(FORMAT, ...) fprintf(stderr,"%s: 第%d行\t%s\n",[[[NSString stringWithUTF8String:__FILE__] lastPathComponent] UTF8String], __LINE__, [[NSString stringWithFormat:FORMAT, ##__VA_ARGS__] UTF8String]);

#else
#define MyLog(...)
#endif
```
『在 Debug 模式下打印出类名、行数、打印信息』

如图：![](https://github.com/CalvinCheungCoder/Blog/blob/master/Objective-C/mylog.jpeg)
