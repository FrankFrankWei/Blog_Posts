title: Objective-C编程思想： 目标-动作
tags: ios, target-action, 目标-动作
categories: ios, 自译自乐, Objective-C编程思想
---
# 目标-动作
>译自[Target-Action](https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaEncyclopedia/Target-Action/Target-Action.html)

尽管代理，绑定和通知三种机制在处理程序中对象间固定格式的通信很有用，然而它们并不是特别适合多数UI控件的通信。一个典型应用程序的UI通常包含若干图形对象，最常见的图形对象就是控件。一个控件是一个物理/逻辑上的器件（按钮、滑动条、复选框等）；就好比真正的收音机，你可以通过调音器（控件）向收音机（控件所属系统）表达你的意图-调节音
量的大小，这就是一个应用。

UI中控件的职能简单明确：理解用户的意图并协调其他对象封装出请求。比如说，用户点击/按住返回键，硬件设备会发起一个`原始事件`，按键接收该事件（为Cocoa做了适当的封装）并将它转换成具体应用程序的`操作`。然而，事件本身并不会携带很多关于用
户意图的信息，仅仅是传达了用户点击/按住了一个按键这个信息。因此必须提供一个能将`原始事件`转换为`操作`的机制。它就是
*target-action*。

## 目标
目标是动作信息的接收者。控件或更为常见的是控件的单元用[outlet](https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaEncyclopedia/Outlets/Outlets.html#//apple_ref/doc/uid/TP40010810-CH10-SW1)保存动作消息的目标。虽然任何实现了适当动作方法的Cocoa对象都可以作为目标，它通常都会是用户自定义的类的实例。

也可以设置控件/单元的outlet为nil，这样的话目标由运行时决定。当目标是nil，应用程序对象（NSApplication/UIApplication）以以下固定顺序搜索适当的接收者：
1. 从关键窗口的第一响应对象开始，顺着响应对象链查找；
> *注意*：OS X中的关键窗口对应用的按键输入做响应，它是菜单和对话框消息的接收者。应用的主窗口主要关注用户操作且通常记录关键状态。

2. 查看窗口对象，然后是窗口对象的代理；
3. 如果主窗口不是关键窗口，从主要窗口的第一响应对象开始，然后在主窗口响应对象链的窗口对象及其代理中冒泡查找;
4. 然后，应用程序对象尝试做出响应。如果无法做到，会尝试用代理做出响应。应用程序对象及其代理是响应链的最末节点。

控件对象不能（也不应该）保存它们的目标。但是，发送动作消息的控件的客户端（通常是应用程序）要确保消息的目标存在并能够接收到动作消息。因此，客户端必须要将消息目标保存在内存环境中。这同样适用于代理和数据源。

## 动作
动作是控件发送给目标的消息，或从目标的角度来说，是目标实现的用以响应动作消息的方法，控件/控件单元以`SEL`型变量存储动作。`SEL`是Objective-C中的数据类型，用于指定消息的签名（即方法签名）。动作消息必须有一个明确、唯一的签名。调用的方法无返回值而且通常仅有一个`id`类型的参数，参数名习惯为`sender`。如下是一个*NSResponder*类的例子，它定义了若干动作方法：
```objective-c
- (void)capitalizeWord:(id)sender;
```

Cocoa框架中某些类定义的动作方法有等效的签名：
```objective-c
- (IBAction)deleteRecord:(id)sender;
```

`IBAction`并不为返回值指定数据类型，即没有返回值。`IBAction`是一个类型修饰符，供`Interface Builder` 在应用将编码添加的动作与内部的动作列表同步的过程中识别。

> iOS Note: UIKit中，动作选择器可以有两种格式。详情参看下文**Target-Action in UIKit**

参数`sender`代表发送动作消息的控件（尽管它可以是真正sender的替代者），就好比明信片的回信地址。目标可以查找sender获取需要的信息。如果真正的sender对象指定另一个对象作为替代者，可以将替代者当真正的sender处理，比如当用户在text
field输入文本，目标中名为`nameEntered`的方法将被调用：
```objective-c
- (void)nameEntered:(id) sender{
    NSString *name = [sender stringValue];
    if(![name isEqualToString:@""]){
        NSMutableArray *names = [self nameList];
        [names addObject:name];
        [sender setStringValue:@""];
    }
}
```
这里，响应方法提取text field的文本内容，添加字符串到数组，最后清空text field。另一些查询sender的方式有：请求一个`NSMatrix`对象获得选择行（[sender selectedRow])；请求`NSButton`对象获取它的状态（[sender state]）；请求任意控件中的单元的tag（[[sender cell] tag]），一个tag是一个数字标识符。

## AppKit框架中的Target-Action(to be continue...)
