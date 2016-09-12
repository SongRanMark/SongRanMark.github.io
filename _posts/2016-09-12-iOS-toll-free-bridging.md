---
layout: post
title: Toll-Free Bridging
excerpt: iOS 开发 Toll-Free Bridging 机制概述
---

### 相关文档

[Toll-Free Bridging](https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaEncyclopedia/Toll-FreeBridgin/Toll-FreeBridgin.html)  
[Toll-Free Bridged Types](https://developer.apple.com/library/ios/documentation/CoreFoundation/Conceptual/CFDesignConcepts/Articles/tollFreeBridgedTypes.html)

### 概述

Toll-Free Bridging 是指，在一部分 Core Foundation 框架 和 Foundation 框架相配对的数据类型间，可自动转换使用的机制。语法则是在变量前的括号中写入配对的数据类型，例：

```
NSLocale *gbNSLocale = [[NSLocale alloc] initWithLocaleIdentifier:@"en_GB"];
CFLocaleRef gbCFLocale = (CFLocaleRef) gbNSLocale;
CFStringRef cfIdentifier = CFLocaleGetIdentifier (gbCFLocale);
NSLog(@"cfIdentifier: %@", (NSString *)cfIdentifier);
// logs: "cfIdentifier: en_GB"
CFRelease((CFLocaleRef) gbNSLocale);

CFLocaleRef myCFLocale = CFLocaleCopyCurrent();
NSLocale * myNSLocale = (NSLocale *) myCFLocale;
[myNSLocale autorelease];
NSString *nsIdentifier = [myNSLocale localeIdentifier];
CFShow((CFStringRef) [@"nsIdentifier: " stringByAppendingString:nsIdentifier]);
// logs identifier for current locale
```

所以，当有一个 Foundation 级的变量时，通过自动转换，就可以直接传入一个需要 Core Foundation 对应类型参数的函数中，当然也可以向 Core Foundation 级的变量发送 Objective-C 的消息。包括CFRelease、release、autoreleased 等。

并不是所有的对应类型间都可自动转换，例如 NSRunloop 与 CFRunLoop、NSBundle 与 CFBundle 以及 NSDateFormatter 与 CFDateFormatter 之间，都是不支持的，支持的数据类型如下表所示：

![](http://oc34tply2.bkt.clouddn.com/iOS_toll_free_bridging.png)

### Toll-Free Bridging 与 ARC

MRC 下的 Toll－Free Bridging 因为不涉及内存管理的转移，相互之间可以直接交换使用，当使用 ARC 时，由于Core Foundation 框架并不支持 ARC，此时编译器不知道该如何处理这个同时有 ObjC 指针和 CFTypeRef 指向的对象，所以除了转换类型，还需指定内存管理所有权的改变，可通过 `__bridge`、`__bridge_retained` 和 `CFBridgingRetain`、`__bridge_transfer` 和 `CFBridgingRelease`。
例`CFLocaleRef gbCFLocale = (__bridge CFLocaleRef)gbNSLocale;`

- `__bridge`，类型转换时并不改变内存管理方式。当从 Foundation 向 Core Foundation 转换时，编译器仍然负责管理在 Foundation 端的引用计数，不会在 bridge 的时候 retain Core Foundation 端的对象，所以也不需要开发者在 Core Foundation 端手动释放。当从 Core Foundation 向 Foundation 转换时，编译器会负责 Foundation 一端的内存管理，而开发者仍需要负责管理 Core Foundation 一端的内存，负责 release 对象。
- `__bridge_retained` 和 `CFBridgingRetain`，当从 Foundation 类型转换为 Core Foundation 类型时，如果 Foundation 的变量被释放，那么此时使用 Core Foundation 的变量会崩溃。通过`__bridge_retained`，在 bridge 的时候，编译器会 retain Core Foundation 端的对象，之后由开发者负责手动管理内存，这样即使 Foundation 的变量被释放，也不会影响 Core Foundation 的变量的使用。
- `__bridge_transfer` 和 `CFBridgingRelease`，当从 Core Foundation 类型转换为 Foundation 类型时，编译器转移了对象的所有权，开发者不再需要负责 Core Foundation 端对象的内存管理。

### Toll-Free Bridging 原理

[Toll-Free Bridging 是如何实现的？](http://gracelancy.com/blog/2014/04/21/toll-free-bridging/)  
[Toll Free Bridging Internals](https://www.mikeash.com/pyblog/friday-qa-2010-01-22-toll-free-bridging-internals.html)