---
title: Objective-C代码规范
date: 2016-04-08 13:13:04
categories:
    - tech
tags:
    - iOS
    - Objective-C
---
团队成员不断增加，为了代码的可读性，简单做了下代码规范。

### 基本
* 将Tab制表符设置为4个空格
* 每行长度最好能控制在80左右（非强制）

### 条件语句
#### 必要的地方请使用断言,不要打印日志
```objective-c
NSAssert(condition, desc, ...)
NSCAssert(condition, desc, ...)
NSParameterAssert(condition)
NSCParameterAssert(condition)
```

#### 不推荐犹达表达式（除非你写c/c++）
```objective-c
if (count == 5) //推荐

if (5 == count) //不推荐
```

#### 条件分支语句换行？
```objective-c
//不换行
if (count == 5) {
    ...
} else {
    ...
}

//换行
if (count == 5) {
    ...
} 
else {
    ...
}

```

#### nil和BOOL的检查
* 不要讲BOOL类型变量直接与YES/NO比较  

```objective-c
BOOL great = [foo isGreat];

if (great) //推荐

if (great == YES) //不推荐

```

* 检查对象是否为nil时，不用显示与nil比较
注：swift是强制要求返回布尔值的或使用let语句

```objective-c
NSObject *object = [[NSObject alloc] init];

if (object) //推荐

if (object != nil) //不推荐

if (!object) //推荐

if (object == nil) //不推荐

```

#### 方法中的重要部分不要嵌套在判断分支中
注：类似swift中的guard语句
```objective-c
//推荐
 - (void)someMethod {
    if (![someOther boolValue]) {
        return;
    }
    
    //Do something ...
}

//不推荐
- (void)someMethod {
    if ([someOther boolValue]) {
        //Do something ...
    }
}
```

#### 复杂表达式
```objective-c
//推荐：使用子句,在必要情况下在子句后添加注释
BOOL nameContainsSwift = [sessionName containsString:@"Swift"];
BOOL isCurrentYear = [sessionDateCompontents year] == 2014;
BOOL isSwiftSession = nameContainsSwift && isCurrentYear;
if (isSwiftSession) {
    // Do something very cool
}

//不推荐：将复杂的判断语句写在if后面

```

#### 三元运算符
* 不要嵌套使用
* 当三元运算符的第二个参数返回和条件语句中已检查的对象一样的对象的时候

```objective-c
result = object ? : [self createObject]; //推荐
result = object ？ object : [self createObject]; //不推荐 
```

#### 处理错误
* 有些方法通过参数返回error的引用。使用这样的方法时应当检查方法的返回值，而不是error的引用。（一些苹果的API在成功的情况下会对error参数写入垃圾值，所以直接检查error的引用会出不可预测的问题，不过还未找到证据）

```objective-c
NSError *error = nil;

//推荐
if (![self trySomethingWithError:&error]) {
    //Handle Error
}

//不推荐
id objct = [self trySomethingWithError:&error];

if (error) {
    //Handle Error
}
```
#####switch-case
* 除非编译器强制要求，括号在case语句里面是不必要的，但是当一个case包含该多行语句时，需要括号。
* 当在switch语句里面使用一个可枚举的变量的时候，default是不必要的

```objective-c
switch (condition) {
    case 1:
    case 2:// code executed for values 1 and 2
        break;
    case 3: {
        //multi-line
          break;
      }
    default:// ...
        break;
}

        switch (type) {
            case MiddleHeaderTypeNone:
                //...
                break;
            case MiddleHeaderTypeWaitPay:
                //...
                break;
            case MiddleHeaderTypeWaitSure:
                //...
                break;
            case MiddleHeaderTypecontactShop:
                //...
                break;
            case MiddleHeaderTypeOrderCode:
                //...
                break;
            case MiddleHeaderTypeReSubmit:
                //...
                break;
        }

```

