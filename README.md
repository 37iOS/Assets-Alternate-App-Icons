# Assets-Alternate-App-Icons

Xcode13 配置多套 App 图标的方法 --- AppStore 图标 A/B Test 实践

背景文章：[Xcode13 配置多套 App 图标的方法 --- AppStore 图标 A/B Test 实践 - 掘金](https://juejin.cn/post/7044748618078617613)

![Xcode13-Alternatelcons-0](images/Xcode13-Alternatelcons-0.png)

多套图标测试的配置总结：

1. 使用 Xcode 13
2. 在 `Assets.xcassets` 创建多套测试的图标，并添加对应的图标
3. 将 `Include all app icon assets` 设置为 YSE


> 注意事项： 
> 1、图标不要有透明通道
> 2、所以有图标格式要用 PNG （不能用 JPG）


项目效果示例：
![Xcode13-Alternatelcons-8](images/Xcode13-Alternatelcons-8.jpg)


# 使用教程

> 最近苹果推出 App Store 产品页的新功能，其中在 app 产品页的不同版本上使用不同的图标，通过  A/B Test 找出效果最佳的版本。但是苹果文档并没有给出详细的教程，怎么在 Xcode 中集成多套图标呢？这就是本文要讲解的内容，适合 iOS 技术开发同学阅读。


### 一、背景

2021 年 12 月 08 日，苹果 [推出 App Store 产品页的新功能](https://developer.apple.com/cn/news/?id=2xnhx92t)，在 App Store 中开发者可以针对 app 产品页的不同版本上使用不同的图标、截屏和 app 预览，通过  A/B Test 找出效果最佳的版本。有部分读取可能知道我们在之前的文章 [解读 AppStore 新功能：自定义产品页面和 A/B Test 工具](https://juejin.cn/post/6986478834040209445#heading-4)，当时留下这个一个问题：**测试不同的 app 图标，提交新图标的 app 审核流程是怎么样？**

在 [Get ready to optimize your App Store product page - WWDC21](https://developer.apple.com/videos/play/wwdc2021/10295) 视频中有这样一段话：

> And remember, to test a variation of your app icon, you'll need to include the icon assets in the binary of your app version that is currently live, so make sure to prepare your app releases accordingly.
> 请记住，要测试 app 图标的变体，您需要将图标集包含在当前上线的 app 版本的二进制文件中，因此请确保相应地准备应用版本。


#### 1.1 Xcode 集成多套测试 App 图标

怎么包含不同的图标集到 app 中呢？首先，想到的是 Xcode 13 beta 版本，然后在苹果的文档 [Xcode 13 Beta 3 Release Notes | Apple Developer Documentation](https://developer.apple.com/documentation/Xcode-Release-Notes/xcode-13-beta-release-notes) 中找到这样一段话：

> **Asset Catalogs**
>  
> **New Features**
>  
> At runtime, your app can now use iOS app icon assets from its asset catalog as alternate app icons. A new build setting, “Include all app icon assets,” controls whether Xcode includes all app icon sets in the built product. When the setting is disabled, Xcode includes the primary app icon, along with the icons specified in the new setting, “Alternate app icon sets.” The asset catalog compiler inserts the appropriate content into the Info.plist of the built product. (33600923)
> 在运行时，您的 app 现在可以使用其资产目录中的 iOS app 图标资产作为备用 app 图标。新的构建设置“包括所有 app 图标资产”控制 Xcode 是否包含构建产品中的所有 app 图标集。当该设置被禁用时，Xcode 包括主 app 图标，以及在新设置“备用 app 图标集”中指定的图标。资产目录编译器将适当的内容插入到构建产品的 Info.plist 中。 (33600923)

先说结论：实现多套App图标集成，是要依赖 **Xcode 13**! **Xcode 13**! **Xcode 13**!


### 二、正文

在 Xcode 13 之前，如果要实现 iOS App 动态切换图标，需要在 Info.plist 中添加 `CFBundleAlternatelcons` 相关字段来声明对应的备用图标。如果要多套 App 图标，那么需要添加很多标签，不够直观和高效。

所以，在 Xcode 13 开始，可能通过项目的 `Assets.xcassets` 里创建 AppIcon 图标模板的形式，直观又方便管理图标。

#### 2.1 如何添加多套 App 图标

首先，我们直接来说一下怎么做，其实也不复杂。最后文章，在来总结一下注意事项。

![Xcode13-Alternatelcons-1](images/Xcode13-Alternatelcons-1.jpg)

跟添加 App 图标一样，把所有需要的图标集都添加到 `Assets.xcassets` 中就可以。

然后将 `Include all app icon assets` 改为 `YES`。（注意，需要 Xcode 13 以上才有这个字段！）

![Xcode13-Alternatelcons-2](images/Xcode13-Alternatelcons-2.jpg)


> 选项 `Include all app icon assets` 配置的作用是打包时决定 Asset Catalogs 编译器要不要把所有的备用图标也编译到 asset 集中。


所以，简单来总结：

1. 使用 Xcode 13 
2. 在 `Assets.xcassets` 创建多套测试的图标，并添加对应的图标
3. 在 `Include all app icon assets` 设置为 `YSE` 



#### 2.2 原理分析解答

##### 如何验证多套图标是否添加成功

要验证是否成功，就要知道 Xcode 做了什么。将`Include all app icon assets` 改为 `YES`，Xcode 做了几件事：

* 把每套 icon 的 60x60@2x 和 60x60@3x 两张 iOS App 图标打包到 `Assets.car` 文件中
* 把每套 icon 的 60x60@2x 和 60x60@3x 两张 iOS App 图标放到包体目录中
* 在 Info.plist 的 `CFBundleAlternateIcons`  字段下添加备用图标为名字的 key-value

所以，从这 3 个方面可以验证。

打包后，可以查看包体下的 Info.plist 文件下 Icon files (iOS 5) 配置下是否有 `CFBundleAlternateIcons` 对应的多套图标的名字：

![Xcode13-Alternatelcons-3](images/Xcode13-Alternatelcons-3.jpg)

查看 `Assets.car` 文件里的是否包含图标也可以验证，这里就不重复了。


##### 只使用部分备用的图标

可以通过 `Alternate App Icon Sets` 设置只使用的备份图标。前提条件时，`Include all app icon assets` 要设置为 `NO`.

![Xcode13-Alternatelcons-4](images/Xcode13-Alternatelcons-4.jpg)

通过上图，就可以指定只设置 3 套备用图标。**需要特别注意：**

* 填写的图标集名字，一定要与 `Assets.car` 里的名字一致
* 如果填写了错误或者不存在的名字，Xcode 会忽视并且不会报错

所以，可以通过上面说到的验证方法确定名字没有填写错。

> 注：当 `Include all app icon assets` 要设置为 `YSE` 时，Xcode 会忽视 `Alternate App Icon Sets` 设置的内容。
> 另外，`Assets.xcassets` 下的多套图标可以创建一个文件夹来包裹，这样更加好分类管理。

##### 不要用有透明的图标

![Xcode13-Alternatelcons-5](images/Xcode13-Alternatelcons-5.jpg)
 
如上图所示，使用有透明区域的图片，最终显示的图标，背景将会变成黑色：

![Xcode13-Alternatelcons-6](images/Xcode13-Alternatelcons-6.jpg)


##### 1024 图标不要用 JPG 格式

1024x1024 的商店图片，作为主图标时可以用jpg。如果用 png 格式，则不能有透明区域，否则上传 ipa 包体时会报错，无法上报。
![Xcode13-Alternatelcons-7](images/Xcode13-Alternatelcons-7.jpg)

而备用的图标，则 **不能使用 jpg！** 不能使用 jpg！不能使用 jpg！

上报时会报错：
```
Dear Developer,

We identified one or more issues with a recent delivery for your app, "斗罗大陆：魂师对决" 2.3.1 (2.3.1). Please correct the following issues, then upload again.

ITMS-90033: Invalid Image - - Image found at the path 'TestAIcon-1x~marketing-0-7-0-85-220.jpeg' referenced under key 'CFBundleAlternateIcons' is not a valid PNG file
```

所以，备用图标的 1024x1024 一定要用 PNG 格式！

##### 代码进行图标切换

最后，因为已经配图和验证了备用图片已经生效，那么也可以用代码来调用，切换备用图标测试：


设置某个备用图标时，传入图标集的名字就可以，一句代码：
```
UIApplication.shared.setAlternateIconName("Rainbow")
```

设置回默认图标的代码：

```
UIApplication.shared.setAlternateIconName(nil)
```

最后，我们将以上配置示例，上传到 GitHub 上，提供了 ObjC、Swift、SwiftUI 的版本，大家可以参考。

* [37iOS/Assets-Alternate-App-Icons - GitHub](https://github.com/37iOS/Assets-Alternate-App-Icons)

![Xcode13-Alternatelcons-8](images/Xcode13-Alternatelcons-8.jpg)

##### 更灵活的配置

`Include all app icon assets` 在 Xcode 的对应 buildSettings 的名字是  `ASSETCATALOG_COMPILER_INCLUDE_ALL_APPICON_ASSETS`，而 `Alternate App Icon Sets` 对应的字段是 `ASSETCATALOG_COMPILER_ALTERNATE_APPICON_NAMES`。还有一个设置 App 主图标的字段 `Primary App Icon Set Name` 对应是 `ASSETCATALOG_COMPILER_APPICON_NAME`。


其实这些字段，在 Xcode 13 有映射成了 GUI 界面：

![Xcode13-Alternatelcons-9](images/Xcode13-Alternatelcons-9.jpg)

这有什么好处？通过多套图标测试后的数据，可能需要使用某个备用图标设为主图标，通过 `General` 面板上，可以快速把备用的图标集改成主图标。


##### 苹果后台的配置

需要注意，包含多套图标的包，需要按包体送审的流程过审后，才能在苹果后台 `产品页优化` 标签下看到多套图标，`App Icon` 分标签下：

![Xcode13-Alternatelcons-10](images/Xcode13-Alternatelcons-10.png)


具体，如果配置多套图标的测试方案，可以参考苹果文档：[配置处理方案 - App Store Connect 帮助](https://help.apple.com/app-store-connect/#/devb53f12312)。这里就不展开了。


### 总结

Xcode 13 支持通过 `Assets.xcassets` 配置多套备用图标，减少了开发者以前繁琐的配置，提升开发体验，希望后续有更多开发者来尝试，多套图标能一定程度上满足用户的自定义需求。

对多套 App 素材进行 A/B 测试，找出效果最佳的素材，是一个优秀的产品迭代优化的手段。大家都可以马上尝试一下，`找出最具吸引力的版本`，更吸引更高效的 App Store 产品页！

最后，我们现在参加《2021年度掘金人气团队榜》，希望大家可以给我们加油打气！鼓励我们2022年在创新篇~ 

* 加油打气入口：[投票链接](https://rank.juejin.cn/rank/2021/1002387318511214)
* 加油打气入口：[投票链接](https://rank.juejin.cn/rank/2021/1002387318511214)
* 加油打气入口：[投票链接](https://rank.juejin.cn/rank/2021/1002387318511214)


## 参考

- [解读 AppStore 新功能：自定义产品页面和 A/B Test 工具](https://juejin.cn/post/6986478834040209445)
- [现已推出 App Store 产品页的新功能 - 新闻 - Apple Developer](https://developer.apple.com/cn/news/?id=2xnhx92t)
- [Get ready to optimize your App Store product page - WWDC21](https://developer.apple.com/videos/play/wwdc2021/10295)
- [Xcode 13 Beta 3 Release Notes | Apple Developer Documentation](https://developer.apple.com/documentation/Xcode-Release-Notes/xcode-13-beta-release-notes) 
- [How to change your app icon dynamically with setAlternateIconName()](https://www.hackingwithswift.com/example-code/uikit/how-to-change-your-app-icon-dynamically-with-setalternateiconname)
- [Alternate App Icons using Asset Catalogs in Xcode 13| Medium](https://katzenbaer.medium.com/alternate-app-icons-using-asset-catalogs-in-xcode-13-da6387d1cd78)
- [Simple alternate app icons with Xcode 13 and SwiftUI](https://iosexample.com/simple-alternate-app-icons-with-xcode-13-and-swiftui/)
- [jknlsn/XCode13-Alternate-App-Icons](https://github.com/jknlsn/XCode13-Alternate-App-Icons)
- [配置处理方案 - App Store Connect 帮助](https://help.apple.com/app-store-connect/#/devb53f12312)
- [Product Page Optimization - App Store - Apple Developer](https://developer.apple.com/app-store/product-page-optimization/)
- [37iOS/Assets-Alternate-App-Icons - GitHub](https://github.com/37iOS/Assets-Alternate-App-Icons)