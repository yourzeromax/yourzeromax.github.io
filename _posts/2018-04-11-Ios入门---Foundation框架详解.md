---
layout:     post
title:      Ios入门---Foundation框架详解
subtitle:   深入理解Ios中Foundation框架
date:       2018-04-11
author:     yourzeromax
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Ios
---  
  
# 写在前面  
在最近的实习中，又接到一项任务，要改一个ios的项目（憋问我为什么一个Android攻城狮要搞ios，任性），于是在准备阶段又重新开始看了一遍Objective-c（下称oc），在温习的过程中，回想起了当时从Android开发转向学习ios开发的痛苦经历，一下又是什么基础数据类型，一下又是什么NSInteger等等；一会儿是cocoa框架、一会儿又是UIKit、一会儿又是Foundation框架...搞得自己心烦意乱，完全不知道谁是谁，所以至少在这篇文章，要给大家讲清楚Foundation框架。  

# 什么是Foundation框架  
Foundation一词，翻译成中文是基础的意思（ios中的Foundation和H5开发中的不是同一个东西哈。），顾名思义，Foundation框架是ios开发中最为重要的一个框架之一。其实理解Foundation框架并不难，只要记住它是为开发提供一些基础的类就行，具体有如下几个类：  

类型 | 所含类| 描述
---|---|---
数据存储 | NSArray、NSDictionary、NSSet| 提供存储“数组”
字符串 | NSCharacterSet、NSString| 提供字符串功能
日期和时间 | NSDate、NSCalendar、NSTimeZone| 提供时间对象
异常处理 | NSException| 处理突发情况
文件处理 | NSFileManager| 文件管理
URL网络 | NSURL等| 提供Internet访问  

是不是似曾相识？如果你有Android开发基础的，就会发现oc就是将上述功能的类剥离开形成了一个独立的框架而已，当然这个框架是默认包含在开发包里的。  
在这里说明一下，如果读者还是初学者的话，建议先不要去学习异常处理和文件处理。接下来将分别介绍每个类型所包含类的用法和功能。  

# 数据存储  

## NSArray & NSMutableArray  
NSArray是用来装不可变的对象数组，NSMutableArray用于容纳一个可变数组对象。NSMutableArray继承自NSArray，所以带Mutable字眼的都能够拥有不带字眼的实例方法（所有都是这样，不再阐述）  

**NSArray的重要方法列表：**
方法| 描述
---|---
alloc/initWithObjects | 用来初始化一个数组对象。
objectAtIndex | 在特定索引allReturns对象。
count|返回的对象的数量  

**NSMutableArray的重要方法列表：**  
方法| 描述
---|---
removeAllObjects|清空数组。
addObject|数组末尾插入一个给定的对象。
removeObjectAtIndex|这是用来删除objectAt 指定索引的对象
exchangeObjectAtIndex:withObjectAtIndex|改变阵列中的对象在给定的索引。
replaceObjectAtIndex:withObject|替换的对象与对象在索引。  

**代码示例**  
```
#import <Foundation/Foundation.h>

int main()
{
   NSArray *array = [[NSArray alloc]
   initWithObjects:@"string1", @"string2",@"string3",nil];
   NSString *string1 = [array objectAtIndex:0];
   NSLog(@"The object in array at Index 0 is %@",string1);
   NSMutableArray *mutableArray = [[NSMutableArray alloc]init];
   [mutableArray addObject: @"string"];
   string1 = [mutableArray objectAtIndex:0];
   NSLog(@"The object in mutableArray at Index 0 is %@",string1); 
   return 0;
}
```  
## NSDictionary & NSMutableDictionary 

**NSDictionary的重要方法列表：**
方法| 描述
---|---
alloc/initWithObjectsAndKeys|初始化一个新分配的字典带构建从指定的集合值和键的条目。
valueForKey|返回与给定键关联的值。
count|返回在字典中的条目的数量。

**NSMutableDictionary的重要方法列表：**  
方法| 描述
---|---
removeAllObjects|清空字典条目。
removeObjectForKey|从字典删除给定键及其关联值。
setValue:forKey|添加一个给定的键 - 值对到字典中。

**代码示例**  
```
#import <Foundation/Foundation.h>

int main()
{
   NSDictionary *dictionary = [[NSDictionary alloc] initWithObjectsAndKeys:
   @"string1",@"key1", @"string2",@"key2",@"string3",@"key3",nil];
   NSString *string1 = [dictionary objectForKey:@"key1"];
   NSLog(@"The object for key, key1 in dictionary is %@",string1);
   NSMutableDictionary *mutableDictionary = [[NSMutableDictionary alloc]init];
   [mutableDictionary setValue:@"string" forKey:@"key1"];
   string1 = [mutableDictionary objectForKey:@"key1"];
   NSLog(@"The object for key, key1 in mutableDictionary is %@",string1); 
   return 0;
}
```  
## NSSet & NSMutableSet

**NSSet的重要方法列表：**
方法| 描述
---|---
aalloc/initWithObjects|初始化一个新分配的成员采取从指定的对象列表。
allObjects |返回一个数组，包含集合的成员或一个空数组（如果该组没有成员）。
count|返回集合中的成员数量。

**NSMutableSet的重要方法列表：**  
方法| 描述
---|---
removeAllObjects|清空其所有成员的集合。
addObject|添加一个给定的对象的集合（如果它还不是成员）。
removeObject|从集合中删除给定的对象。

