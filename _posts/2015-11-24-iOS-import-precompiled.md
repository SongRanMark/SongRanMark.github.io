---
layout: post
title: 预编译头文件及#import导入头文件的认识
excerpt: 简述 Objective-C 中依赖关系管理使用的#import，苹果不再建议使用 precompiled prefix file 的原因，和预编译头文件的代替方案 Modules 的内容
---

### C语言相关内容

C语言编译过程：

1. 预处理阶段，每当编译源文件的时候，编译器首先做的是一些预处理工作。比如预处理器会处理源文件中的宏定义，将代码中的宏用其对应定义的具体内容进行替换。同样#include的本质实际上是删掉当前行，把*.h的内容完全插入到当前行的位置,如果\*.h中也使用了类似的宏引入，则会按照同样的处理方式用各个宏对应的真正代码进行逐级替代。
1. 语法检查阶段
1. 编译阶段，以.c文件为单位首先编译成纯汇编语句，再将之汇编成跟 CPU 相关的二进制码，生成相对应的目标文件(.obj)。
1. 链接阶段，将各个目标文件中的各段代码进行绝对地址定位，与C语言库函数组合，生成跟特定平台相关的可执行文件。

在真实的程序开发过程中，不可能将所有的代码写在一个文件中，所以需要根据功能不同分块写入不同的.c文件中。但不同.c文件又需要调用其他文件中的函数，如果直接include.c文件，那么在链接阶段就会发现重复定义的错误。头文件就是一个很好地解决方案，比如有A.c文件实现了一些函数，可以在A.h中声明这些函数，当其他.c文件想使用A.c文件中的函数时，就可以includeA.h文件(头文件名称的对应只是一种规范，本质上没有关联性)。

\#include重复导入的问题：如果A.h和B.h都导入了C.h，那么当另外一个.c文件包含了A.h和B.h时，就会出现重复包含C.h的编译错误。方法是将头文件中的写法改为

```
#ifndef C_H
#define C_H

//头文件的真正内容

#endif 
```

### Objective-C的#import

Objective-C是C语言的超集，支持面向对象。因此编译过程可以类比C语言的编译过程。编译应该是以.m文件为单位，而.h文件作为接口暴露出来，其中写类的声明供其他文件调用。.m文件最终也是要进行链接组成可执行文件。

注意，.h和.m本质上同样没有对应关系，只有文件中声明和实现的类是对应的。一个.m文件中可以写多个类实现，其所对应的类声明也可以写到不同的.h文件中。以类名命名相对应的.h和.m文件，并只写该类的声明与实现是一种规范。
在Objective-c语言中，使用#import来导入头文件，其作用同样是将头文件内容替换入该文件，只不过优化的地方在于，使用#import指令，可以保证头文件内容不会重复导入。

关于如何优化#import的编写可以看这两篇文章，[#imports Gone Wild! How to Tame File Dependencies](http://qualitycoding.org/file-dependencies/)，[Why #import Order Matters](http://qualitycoding.org/import-order/)
基本思路就是

- 在.h头文件中，尽量少的去引入其他的类或库，多使用@class，@protocol这种前置声明，然后在实现文件中真正用到类结构时再#import，前置声明可以最小化依赖关系，比如有A,B,C三个类，如果在B.h中导入了C.h，A.h中导入了B.h，那么当C.h中有修改，则A,B,C三个.m实现文件都需要重新编译，但实际上A与C之间并不需要引用，所以此时在B.h中应该对C作前置声明，在B.m中在导入，这样当C.h改变时，只会影响真正使用到得B.m，节约了编译时间
- 在.m实现文件中则要确保清除因各种原因遗留的已不需要的.h文件，另外，因为.m文件中导入的头文件较多，还需要注意排序的问题。我一般习惯的顺序是自身的头文件，然后是项目内的其他文件(顺序是 controller、view、model、API请求类)，接着是一些category的头文件，最后是第三方库的头文件。感觉这样的顺序还是比较清晰的。

### 预编译头文件

.pch文件为precompiled prefix file，即预编译头文件。它的作用是对编译过程加速，预编译头文件中导入的文件和其他一些内容会被提前编译，所以当项目真正编译时，这些内容可直接载入，不需要再去编译了。在编译阶段，预编译头文件的内容会被默认替换到每一个源文件的开头，就相当于是XCode会帮你在文件开头加这么一行

