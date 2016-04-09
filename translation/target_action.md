title: 目标-动作模式
tags: ios, target-action, 目标-动作模式
categories: ios, 自译自乐
---
#目标-动作模式
>译自[Target-Action](https://developer.apple.com/library/ios/documentation/General/Conceptual/Devpedia-CocoaApp/TargetAction.html)
目标-动作模式是一种当一个事件发生时一对象持有发送给另一对象的消息的必要信息的设计模式。
~~目标-动作模式是一种当一个事件发生时一对象持有一些必要信息作为消息发送给另一对象的设计模式。~~
必要信息包含两项数据：一个用以指定被调用方法的`动作`选择器；一个接收消息的对象，即`目标`。事件发生时发送的消息称为`动作消息`。尽管`目标`可以是任意对象，甚至是框架对象，通常是指定自定义控制器
作为处理`动作消息`的`目标`。

![](http://ppoffice.github.io/hexo-theme-hueman/gallery/thumbnail.jpg "") 七牛target-action 图片

任意事件都可以触发`动作消息`，就如任意对象都可以发送消息。例如，当自身的手势被识别时，一个手势识别对象
可以发送一条`动作消息`给另一对象，在按钮和滑动条等控件中有很多目标-动作模式的典型应用。

## 动作方法的格式必须是确定的
动作方法必须有约定的签名。UIKit框架兼容一些签名的变种，但是两个平台（哪两个）都接收如下的动作方法：

> - (IBAction)doSomething:(id)sender;

IBAction修饰符，等同于返回值void，它指名此方法是一个`动作`，供Interface Builder(下简称IB)识别。要让动作方法在IB中显示，必须先在接收动作消息的对象类的头文件中声明它。

