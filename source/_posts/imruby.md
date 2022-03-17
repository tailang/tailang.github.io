---
title: iMRuby:在iOS上更方便运行Ruby
date: 2021-02-15 16:42:00
categories:
    - tech
tags:
    - ruby
    - iOS
---
![imruby](/images/imruby_logo.png)  
感谢小鱼章章同学的logo设计
iMRuby是一个运行于iOS的framework，作为ObjC与Ruby（MRuby）互相调用的桥接层。API设计上尽量向JavaScriptCore framework对齐，如JavaScriptCore framework有JSContext、JSValue，iMRuby提供了MRBcontext，MRBValue，所以使用上非常相似。  

接下来简单分享下iMRuby的使用，先抛一下项目地址：  
MRubyFramework：https://github.com/tailang/MRubyFramework  
iMRuby：https://github.com/tailang/iMRuby  

注：如何将mruby移植到iOS上，可以参考MRubyFramework项目的Rakefile和build_config.rb  

### Demo
先来感受下iMRuby，下图是使用ruby编写的一个View demo，demo代码如下  
![iOSdemo](/images/imruby_demo.png)

```objc

// ObjC
// MRBViewController.m
#import "MRBViewController.h"
@import iMRuby;

@interface MRBViewController ()

@property (nonatomic, strong) MRBContext *context;

@end

@implementation MRBViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    NSString *scriptPath = [[NSBundle mainBundle] pathForResource:@"view" ofType:@"rb"];
    NSString *script = [NSString stringWithContentsOfFile:scriptPath encoding:NSUTF8StringEncoding error:nil];
    self.context = [[MRBContext alloc] init];
    self.context.exceptionHandler = ^(NSError * _Nonnull exception) {
        NSLog(@"%@", exception.userInfo[@"msg"]);
    };
    // 加载执行view.rb ruby代码
    [self.context evaluateScript:script];
    // 调用view.rb 中create_view方法，并将self.view作为参数
    MRBValue *superView = [MRBValue valueWithObject:self.view inContext:self.context];
    [self.context callFunc:@"create_view" args:@[superView]];
}

@end
```

```ruby
# Ruby
# view.rb
require_cocoa "UIColor"
require_cocoa "UIView"

def create_view(super_view)
    red = UIColor.redColor;
    blue = UIColor.blueColor;
    green = UIColor.greenColor;
    
    super_view.setBackgroundColor_(red)
    
    blue_sub_view = UIView.alloc.init
    blue_sub_view.setFrame_({'x' => 40, 'y' => 200, 'width' => 100, 'height' => 100})
    blue_sub_view.setBackgroundColor_(blue)
    super_view.addSubview_(blue_sub_view)
    
    green_sub_view = UIView.alloc.init
    green_sub_view.setFrame_({'x' => 180, 'y' => 200, 'width' => 100, 'height' => 100})
    green_sub_view.setBackgroundColor_(green)
    super_view.addSubview_(green_sub_view)
end
```
   
### MRBContex
类似JSContext，MRBContext是运行Ruby代码的环境，底层是对mruby 的mrb_state的封装。
```objc

// 主要api
// ruby异常的捕获handler
@property (nonatomic, copy, nullable) void (^exceptionHandler)(NSError *exception);

// 直接执行ruby代码脚本
- (MRBValue *)evaluateScript:(NSString *)script;
// 直接调用context中定义的ruby方法
- (MRBValue *)callFunc:(NSString *)funcName args:(NSArray <MRBValue *> *)args;
// 将block作为方法实现，注册到context给ruby调用
- (BOOL)registerFunc:(NSString *)funcName block:(id)block;
// 将value注册到context作为一个ruby const变量给ruby调用
- (BOOL)registerConst:(NSString *)constName value:(id)value;

// 下标语法的支持
- (MRBValue *)objectForKeyedSubscript:(NSString *)key;
- (void)setObject:(id)object forKeyedSubscript:(NSString *)key;
```

### MRBValue
类似JSValue，MRBValue提供了一系列两端值转换方法。  
![type](/images/imruby_type.png)  
上图是ObjC type与ruby type的转换映射图，暂时只支持这些类型的转换。  
除了CGPoint、CGSize、CGRect、NSRange这几个struct会被映射成ruby特殊的Hash对象，其它struct暂不支持。  
NSDate会被映射成ruby中的Time对象，原理是通过时间戳实现。  

Block、id、Class分别会被映射成Ruby的MRBCocoa::Block、MRBCocoa::Object、MRBCocoa::Klass对象，具体实现可参考项目MRBWrapperData目录下的实现，除此之外ruby的Proc对象也可以通过to_cocoa_block(return type, params type...)转换成MRBCocoa::Block传递给ObjC使用,原理是通过ffi动态生成c函数实现（参考JPBlock）。  

### MRBExport（已移除）

之前参考了JSExport Protocol的设计，实现了MRBExport去暴露方法提供给ruby使用，但后来发现这样设计ruby调用ObjC方法约束太大，所以暂时移除了该设计实现，取而代之是约定的方式将ObjC方法全部暴露给ruby使用。  

### 内存管理

ObjC在ARC模式下是通过自动引用计数实现对象回收的，而ruby是通过GC（垃圾回收）机制实现内存管理的，所以在桥接中势必会引起内存管理问题，现在iMRuby只是做了很简单的处理，在ObjC对象被使用转换成mrb_value(MRBValue)后做了retain，当mrb_value被ruby GC回收后做了release，这样保证了ruby在使用ObjC对象时不被提前释放的问题，但这样带来的问题是：会引起ObjC对象的生命周期被延长（取决于ruby 何时GC）。  