### 命名
总要求：
```objective-c
alloc   == Allocate                 max    == Maximum
alt     == Alternate                min    == Minimum
app     == Application              msg    == Message
calc    == Calculate                nib    == Interface Builder archive
dealloc == Deallocate               pboard == Pasteboard
func    == Function                 rect   == Rectangle
horiz   == Horizontal               Rep    == Representation (used in class name such as NSBitmapImageRep).
info    == Information              temp   == Temporary
init    == Initialize               vert   == Vertical
int     == Integer
```

* 清晰，不要用单词简写除了一些通用的简写，如上，不要用拼音及单个字母等奇葩命名
* 不要有歧义，尽量做到代码自解释
* 命名一致性，不要一会count，一会getNumber
* 使用前戳？

#### 常量
注：除了必要的地方使用宏，不然请尽量使用常量，好处：
1. 宏定义的常量没有常量的类型信息；
2. 在头文件中使用宏定义常量，会导致引入该头文件的其他文件都会出现这个名字；
3. 即使重复宏定义了常量值，编译器也不会发出警告，导致程序中的常量值不一致； 
4.驼峰

```objective-c
//不需要暴露给外部使用，则在实现文件中定义
static NSString *const theConstString = @"the Const String"

//暴露给外部使用，则在头文件声明，实现文件赋值
//.h
extern NSString *const AppDidLauchOptionsShortcutToScanCodeNotification; 

//.m
NSString *const AppDidLauchOptionsShortcutToScanCodeNotification =
@"appDidLauchOptionsShortcutToScanCodeNotification";
```

#### 方法
```objective-c
- (void)short:(GTMFoo *)theFoo
        longKeyword:(NSRect)theRect
  evenLongerKeyword:(float)theInterval
              error:(NSError **)theError {
    ...
}
```
* `-`和`（void）`直接必须有一个空格
* `{`紧接着方法名，有一个空格
* 如果参数或名称很长需要分行
* 在分行时，如果第一段名称过短，后续名称可以以Tab的长度（4个空格）为单位进行缩进
* 方法名中尽量不要出现and

#### 字面值
* 使用字面值来创建不可变的 NSString , NSDictionary , NSArray ,  NSNumber 对像
*  NSDictionary , NSArray 如果构造代码不写在一行内，构造元素需要使用两个空格来进行缩进，右括号]或者}写在新的一行，并且与调用语法糖那行代码的第一个非空字符对齐

```objective-c
//正确，在语法糖的"[]"或者"{}"两端留有空格
NSArray *array = @[ [foo description], @"Another String", [bar description] ];
NSDictionary *dict = @{ NSForegroundColorAttributeName : [NSColor redColor] };

NSDictionary *dic = @{
  @"key1" : @"value1",
  @"key2" : @"value1",
  @"key3" : @"value1",
  @"key4" : @"value1"
};
    
NSArray *array = @[
  @"This",
  @"is",
  @"an",
  @"array"
];
```
#### 枚举
* 推荐使用NS_ENUM和NS_OPTIONS定义枚举
* 能用枚举解决的问题，不要用宏

```objective-c
typedef NS_ENUM(NSInteger, NSMatrixMode) {   
    NSRadioModeMatrix,
    NSHighlightModeMatrix,
    NSListModeMatrix, 
    NSTrackModeMatrix
};

typedef NS_OPTIONS(NSUInteger, NSWindowMask) {   
    NSBorderlessWindowMask = 0,
    NSTitledWindowMask = 1 << 0, 
    NSClosableWindowMask = 1 << 1, 
    NSMiniaturizableWindowMask = 1 << 2, 
    NSResizableWindowMask = 1 << 3
};
```

#### cocoa
* 命名尽量清楚，看到名称就能知道他的类型，主要是一些UI组件

```objective-c
//推荐
UILabel *nameLabel = [[UILabel alloc] init];

//不推荐
UILabel *name = [[UILabel alloc] init];
UILabel *nameLab = [[UILabel alloc] init];

```

