< [Back](README.md)

Objective-C 编码规范
======================

<a name='TOC'/></a>目录
----
* [代码组织](#code-organization)
  * [代码分类](#code-assoeted)
  * [构造方法](#code-init)
  * [头文件导入](#code-import)
  * [黄金路径](#code-way)
  * [错误返回](#code-error)
* [代码格式化](#formatting)
  * [空格](#spaces)
  * [花括号](#braces)
  * [布尔值](#bool)
  * [条件语句](#ifelse)
  * [点符号语法](#point)
* [属性](#attribute)
  * [私有属性](#attribute-private)
  * [属性书写规范](#attribute-norm)
* [命名](#naming)
  * [基本原则](#naming-basic-principle)
  * [前缀](#prefix)
  * [方法名](#naming-method)
  * [协议名](#naming-protocol)
  * [通知命名](#naming-notifications)
  * [临时变量命名](#naming-temporary-variable)
  * [常量命名](#naming-constant)
  * [大小写](#naming-match-case)
  * [缩写](#abbreviation)
  * [视图命名](#naming-viewcontroller)
  * [其他](#naming-others)
* [常量使用](#constant)
* [枚举类型](#enum)
* [注释](#comment)
* [其他](#others)
  * [异常](#exception)

<a name='code-organization'></a>代码组织
----
* 函数长度（行数）不应超过2/3屏幕，禁止超过70行。
* 单个文件总长度不能超过500行
* 按类别排序（如把IBAction放在一块）
* 尽量不要出现超过两层循环的代码，可以用函数或block替代。
* .h文件只暴露别人需要用的属性和方法，其他的都放在.m文件中

### <a name='code-assoeted'></a>代码分类
* 在函数分组和protocol/delegate实现中使用#pragma mark -来分类方法
```C
#pragma mark - Lifecycle

- (instancetype)init {}
- (void)dealloc {}
- (void)viewDidLoad {}
- (void)viewWillAppear:(BOOL)animated {}
- (void)didReceiveMemoryWarning {}

#pragma mark - Custom Accessors

- (void)setCustomProperty:(id)value {}
- (id)customProperty {}

#pragma mark - IBActions

- (IBAction)submitData:(id)sender {}

#pragma mark - Public

- (void)publicMethod {}

#pragma mark - Private

- (void)privateMethod {}
```
### <a name='code-init'></a>构造方法
* 构造方法必须用instancetype返回值，不能用id
```C
// OK
- (instancetype)initWithAppraiseVc:(YCAppraiseHomeController *)vc;
+ (instancetype)appraiseResultRequestWithAppraiseVc:(YCAppraiseHomeController *)vc;
// NO
- (id)initWithAppraiseVc:(YCAppraiseHomeController *)vc;
+ (id)appraiseResultRequestWithAppraiseVc:(YCAppraiseHomeController *)vc;
```
### <a name='code-import'></a>头文件导入
* .h文件用不要导入其他头文件，需要时用@class替代
```C
// 错误
#import "YCResultModel.h"

@interface YCResultCellTool : NSObject

+ (CGFloat)resultCellHeightForRowAtIndexPath:(NSIndexPath *)indexPath model:(YCResultModel *)model;

@end
// OK
@class YCResultModel;

@interface YCResultCellTool : NSObject

+ (CGFloat)resultCellHeightForRowAtIndexPath:(NSIndexPath *)indexPath model:(YCResultModel *)model;

@end
 // 执行代码
```
### <a name='code-way'></a>黄金路径
* 黄金路径
```C
// 错误
if ([path isURL]) {
    // 执行代码
}
// OK
if (![path isURL]) {
   return;
}
 // 执行代码
```
### <a name='code-error'></a>错误返回
* 尽早返回错误
```C
// 为了简化示例，没有错误处理，并使用了伪代码

// 糟糕的例子
- (Task *)creatTaskWithPath:(NSString *)path {
    Task *aTask;
    if ([path isURL]) {
        if ([fileManager isWritableFileAtPath:path]) {
            if (![taskManager hasTaskWithPath:path]) {
                aTask = [[Task alloc] initWithPath:path];
            }
            else {
                return nil;
            }
        }
        else {
            return nil;
        }
    }
    else {
        return nil;
    }
    return aTask;
}

// 改写的例子
- (Task *)creatTaskWithPath:(NSString *)path {
    if (![path isURL]) {
        return nil;
    }
    
    if (![fileManager isWritableFileAtPath:path]) {
        return nil;
    }
    
    if ([taskManager hasTaskWithPath:path]) {
        return nil;
    }
    
    Task *aTask = [[Task alloc] initWithPath:path];
    return aTask;
}
```

<a name='formatting'/></a>代码格式化
-----
### <a name='spaces'></a>空格
* 方法声明在方法类型与返回类型之间要有空格。

```Objective-C
// 糟糕
-(void)methodName:(NSString *)string;

// OK
- (void)methodName:(NSString *)string;
```

* 条件判断的括号内侧不应有空格。

```C
// 糟糕
if ( a < b ) {
    // something
}

// OK
if (a < b) {
    // something
}
```

* 关系运算符（如 `>=`、`!=`）和逻辑运算符（如 `&&`、`||`）两边要有空格。

```C
// OK
(someValue > 100)? YES : NO

// OK
(items)?: @[]
```

* 二元算数运算符两侧是否加空格不确定，根据情况自己定。一元运算符与操作数之前没有空格。

* 多个参数逗号后留一个空格（这也符合正常的西文语法）。


### <a name='braces'></a>花括号
* 花括号不要另起一行。

```Objective-C
  - (void)methodName:(NSString *)string {
   ↑空格                      ↑空格    ↑空格，推荐花括号在一行
      if () {
   空格↑  ↑空格，花括号不要另起一行
      }
      else {
要换行↑   ↑空格，花括号不要另起一行
      }    
  }
```

### <a name='bool'></a>布尔值
* 既然nil解析成NO，所以没有必要在条件语句比较。不要拿某样东西直接与YES比较，因为YES被定义为1和一个BOOL能被设置为8位。
```
// OK
if (someObject) {}
if (![anotherObject boolValue]) {}
// NO
if (someObject == nil) {}
if ([anotherObject boolValue] == NO) {}
if (isAwesome == YES) {} // Never do this.
if (isAwesome == true) {} // Never do this.
```

* 如果BOOL属性的名字是一个形容词，属性就能忽略"is"前缀，但要指定get访问器的惯用名称。例如：
```
// OK
@property (assign, getter=isEditable) BOOL editable;
```

### <a name='ifelse'></a>条件语句
* 条件语句主体为了防止出错应该使用大括号包围，即使条件语句主体能够不用大括号编写(如，只用一行代码)。这些错误包括添加第二行代码和期望它成为if语句；还有，even more dangerous defect可能发生在if语句里面一行代码被注释了，然后下一行代码不知不觉地成为if语句的一部分。除此之外，这种风格与其他条件语句的风格保持一致，所以更加容易阅读。
```
// OK
if (!error) {
  return success;
}
或
if (!error) return success;
// NO
if (!error)
  return success;
```
### <a name='point'></a>点符号语法
* 点语法是一种很方便封装访问方法调用的方式。当你使用点语法时，通过使用getter或setter方法，属性仍然被访问或修改。
* 点语法应该总是被用来访问和修改属性，因为它使代码更加简洁。[]符号更偏向于用在其他例子。
```
// OK
NSInteger arrayCount = self.array.count;
view.backgroundColor = [UIColor orangeColor];
[UIApplication sharedApplication].delegate;
// NO
NSInteger arrayCount = [self.array count];
[view setBackgroundColor:[UIColor orangeColor]];
[[UIApplication sharedApplication] delegate];
```

<a name='attribute'></a>属性
----
### <a name='attribute-private'></a>私有属性
* 私有属性写在匿名分类中，不要暴漏在.h文件中
```C
@interface RWTDetailViewController ()

@property (strong, nonatomic) GADBannerView *googleAdView;
@property (strong, nonatomic) ADBannerView *iAdView;
@property (strong, nonatomic) UIWebView *adXWebView;

@end
```

### <a name='attribute-norm'></a>属性书写规范
```C
// OK
@interface RWTTutorial ()

@property (strong, nonatomic) NSString *tutorialName;

@end
// 糟糕
@interface RWTTutorial () {
  NSString *tutorialName;
}
```
* 所有属性特性应该显式地列出来，有助于新手阅读代码。属性特性的顺序应该是storage、atomicity，与在Interface Builder连接UI元素时自动生成代码一致。
```C
// OK
@property (weak, nonatomic) IBOutlet UIView *containerView;
@property (strong, nonatomic) NSString *tutorialName;
// NO
@property (nonatomic, weak) IBOutlet UIView *containerView;
@property (nonatomic) NSString *tutorialName;
```

<a name='naming'/></a>命名
-----
### <a name='naming-basic-principle'></a>基本原则

* 仿照 Cocoa 风格来，使用长命名风格
* 变量命名推荐的命名语素顺序是：最开头是命名空间简写，然后越重要、区别度越大的语素越要往前放。经典的结构是：作用范围+限定修饰+类型。例：

```C
extern ushort APIDefaultPageSize;        // 还行，能明白意思了
extern ushort APIDefaultFetchPageSize;   // 加上些限定更好一些
extern ushort APIFetchPageSizeDefault;   // 再好些，把重要的往前放

YHToolbarComment    // 不推荐
YHCommentToolbar    // OK，把类型（toolbar）置后
```

<a name='prefix'/></a>前缀
* 类名、protocols、常量和枚举应带有命名空间前缀；
* 类方法不要带前缀
```C
类名 YCGuideController
protocols YCGuideControllerDelegate （模仿苹果以delegate、Datasource、ing结尾）
常量 YCMainPhoneNum  (NSKeyValueChangeOldKey)
枚举 YCBrandListType（UITableViewCellEditingStyle）
类方法 + (instancetype)brandCellWithTableView:(UITableView *)tableView;
```

### <a name='naming-method'></a>方法名
* 以 `alloc`、`copy`、`init`、`mutableCopy`、`new` 开头的方法要注意，它们会改变ARC的行为。[^1]
* 以 `get`、`set` 开头的方法有特殊的意义，不要随意定义。
  1. set 是属性默认的设置方法，如果函数不是为了设置类成员，则不要用 `set` 开头，可用 `setup` 替代。
  2. get 和属性方法无关，但在 Cocoa 中，其标准行为是通过引用传值，而不是直接返回结果的。欲获取变量，直接以变量名为名，如：`userInfomation`，而不是 `getUserInfomation`。

例：

```Objective-C
// OK
- (NSString *)name;

// 糟糕，应用上面的写法
- (NSString *)getName;

// OK，但极少使用
- (void)getName:(NSString **)buffer range:(NSRange)inRange;

// OK
- (NSSize)cellSize;

// 糟糕，应用上面的写法
- (NSSize)calcCellSize;

// 对 controller 做一般设置，OK
- (void)setupController;

// 列出具体设置的内容，更好
- (void)setupControllerObservers;

// 糟糕，set 专用于设置属性
- (void)setController;
```

```Objective-C
// 来自官方文档
insertObject:atIndex:    // OK
insert:at:               // 不清晰，插入了什么？at 具体指哪里？
removeObjectAtIndex:     // OK
removeObject:            // OK
remove:                  // 糟糕，什么被移除了？
```

### <a name='naming-protocol'></a>协议名
* 好的协议名应能立刻让人分辨出这不是一个类名，除了以常用的 delegate、dateSource 做结尾外，还可以使用 …ing 这种形式，如：`NSCoding`、`NSCopying`、`NSLocking`。


### <a name='naming-notifications'></a>通知命名
* 基本命名格式是：`[与通知相关的类名] + [Did | Will] + [UniquePartOfName] + Notification`，例：

```Objective-C
NSApplicationDidBecomeActiveNotification
NSWindowDidMiniaturizeNotification
NSTextViewDidChangeSelectionNotification
NSColorPanelColorDidChangeNotification
```

### <a name='naming-temporary-variable'></a>临时变量命名
* 临时变量可以写得很短，如 i、k、vc 这样；
* 临时变量可以使用匈牙利前缀，但数据类型不可以作为前缀：

```C
// OK
wCell, vcMaster, vToolbar

// 糟糕，数据类型作为前缀
bool_switchState, floatBoxHeight
```

推荐的前缀：

前缀 | 含义
-----|-----
ix   | 序号，起始为0
in   | 序号（自然数范围），起始为1
if   | 类型为浮点的“序号”
x    | 坐标
y    | 坐标
w    | 宽度
h    | 高度
vc   | 视图控制器
v    | 视图

### <a name='naming-constant'></a>常量命名
* 除以上规则约定外，其他常量约定了以下前缀：

前缀   | 含义
-------|-----
k      | 宏常量
CDEN   | Core Data entity name
UDk    | User Default key
APIURL | 接口地址

另见：[常量管理](#constant)

### <a name='naming-match-case'></a>大小写
* 类名采用大驼峰（`UpperCamelCase`）
* 类成员、方法小驼峰（`lowerCamelCase`）
* 局部变量大小写首选小驼峰，也可使用小写下划线的形式（`snake_case`）
* C函数的命名用大驼峰

### <a name='abbreviation'/></a>缩写
* 可以使用广泛使用的缩写，如 `URL`、`JSON`，并且缩写要大写。但像将`download`简写为`dl`这种是不可以的。


```Objective-C
// OK
ID, URL, JSON, WWW

// 糟糕
id, Url, json, www

destinationSelection       // OK
destSel                    // 糟糕
setBackgroundColor:        // OK
setBkgdColor:              // 糟糕
```

### <a name='naming-viewcontroller'></a>视图命名

* 经常为了便于多个界面复用，我们会把 model 的显示统一在一个 view controller 中，在其他界面嵌入这个 view controller。我们把这类专门管理显示的 view controller 叫做 `displayer`。如：

```
UserListDisplayer
TagListDisplayer
```

* UIView 级别的组件不要以 Controller 或 Displayer 结尾，如果起到管理作用可以使用 control 结尾。
* PS: 如果你不用 Xcode 的 Open Quickly（默认快捷键 Command+Shift+O），强烈建议你改一下习惯

### <a name='naming-others'></a>其他
i，j专用于循环标号

为私有方法命名不要直接以“_”开头，而应以“命名空间_”开头。

<a name='constant'></a>常量使用
----
* 常量应该使用static来声明而不是使用#define，除非显式地使用宏（需要调用宏定义的语句）。
* 定义cell的cellReuseIdentifier时必须使用static（大量的创建NSString会耗内存）

常量定义示例：
```
// 头文件
static NSString * const RWTAboutViewControllerCompanyName;

// 实现文件
NSString * const RWTAboutViewControllerCompanyName = @"Name";
```
* 定义通知、userDefault常量时，应该与常量名相同
通知常量定义示例：
```
// 头文件
static NSString * const YCHomeChangeCellNotification;

// 实现文件
NSString * const YCHomeChangeCellNotification = @"YCHomeChangeCellNotification";
```

<a name='enum'></a>枚举类型
----
不推荐使用enum，推荐使用新的固定基本类型规格，因为它有更强的类型检查和代码补全。现在SDK有一个宏NS_ENUM()来帮助和鼓励你使用固定的基本类型。
```
// 推荐
typedef NS_ENUM(NSInteger, RWTLeftMenuTopItemType) {
  RWTLeftMenuTopItemMain,
  RWTLeftMenuTopItemShows,
  RWTLeftMenuTopItemSchedule
};
// 不推荐
enum GlobalConstants {
  kMaxPinSize = 5,
  kMaxPinCount = 500,
};
```

<a name='comment'></a>注释
----
* 当需要注释时，注释应该用来解释这段特殊代码为什么要这样做。任何被使用的注释都必须保持最新或被删除。

* 一般都避免使用块注释，因为代码尽可能做到自解释，只有当断断续续或几行代码时才需要注释。例外：这不应用在生成文档的注释

<a name='others'></a>其他
------
### <a name='exception'></a>异常
* 作为被调用模块的维护者，当被调用不当时（参数有问题、不和时宜），如何处理需要考虑（抛出异常还是返回错误状态）；
* 不要依赖 try catch，它不是代替你做检查、填补遗漏的工具。
