title: Objective-C编程思想： 目标-动作
tags: ios, target-action, 目标-动作
categories: ios, 自译自乐, Objective-C编程思想
---
# 目标-动作
>译自[Target-Action](https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaEncyclopedia/Target-Action/Target-Action.html)

尽管代理，绑定和通知三种机制在处理程序中对象间固定格式的通信很有用，然而它们并不是特别适合多数UI控件的通信。一个典型应用程序的UI通常包含若干图形对象，最常见的图形对象就是控件。一个控件是一个物理/逻辑上的器件（按钮、滑动条、复选框等）；就好比真正的收音机，你可以通过调音器（控件）向收音机（控件所属系统）表达你的意图-调节音
量的大小，这就是一个应用。

UI中控件的职能简单明确：理解用户的意图并协调其他对象封装出请求。比如说，用户点击/按住Return键，硬件设备会发起一个`原始事件`，按键接收该事件（为Cocoa做了适当的封装
）并将它转换成具体应用程序的`操作`。然而，事件本身并不会携带很多关于用户意图的信
息，仅仅是传达了用户点击/按住了一个按键这个信息。因此必须提供一个能将`原始事件`
转换为`操作`的机制。它就是*target-action*。