### 类
#### Pragma
```
//分代码块
#pragma mark - View Lifecycle
#pragma mark - Public Methods
#pragma mark - Other Methods
#pragma mark - Touch Methods
#pragma mark - Notification Methods
#pragma mark - Request Methods
#pragma mark - Delegate Methods
#pragma mark - Override SuperClass Methods
#pragma mark - Getters and Setters
```
* 其它作用

```objective-c
//忽略警告
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"

[myObj performSelector:mySelector withObject:name];

#pragma clang diagnostic pop

//忽略没有使用变量的编译警告
- (void)giveMeFive
{
    NSString *foo;
    #pragma unused (foo)

    return 5;
}
```

#### 属性
* 命名清晰
* `*`位置

```objective-c
//推荐
NSString *text;

//不推荐
NSString* text; //c++程序员 喜欢使用这种写法
NSString * text;
```

* 成员变量和属性的选择？
尽量使用属性，并且使用getter和setter访问 

```
//好处
1.使用 setter 会遵守定义的内存管理语义(strong, weak, copy，etc...) 这回定义更多相关的在ARC是钱，因为它始终是相关的个例子，copy每个时候你用 setter 并且传送数据的时候，它会复制数据而不用额外的操作
2.KVO 通知(willChangeValueForKey, didChangeValueForKey) 会被自动执行
3.更容易debug：你可以设置一个断点在属性声明上并且断点会每次 getter / setter 方法调用的时候执行，或者你可以在自己的自定义 setter/getter 设置断点。
4.允许在一个单独的地方为设置值添加额外的逻辑。

//getter访问的其他好处
1.它是对未来的变化有扩展能力的（比如，属性是自动生成的）
2.它允许子类化
3.更简单的debug（比如，允许拿出一个断点在 getter 方法里面，并且看谁访问了特别的 getter
4.它让意图更加清晰和明确：通过访问 ivar _anIvar
 你可以明确的访问 self->_anIvar
.这可能导致问题。在 block 里面访问 ivar （你捕捉并且 retain 了 sefl 即使你没有明确的看到 self 关键词）
5.它自动产生KVO 通知
6.在消息发送的时候增加的开销是微不足道的。
```

* 属性定义

```objective-c
//1.相同类型的属性定义，不推荐写在同一行
@property (nonatomic, assign) double allFee, radioFee,redPacketPrice, hasPayPrice, needPayPrice ,originPrice ,originServiceCharge;

//2.顺序为：原子性，读写，内存管理
@property (nonatomic, readwrite, copy) NSString *name;

//3.提供一个公用的getter（内部、外部）和一个私有的setter（只能内部设置）
@interface MyClass : NSObject
@property (nonatomic, readonly) NSObject *object
@end

@implementation MyClass ()
@property (nonatomic, readwrite, strong) NSObject *object
@end

//4.需要暴露的属性写在.h,私有的属性写在.m

//5. 布尔类型的属性
//推荐
@propert y(nonatomic,getter=isEditing) BOOL editing; 

//不推荐
@propert y(nonatomic, assign) BOOL isEditing;
```

* 可变对象与不可变对象 NSString、NSArray、NSDictionary等

```objective-c
//1.必须使用copy修饰
@property (nonatomic,copy) NSArray *elements

//2.最好不要直接将可变对象直接暴露出来，而是专门定义一个不可变对象，重写getter方法，暴露出来

/* .h */
@property (nonatomic, readonly, copy) NSArray *elements

/* .m */
@property (nonatomic, strong) NSMutilArray *mutableElements

- (NSArray *)elements {
  return [self.mutableElements copy];
}

```

* 懒加载（如果有些地方不使用懒加载，需要创建对象，请使用代码块）
```objective-c
//一个 GCC 非常模糊的特性，以及 Clang 也有的特性是，代码块如果在闭合的圆括号内的话，会返回最后语句的值
NSURL *url = ({
      NSString *urlString = [NSString stringWithFormat:@"%@/%@", baseURLString, endpoint];
      [NSURL URLWithString:urlString];
});
```