```
#import "xxx.pch"
```

所以其他文件可以直接使用在预编译头文件中的内容而不需要导入这些头文件了。基于这种情况我们可以将一些不会变化的框架的头文件添加到该文件中，这些框架包含大量的文件可以被预先编译，同时在使用这些公用的框架时也不需要再导入头文件，就可以直接使用了。但是项目中实现的一些公用的类的头文件并不应该放入预编译头文件中，因为预编译头文件的内容会被引入到所有源文件中，所以一旦预编译头文件中导入的文件有改动，那么会导致项目中的其他文件都会被重新编译（正常情况下，只会编译一些改动过的和改动的时候影响到的文件）。而项目中的文件跟框架文件不同，即使是公用文件，也会存在一定频率的改动。这样做的另一个弊端是，源文件中会存在隐蔽的依赖关系，当该源文件被复用到另一个项目时，这些依赖关系并不能从导入的头文件处看清楚，而是会看到一些报错信息。

### 抛弃预编译头文件

从XCode6开始，新建的项目工程中已经不再默认生成预编译头文件，这可能说明苹果不建议再使用预编译头文件了，而原因则是另一个特性：Modules，Modules出现在更早的XCode5。

> New Features in Xcode5：  
Modules for system frameworks speed build time and provide an alternate means to import APIs from the SDK instead of using the C preprocessor. Modules provide many of the build-time improvements of precompiled headers with less maintenance or need for optimisation. They are designed for easy adoption with little or no source changes. Beyond build-time improvements, modules provide a cleaner API model that enables many great features in the tools, such as Auto Linking.
 All new projects created in Xcode 5 now build with modules enabled by default. For existing projects, you enable modules by using the project Build Settings panel. Search for “module” and set Enable Modules (C and Objective-C) to YES.
Auto Linking is enabled for frameworks imported by code modules. When a source file includes a header from a framework that supports modules, the compiler generates extra information in the object file to automatically link in that framework. The result is that, in most cases, you will not need to specify a separate list of the frameworks to link with your target when you use a framework API that supports modules.

Modules的作用就是加快框架的编译速度并且提供一种代替预编译的导入框架的方法。并且Modules支持Auto Linking，当你使用Modules时，编译器会自动链接你所需要的框架而不需要你再去手动链接。Modules的语法时是@import，例如：

```
 @import UIKit;
```

但其实原项目中的#import并不需要改动，如果支持Modules的话，Xcode会自动将其转换为支持Modules的@import。

Modules相当于在编译时载入了一个框架的已经编译的版本。其实质是将框架进行了封装，然后在实际编译之时加入了一个用来存放已编译添加过的Modules列表。如果在编译的文件中引用到某个Modules的话，将首先在这个列表内查找，找到的话说明已经被加载过则直接使用已有的，如果没有找到，则把引用的头文件编译后加入到这个表中。这样被引用到的Modules只会被编译一次，从而同时解决了编译时间和引用泛滥两方面的问题。

所以抛弃预编译头文件不会丢失加速编译的特性，还可以避免其带来的改动导致全局重新编译和依赖显示不明确的弊端。stackoverflow中对不使用预编译头文件的讨论：  
[Why isn't ProjectName-Prefix.pch created automatically in Xcode 6?](http://stackoverflow.com/questions/24158648/why-isnt-projectname-prefix-pch-created-automatically-in-xcode-6)  
另一篇介绍很全面的文章：  
[Modules and Precompiled Headers](http://useyourloaf.com/blog/modules-and-precompiled-headers.html)

### 最后

总结到此，我现在在项目中并没用去掉预编译头文件，但我只保留了唯一的一个有关自定义Log的宏(实在不想在每一个源文件中添加一遍有关Log的头文件了)。在新项目中，我想我会尝试不使用预编译头文件，而是将有关Log的宏放在一个单独的Log头文件中，并且对系统框架的引用使用新的@import语法。最后提一句，现在Modules貌似已经支持非系统框架了。

[在Release版本禁用NSLog函数](http://www.jianshu.com/p/32f8e7bf7647)