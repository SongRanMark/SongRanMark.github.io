---
layout: post
title: ios开发-图片资源管理
excerpt: 图片的命名规范及 Asset Catalog 的使用
---

移动应用作为面向用户的客户端，展示内容的界面需要大量的图片资源，在 iOS 开发中，如果对图片资源有规范的命名和清晰的分类管理，可以使开发人员更方便的使用和提高开发效率。

### 图片格式

##### PNG VS JPG

开发中主流使用的格式主要是 .png 和 .jpg 两种，.png 相对于 .jpg 的优点是，解压缩效率高，对 CPU 消耗小，而且是无损压缩，苹果公司推荐使用的格式也是 .png。而且 Asset Catalog 仅支持 .png 格式，如果项目中有 .jpg 格式的资源，则不能放入其中，需要再单独建立普通文件夹存放。

##### PDF

如果你的项目支持 PDF 矢量图，那么就无需像位图那样管理很多适配的图片资源了，每一种图片，只需要向 Asset Catalog 导入一张矢量图即可，Xcode 会在编译时将其转化为需要的位图，如1倍图到3倍图等。

### 图片命名规范

[iOS 及 Android 应用开发中，怎样命名和管理切图资源最科学？](https://www.zhihu.com/question/21348866)

[iOS 切图文件命名规范](http://www.jianshu.com/p/2896b2823b65)

最基本的原则是，文件名应该只描述图片的用途，而不描述图片的样式，所以名字中不应出现对颜色，样式等描述的单词。一种规范的格式是：

```
module_identifier_type_state
```

* module，代表的是功能模块，大部分的图片是对应着不同的界面，即对应了不同的功能模块，包括登录和启动界面也应作为一个单独的模块。对于一些在很多界面中通用的资源，可以放在 common 模块中。针对 iOS 开发来说，一般会将 TabBar 上用到的资源单独的抽出为 tab 模块。把 module 放在第一位的好处是相对于 type(btn，icon等) 更利于检索。可以把 module 理解为命名空间（namespace）的作用。
* identifier，主要作用就是用来描述图片的用途，如一个用于设置按钮的图片，这一部分就可以写为 setting。当一个单词无法准确描述用途时，也可用多个单词由 _ 分隔。
* type，说明图片的类型，额外提供了一些图片的信息，一般可以分为背景(bg)，图标(icon)，图片(img)，按钮(btn)等。当然除了按钮(btn)外也可以设置一些其他控件，如输入框，可以写为 input 或 tf。
* state，例如一个按钮的不同状态要对应不同的图片，在描述符相同的情况下，就可以用 state 来区分，可以是 normal，highlighted，selected，disabled 等。如果你的图片命名中用到了颜色，看看是否可以用 state 来取代那种通过样式来区别命名的方式。

除了上述主要内容外，还有一些其他的基本要求：

* 命名使用英文并且全小写，不使用拼音。
* 单词间用 _ 分隔而不是其他符号。

### 分组管理

可以按照 module 来建立目录对图片资源进行分组管理。其实这里分目录并不与图片名以 module 开头功能重复，分目录的作用是，当项目中的图片越来越多时，可以对整个图片文件列表有清晰方便的管理。而在使用图片时，则可以通过图片名中的 module 来高效的索引和清楚的指明。·

### Asset Catalog

[Asset Catalog Help](https://developer.apple.com/library/ios/recipes/xcode_help-image_catalog-1.0/_index.html)

[Asset Catalog Format Reference](https://developer.apple.com/library/ios/documentation/Xcode/Reference/xcode_ref-Asset_Catalog_Format/index.html)

Asset Catalog 是从 iOS7 开始引入的一种更方便的管理应用内资源的机制，非常适合用来管理应用内的图片资源。Asset Catalog 提供对图片和通用类型数据的管理，图片数据要求必须是 .png 格式，而对于通用类型的数据，可以是除了二进制可执行文件以外的任何类型。使用 Asset Catalog 另一个好处是，它能帮助我们实现 APP 瘦身（APP Thinning），Asset Catalog 能为不同的平台、不同设备甚至相同类型的设备但是不同的配置（比如内存大小等）提供定制化的资源供给解决方案，当用户下载应用时，只有该用户的设备需要的资源才会被下载下来，这样就减小了用户下载的包的大小。

![](http://upload-images.jianshu.io/upload_images/669609-7f4b6b57b0db3c59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Asset Catalog 的使用非常简单，选中文件后，Xcode 提供了一个方便的操作区，左侧是所有资源项的列表，在列表底部可以新建，删资源项，当然也直接将图片拖入列表建立资源项。中间显示了该资源项内的所有适配的资源文件，右侧是资源项的属性检查器。而在编程时，使用的就是资源项的名称。

![](http://upload-images.jianshu.io/upload_images/669609-fb31c2709b3156da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Asset Catalog 本质上就是通过目录来管理资源，其本身是一个 .  xcassets 后缀的目录，目录内包含了所有的资源项，例如可能是一个 Image Set 或 Data Set，也有可能是 App Icons 或 Launch Images，它们都是后缀为 .imageset，.launchimage 或 .appiconset 的目录。从命名(Set，Icons，Images)中也可以看出每一项都代表了一个资源集合，当然要用目录来管理。资源项目录里存放的是某一资源针对不同设备，Size Classes，内存，分辨率等而定制的不同版本的文件。.xcassets 目录内还有一种没有后缀的普通目录对应的就是 Asset Catalog 中的 Folder，用来对资源项进行分组管理。

![](http://upload-images.jianshu.io/upload_images/669609-b87e3116626adda3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


而要让程序可以正确找到不同版本的文件，Xcode 会自动在资源项目录里创建一个特殊的配置文件 Contents.json，在这个文件中记录了该资源的不同版本的文件对应的设备选项，以供程序可以选择正确的图片资源用于显示。文件内容类似如下形式：

```
{
  "images" : [
    {
      "idiom" : "universal",
      "scale" : "1x"
    },
    {
      "idiom" : "universal",
      "filename" : "payment_success_icon@2x.png",
      "scale" : "2x"
    },
    {
      "idiom" : "universal",
      "scale" : "3x"
    }
  ],
  "info" : {
    "version" : 1,
    "author" : "xcode"
  }
}
```

所以 Asset Catalog 里的每一个资源项(即文件资源的集合目录)应按照图片的命名规范来命名，因为最终在代码和 Interface Builder 里我们直接引用的都是 Asset Catalog 里的资源项。而最终具体使用哪一个版本的资源文件就是系统根据配置文件 Contents.json 来决定了。这样看来，其实具体的图片文件的命名相对来说就不是很重要了，当我们将图片文件通过 Xcode 在 .xcassets 文件里的不同选项间拖动时，就是在改变 Contents.json 文件里的映射记录，而这些映射都是供系统查找用的。但为了文件的规范管理，我们还是应该对所有图片文件有一个统一规范的命名，我认为比较好的一种方法是，首先同一资源项下的所有文件开始都使用资源项的名称，后面加不同的后缀，比如 Universal 里的1倍，2倍，3倍图就可以用 iOS 在没有 Asset Catalog 时的命名规则，在后面加@2x，@3x。其他针对不同设备，Size Classes，内容等也可规定相应的统一后缀。

```
payment_success_icon.png
payment_success_icon@2x.png
payment_success_icon@3x.png
```

##### App Icons

App Icons 是一个特殊的 Image Set，这个资源集合里包含了所有适配的应用图标文件。资源项的名称可直接使用 Xcode 中默认的 AppIcon ，而图片资源的名称，因为一般只会有一套，可以直接使用[Prepo](https://itunes.apple.com/us/app/prepo/id476533227?mt=12)切出的图标名称，格式为`Icon-`后加适配的后缀

```
Icon-76.png
Icon-76@2x.png
Icon-iPadPro@2x.png
Icon-Small.png
Icon-Small@2x.png
Icon-Spotlight-40.png
Icon-Spotlight-40@2x.png
```

##### Launch Images

Launch Images 是另一种特殊的 Image Set，这个资源集合里存放的是所有适配的应用启动图。当然现在可以用 .xib 文件或 .storyboard 文件作为应用启动图适配文件，但其中的图片还是要放到 Asset Catalog 里的。因为一般启动图也是只有一套，所以资源项的名称也可直接使用 Xcode 默认的 LaunchImage，而图片资源可以 `launch_img`开始，后跟 iPad，landscape / portrait，@2x/@3x等后缀。

```
launch_img_iPad_landscape@2x
launch_img_iPad_portrait@2x
launch_img_iPhone_landscape@2x
launch_img_iPhone_portrait@2x
```

##### 可配置的资源文件的选项

* **Size Classes**，可以设置不同 Size Classes 组合对应的 1-3 倍图，iOS8 开始支持。
* **Platforms**，可以针对不同的平台系统(iPhone，iPad，Mac，Apple TV，Apple Watch)设置多倍图。
* **Device Memory**，可以根据设备内存(1G，2G)的不同分别设置图片
* **Device Graphics**，根据不同的设备渲染能力。

而且，这几项是可以同时组合使用的，使开发者可以非常灵活的提供较多个针对不同情况的图片资源文件。

##### 命名空间

使用上述命名规范并不会造成资源项名称相同的情况，但如果有名称冲突的情况，可以通过 .xcassets 内的管理资源项的分组目录提供命名空间的支持。选中该目录，就可以在属性检查器中设置

![](http://upload-images.jianshu.io/upload_images/669609-ce9bb5e86fc806c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用时则需要在名称前加命名空间的名字

```
UIImage *image = [UIImage imageNamed:@"Folder/payment_success_icon"];
```

##### Image Slicing

从 iOS7 开始支持 Image Slicing 功能，通过点击内容区右下角的 Show Slicing 按钮就可以开始配置了。以水平方向为例，在左线与中线之间的部分，为图片的可变的区域，用于图片缩放时的填充和压缩，而中线与右线之间的所有像素将会被隐藏。对于竖直方向，则上线对应水平方向的左线，下线对应水平方向的右线。另外，可以通过界面右侧的属性设置中的Slicing部分对其进行操作，Center包含了可变区域变化的两种模式，分别是stretch(拉伸)和tile(平铺)。

![](http://upload-images.jianshu.io/upload_images/669609-ff44e45ee5ab2019.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### Render Mode

UIImage 的 [renderingMode](http://www.jianshu.com/p/7c9d7605491d) 也集成到了 Asset Catalog 中了。可以在资源项的属性检查器中设置图片的渲染模式。

![](http://upload-images.jianshu.io/upload_images/669609-63695fa11d097950.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 设置图片资源为矢量图


![](http://upload-images.jianshu.io/upload_images/669609-dec7ab3880cd2ebe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