#### 不要使用new，请使用alloc、init
* 更清晰，没什么好说的

#### 初始化方法中，属性赋值不要出现self
```objective-c
//推荐
- (instancetype)init {
    if (self = [super init]) {
        _name = @"name"
    }
    return self;
}

//不推荐
- (instancetype)init {
    if (self = [super init]) {
        self.name = @"name"
    }
    return self;
}
```

### Categories（分类）
* 分类方法使用Apple推荐的方式？小写的前戳和下划线开头

```objective-c
//推荐
@interface NSDate (ZOCTimeExtensions)
- (NSString *)zoc_timeAgoShort;
@end

//不推荐
@interface NSDate (ZOCTimeExtensions)
- (NSString *)zoc_timeAgoShort;
@end
```
### Protocols
注：Apple （Objective-C）几乎只是在委托模式下使用 protocol，但是在swift中面向协议编程被广泛推荐。

* <>括起来的协议和类型名之间是没有空格的

```objective-c
@interface MyProtocoledClass : NSObject<NSWindowDelegate>{
 @private
     id<MyFancyDelegate> _delegate;
}
- (void)setDelegate:(id<MyFancyDelegate>)aDelegate;
@end
```

* 定义时显示指明@optional或@required

```objective-c
@protocol UITableViewDataSource<NSObject>

@required

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section;

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath;

@optional

- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView; 

...

@end
```

### NSNotification
消息名字的定义，千万不要用宏，请使用常量
```objective-c
// Foo.h
extern NSString * const ZOCFooDidBecomeBarNotification

// Foo.m
NSString * const ZOCFooDidBecomeBarNotification = @"ZOCFooDidBecomeBarNotification";
```

### Block
```objective-c
//较短的block写在一行内
[operation setCompletionBlock:^{ [self onOperationDone]; }];


//分行书写的block，内部使用4空格缩进
[operation setCompletionBlock:^{
    [self.delegate newDataAvailable];
}];

//较长的block关键字可以缩进后在新行书写，注意block的右括号'}'和调用block那行代码的第一个非空字符对齐
[[SessionService sharedService]
    loadWindowWithCompletionBlock:^(SessionWindow *window) {
        if (window) {
          [self windowDidLoad:window];
        } else {
          [self errorLoadingWindow];
        }
    }];
    
    
//庞大的block应该单独定义成变量使用 或者 单独实现一个方法，在block中调用
void (^largeBlock)(void) = ^{
    // ...
};
[_operationQueue addOperationWithBlock:largeBlock];


//在一个调用中使用多个block，注意到他们不是像函数那样通过':'对齐的，而是同时进行了4个空格的缩进
[myObject doSomethingWith:arg1
    firstBlock:^(Foo *a) {
        // ...
    }
    secondBlock:^(Bar *b) {
        // ...
    }];
```

### 注释
注：可以使用onev的VVDocumenter插件
* 所有暴露给外部使用的属性接口，需要写注释
* 格式

```objective-c
//比较短的注释，一行可以解决的
// Return a user-readable form of a Frobnozz, html escaped.

//比较长的注释，需要多行的
/**
 This comment serves to demonstrate the format of a docstring.

 Note that the summary line is always at most one line long, and
 after the opening block comment, and each line of text is preceded
 by a single space.
*/
```
参考：
1.[https://github.com/objc-zen/objc-zen-book](https://github.com/objc-zen/objc-zen-book)  
2.[http://www.jianshu.com/p/c150f8fa4d21](http://www.jianshu.com/p/c150f8fa4d21)
3.[https://github.com/QianKaiLu/Objective-C-Coding-Guidelines-In-Chinese](https://github.com/QianKaiLu/Objective-C-Coding-Guidelines-In-Chinese)