**代码示例**  
```
#import <Foundation/Foundation.h>

int main()
{
   NSSet *set = [[NSSet alloc]
   initWithObjects:@"string1", @"string2",@"string3",nil];
   NSArray *setArray = [set allObjects];
   NSLog(@"The objects in set are %@",setArray);
   NSMutableSet *mutableSet = [[NSMutableSet alloc]init];
   [mutableSet addObject:@"string1"];
   setArray = [mutableSet allObjects];
   NSLog(@"The objects in mutableSet are %@",setArray);
   return 0;
}
```   
# 字符串  
在这一个类别中，最为常见的就是大名鼎鼎的NSString，它的构造可以参考下例：
```
NSString *str = @"hello world"
```
使用也是非常简单的，在NSString中，也提供了一些对字符串进行处理的方法（常用）：  

方法 | 描述
---|---
- (unichar)characterAtIndex:(NSUInteger)index | 返回index下标的字符
- (Integer)integerValue;| 返回字符串转化为整型，类似的还有doubleValue等
- (BOOL)hasPrefix:(NSString *)aString|判断是否有aString这个前缀
- (BOOL)hasSuffix:(NSString *)aString|判断是否有aString这个后缀
- (NSUInteger)length|返回字符串的长度  

除此之外，还有很多方法，可以查看Xcode中的Help文档。  
  
# 日期和时间  
在这个类别中，最为重要的是NSDate和NSDateFormatter，其中NSDateFormatter是一种辅助类，能够转换NSDate和NSString。直接看代码示例即可：  
```
#import <Foundation/Foundation.h>

int main()
{
   NSDate *date= [NSDate date];
   NSDateFormatter *dateFormatter = [[NSDateFormatter alloc]init];
   [dateFormatter setDateFormat:@"yyyy-MM-dd"];
   NSString *dateString = [dateFormatter stringFromDate:date];
   NSLog(@"Current date is %@",dateString);
   NSDate *newDate = [dateFormatter dateFromString:dateString];
   NSLog(@"NewDate: %@",newDate);
   return 0;
}
```  
日期格式可以修改为@"yyyy-MM-dd:hh:mm:ss"  

# 异常处理  
在Android开发过程中，Java（或者Kotlin）提供了一套try-catch的模式来为Exception进行抛出和处理，但是在oc语言中并不存在这样的代码，类似地，oc提供了两个标签：@try和@catch、@finally。  
```
#import <Foundation/Foundation.h>

int main()
{
   NSMutableArray *array = [[NSMutableArray alloc]init];        
   @try 
   {
      NSString *string = [array objectAtIndex:10];
   }
   @catch (NSException *exception) 
   {
      NSLog(@"%@ ",exception.name);
      NSLog(@"Reason: %@ ",exception.reason);
   }
   @finally 
   {
      NSLog(@"@@finaly Always Executes");
   }
   return 0;
}
```  

# 文件处理  
在java中，处理文件提供了File类和流的概念（可参考[Java I/O流系列文档](http://yourzeromax.top/2018/04/02/Java-IO%E6%B5%81%E7%90%86%E8%A7%A3%E6%8C%87%E5%8D%97-%E4%B8%80-%E6%B5%81%E6%A6%82%E5%BF%B5%E4%B8%8EFile%E7%B1%BB/)），ios系统中处理文件使用的是NSFileManager帮助类，这也是Foundation框架中一个重要的类，一般情况下来说，一个程序中只需要一个NSFileMananger对象，可以采用单例模式。  

方法 | 描述
---|---
-(BOOL)fileExistsAtPath:(NSString) *path| 判断路径文件是否存在
-(BOOL)contentsEqualAtPath:@"FilePath1" andPath:@" FilePath2"| 比较两个文件内容是否相同
-(BOOL)isWritableFileAtPath:@"FilePath"|是否可写
-(BOOL)isReadableFileAtPath:@"FilePath"|是否可读
-(BOOL)moveItemAtPath:@"FilePath1" toPath:@"FilePath2" error:NULL|移动文件
-(BOOL)copyItemAtPath:@"FilePath1" toPath:@"FilePath2" error:NULL|复制文件
-(BOOL)removeItemAtPath:@"FilePath" error:NULL|删除文件
-(NSData)fileManager contentsAtPath:@"Path"|读取文件
-(void)createFileAtPath:@"" contents:data attributes:nil|写入文件  

ios系统中对文件的操作都是由NSFileManager来进行的，请注意其中的NSDate，会在之后的文件文章中具体说明。  

# URL网络
Foundation框架提供的URL网络访问相关的类非常多：  
- NSMutableURLRequest
- NSURLConnection
- NSURLCache
- NSURLAuthenticationChallenge
- NSURLCredential
- NSURLProtectionSpace
- NSURLResponse
- NSURLDownload  

我个人认为不太需要注重这个原生Foundation提供的类和方法，因为现在的ios项目基本都用的很成熟的第三方开源库，比如ASIHttpRequest、AFNetworking等等，[建议阅读这篇文章](https://www.jianshu.com/p/ab246881efa9)，对了，ios开发集成第三方非常麻烦...需要使用cocoaPods，在国内被墙的大环境下，更新一次是真的恼火，如果你需要梯子的话，[可以参阅本文](http://yourzeromax.top/2018/03/05/%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E6%90%AD%E5%BB%BA%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91%E7%8E%AF%E5%A2%83(%E5%B8%A6%E4%BC%AA%E7%A6%8F%E5%88%A9)/)

# 写在后面
看完这篇文章，是不是觉得Foundation框架原来不过如此？如果你有疑问的话，欢迎留言或者发送电子邮件。

[个人主页](http://yourzeromax.top/)