### 一些小🌰及用法

直接运行一段ruby脚本：
```objc

[self.context evaluateScript:@"puts \"Happy Niu year!\""];
# => Happy Niu year!
```

#### 捕获ruby脚本异常
```objc
self.context.exceptionHandler = ^(NSError * _Nonnull exception) {
        NSLog(@"%@", exception.userInfo[@"msg"]);
    };
[self.context evaluateScript:@"happy_nui_year(2021)"];
# => undefined method 'happy_niu_year' (NOMethodError)
```

#### context注册常量给ruby使用

```objc
// 常量会被注册到MRBCocoa::Const module下
[self.context registerConst:@"Niu" value:@"Happy Niu year!"];
[self.context evaluateScript:@"puts MRBCocoa::Const::Niu"];
# => Happy Niu year!
```

#### context注册方法给ruby调用
```objc

// happy_niu_year方法会被注册到MRBCocoa module下
[self.context registerFunc:@"happy_niu_year" block:^NSInteger(int a, int b){
    NSLog(@"start...");
    int sum = a + b;
    NSLog(@"finish...");
    return sum;
 }];
 MRBValue *sum = [self.context evaluateScript:@"MRBCocoa.happy_niu_year(2020,1)"];
 NSInteger niu_year = sum.toInt64;
 NSLog(@"%@", @(niu_year));
# => start...
# => finish...
# => 2021
```

#### 下标方式注册常量或方法
```objc

// 下标注册对象如果是Block默认注册是方法，其它对象注册成常量
self.context[@"Niu"] = @"Happy Niu year!";
self.context[@"happy_niu_year"] = ^NSInteger(int a, int b) {
     NSLog(@"start...");
     int sum = a + b;
     NSLog(@"finish...");
    return sum;
};
MRBValue *sum = [self.context evaluateScript:@"puts MRBCocoa::Const::Niu;MRBCocoa.happy_niu_year(2020,1)"];
NSInteger niu_year = sum.toInt64;
NSLog(@"%@", @(niu_year));
```

#### ruby调用ObjC对象方法

上面在MRBExport中提到，iMRuby移除了export，而是用约定的方式暴露所用的方法可被调用。
```objc
// 先定义个Person类
@interface Person : NSObject
@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) int age;
- (NSString *)say_something:(NSString *)message;
- (void)coding:(NSString *)code finished:(BOOL(^)(NSString *name, int age))finished;
@end
@implementation Person
- (NSString *)say_something:(NSString *)message
{
    NSLog(@"%@", message);
    return [self.name stringByAppendingString:message];
}
- (void)coding:(NSString *)code finished:(BOOL(^)(NSString *name, int age))finished
{
    NSLog(@"I am coding %@", code);
    if (finished) {
        BOOL f = finished(self.name, self.age);
        NSLog(@"Block return value: %@", @(f));
    }
}
@end
```

```ruby

# 编写Ruby脚本demo.rb
require_cocoa 'Person'

person = Person.alloc.init
person.setName_('anan')
person.setAge_(2)
message = person.say__something_("happy Niu year!")
puts message

finished = Proc.new {|name, age| puts "I am #{name}, #{age} year old"; true}
finished_block = finished.to_cocoa_block("BOOL, NSString *, int");
person.coding_finished_("Ruby", finished_block)

```

```objc

// 运行
 NSString *demoScriptPath = [[NSBundle mainBundle] pathForResource:@"demo" ofType:@"rb"];
 NSString *demoScript = [NSString stringWithContentsOfFile:demoScriptPath encoding:NSUTF8StringEncoding error:nil];
 [self.context evaluateScript:demoScript];
 
 // log
 // happy Niu year!
 // ananhappy Niu year!
 // I am coding Ruby \n I am anan, 2 year old
 // Block return value: 1
```

`require_cocoa 'Person' ：require_cocoa` 其实是一个自定义在Kernel module中方法，该方法接收一个字符串参数，通过字符串获取对应的ObjC class对象，然后再将对象转换成MRBCocoa::Klass对象注册为常量到kernel module，这样ruby就能使用这个类对象了。



#### 方法调用规则

1、遇到 ObjC方法中`:`，需要将其转换成`_`;

eg：
```objc
- (NSString *)saySomething:(NSString *)message;
message = person.saySomething_("happy Niu year!")
```
2、如果ObjC方法命名中间已经有`_`,需要使用两个`_`替换，`__`;
eg:
```objc
- (NSString *)say_something:(NSString *)message;
message = person.say__something_("happy Niu year!")
```
3、如果ObjC方法命名开头已经有`_`或`__`等，保持原样，不需要添加多余的"_";

eg:
```objc
- (NSString *)__say_something:(NSString *)message;
message = person.__say__something_("happy Niu year!")
```
4、如果ObjC方法参数中有Block，ruby可以创建Proc对象，然后通过`to_cocoa_block`方法转换成MRBCocoa::Block使用。

eg:
```objc
- (void)coding:(NSString *)code finished:(BOOL(^)(NSString *name, int age))finished;

// ruby
finished = Proc.new {|name, age| puts "I am #{name}, #{age} year old"; true}
finished_block = finished.to_cocoa_block("BOOL, NSString *, int");
person.coding_finished_("Ruby", finished_block)
```

### 结语
当前iMRuby只是一个个人玩具，没有在真实项目中使用过，~~甚至连单元测试都还没加上（未来会补上）~~，肯定会有很多问题，欢迎一起交流学习。