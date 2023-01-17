---
{"title":"一统江湖？——Flutter for All Screens初体验","tags":["Flutter"],"dg-publish":true,"permalink":"/实战教学/Flutter for All Screens/","dgPassFrontmatter":true}
---


## 前言

2018 年 2 月 27 日，Google 发布了 Flutter 的第一个 Beta 版本，由于自己是一个 Google 粉，所以很快就下载尝鲜了，之后还在简书上发过一篇博客《你好，Flutter》，是我的第一篇阅读量过 10w 的文章。在学习 flutter 期间也做过一些零散的笔记，但由于当时觉悟不高，并没整理成册，而且当时正准备保研，手头事情很多加上可学习的资料很少，中途便放弃了。

机缘巧合，最近阅读到了一篇谷歌开发者的文章[《Flutter: a Portable UI Framework for Mobile, Web, Embedded, and Desktop》](https://developers.googleblog.com/2019/05/Flutter-io19.html)，说是现在的 Flutter 已经可以运行在 Android、ios、MacOS、Linux、Windows 和嵌入式设备上了。在好奇心的作祟下，我尝试着利用 Flutter 在一些平台上运行了一些 demo，本文便是记录我利用 Flutter 实现了移动端、桌面端和 Web 端的过程，由于移动端应用的 demo 网上教程很多，所以本文尽快略过，重点将放在桌面端和 web 端。

## Flutter for Mobile

初次了解到 Flutter 的时候便是一个横跨 iOS 和 Android 两个平台的框架，无论是在 Mac/Linux 还是 Windows 上搭建 Flutter 的开发环境都很简单，Windows 上的环境搭建可以参考这篇文章 👉[《安装搭建 Flutter 环境》](https://www.jianshu.com/p/84942e400298)，Mac/Linux 可以参考中文官网给出的教程 👉[《安装和环境配置》](https://flutter-io.cn/docs/get-started/install)

> 如果你在中国的网络环境下使用 Flutter，注意一定要按照要求设置好两个环境变量
>
> ```
> export PUB_HOSTED_URL=https://pub.flutter-io.cn
> export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
> ```

配置好 Flutter 开发环境后，我们通过 Android Studio 新建一个 Flutter 项目，然后启动 iOS/Android 模拟器，选中要运行的模拟器，直接运行 Flutter 项目即可。运行结果如下：

![](https://cdn.ytools.xyz/uPic/1240-20230116112557085.jpeg)

> 这里针对移动端的介绍很少，主要是因为官方文档已经讲解的非常详细，直接阅读即可 👉[《安装和环境配置》](https://flutter-io.cn/docs/get-started/install)

## Flutter for Desktop

### 先决条件

要使 Flutter 在桌面上运行，我们**必须**使用 Flutter 的**master**分支。执行以下代码即可：

```bash
flutter channel master
flutter upgrade
```

现在我们再来执行：

```
flutter doctor
```

应该是可以看到类似于下图的结果（截至 2019 年 7 月 1 日）：

![](https://cdn.ytools.xyz/uPic/1240-20230116112601703.jpeg)

默认情况下，Flutter 没有启用桌面支持。我们可以通过设置环境变量`ENABLE_FLUTTER_DESKTOP = true`来实现。

为此，我们需要在不同的终端中执行不同的命令（临时生效）：

在 macOS 或者 Linux 上:

```
export ENABLE_FLUTTER_DESKTOP=true
```

在 Windows 上:

PowerShell:

```
$env:ENABLE_FLUTTER_DESKTOP="true"
```

CMD:

```
set ENABLE_FLUTTER_DESKTOP=true
```

> **Tips:**以上设置环境变量的方式是临时的，只会在当前终端中生效，想要永久生效请自行配置系统的环境变量

现在我们来执行以下命令，来确保桌面设备已经连接了。

```
flutter devices
```

如果设置成功是会出现下面的结果的：

```bash
$ flutter devices
1 connected devices:

macOS  • macOS  • darwin-x64     • Mac OS X 10.14.5 18F203
```

### 针对不同系统手动配置

时至今日，Flutter for Desktop 仍然是一个实验性功能，这意味着 Flutter 没有工具支持，无法通过 flutter create 命令直接新建出一个桌面应用程序。因此，唯一的选择是手动配置系统特定的文件。值得庆幸的是，Google 的 Flutter 团队已经为我们做好了这件事。

不过在运行 Flutter for Desktop 之前，我们需要先针对 Windows/MacOS 进行手动配置（Linux 的配置与 MacOS 类似）。

#### MacOS

执行`flutter doctor -v`，根据输出信息选择我们需要安装配置的包，Xcode 的下载直接在 Mac App Store 下载即可，Xcode 相关开发包的安装直接执行下面的命令即可。

```
 sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
```

然后通过输入命令 `sudo xcodebuild -license` 来确保已经同意 Xcode 的许可协议。

这里重点提一下 CocoaPods 的安装配置。

CocoaPods 是 iOS 开发、macOS 开发中的包依赖管理工具，效果如 Java 中的 Maven，nodejs 的 npm。

安装只需执行以下命令：

```
sudo gem install cocoapods
```

如果下载太慢可以更换一下国内源

```
gem sources --remove https://rubygems.org/
gem sources -a http://gems.ruby-china.com/
```

然后我们需要对 CocoaPods 初始化，由于 CocoaPods 包有 500 兆左右的大小，直接执行`pod setup`会从 github 上 clone 得极其漫长，这里给出解决方案。

依次执行下面的命令即可：

```
cd ~/.cocoapods/repos
pod repo remove master
git clone https://mirrors.tuna.tsinghua.edu.cn/git/CocoaPods/Specs.git master
pod setup
```

#### Windows

感谢微软爸爸提供的 VS 套件，很多开发环境通过 Visual Studio 直接安装就可以了，如何下载安装 VS 自行百度，安装的时候记得选以下桌面开发的套件：

![](https://cdn.ytools.xyz/uPic/1240-20230116112610539.jpeg)

### 运行官方 demo

根据不同系统配置好环境后，我们便可以开始运行 Google 团队提供的 Flutter for Desktop 案例了。

在终端中执行下述命令：

```bash
git clone https://github.com/google/flutter-desktop-embedding.git
cd example
```

**example**文件夹是这个 demo 的示例应用程序，它具有所有必需的构建脚本，这些脚本在 MacOS，Windows 和 Linux 上运行 Flutter 是必需的。如果我们在 VS Code 中打开示例文件夹，我们将能够看到如下内容：

![](https://cdn.ytools.xyz/uPic/1240-20230116112614395.jpeg)

`lib/main.dart`是整个 flutter 项目的启动文件，这里我们无需过多关注 linux/macos/windows 里面的内容。

接下来在**example**目录下执行下面命令来获取项目所需要的依赖文件

```
flutter packages get
```

在我们开始运行我们的应用程序之前，还有最后一步。虽然我们之前已经配置好了 Flutter 的开发环境，但是由于桌面开发仍有一些配置项是不一样的，所以我们需要执行下面一个命令来确保所有需要的依赖都被安装成功了。

```
flutter precache --macos
```

根据你自己的系统切换所需的该命令之后的参数。

现在我们可以将我们的 Flutter 应用程序作为桌面应用程序运行了。

在终端执行：

```
flutter run
```

终端输出的结果应该是类似下面这样的：

![](https://cdn.ytools.xyz/uPic/1240-20230116112622324.jpeg)

运行起来的结果应该如下图所示：

![](https://cdn.ytools.xyz/uPic/1240-20230116112626997.jpeg)

是不是和之前的 App 一模一样呢？运行在 windows 上也是一样的（因为我没有在 Linux 下配置 Flutter 的环境，所以这里就不放出来了）。

![](https://cdn.ytools.xyz/uPic/1240-20230116112630141.jpeg)

> **Tips**：如果无法运行 demo，记得执行`flutter doctor -v`命令查看究竟还缺少什么依赖

### 简单分析下 lib/main.dart

其实我们新建一个 Flutter 的移动端项目时的 main.dart 代码和该 demo 中的 main.dart 代码几乎类似，但在开头几行还是有些不一样的地方。

- Flutter for Mobile：

![](https://cdn.ytools.xyz/uPic/1240-20230116112634358.jpeg)

- Flutter for Desktop：

![](https://cdn.ytools.xyz/uPic/1240-20230116112635905.jpeg)

此代码提供了一种覆盖默认目标平台的方法。这可以根据应用程序的要求使用。要了解更多相关信息，可以查看[《Target Platform Override》](https://github.com/flutter/flutter/wiki/Desktop-shells#target-platform-override)

### 运行已经存在的 Flutter 项目

现在我们有了必要的配置文件和脚本。也走过了基本的配置流程，接下来我们就可以在桌面上运行*几乎*任何已有的 Flutter 项目了。

有两种方法可以实现上述需求：

1. 我们可以将系统特定文件夹（linux，mac 或 windows）从 example 目录复制到已有项目目录（和 andorid 或 ios 目录同级）并且在 main.dart 中按照上一节的区别修改部分代码。
2. 我们可以使用已有项目中的 lib 文件夹替换 example 目录中的 lib 文件夹，并将 pubspec.yaml 文件替换为现有文件。

Flutter Community 的 Ayush Shekhar 建议采用第二种方式，因为他在使用第一种方式迁移的时候经常出问题，不过我在运行的时候并没有发现问题，所以因人而异吧。

之前做过一款名为“果核”的校园 App，这是他运行在 mac 上的亚子。

![](https://cdn.ytools.xyz/uPic/1240-20230116112639930.jpeg)

> **Tips：**我在使用 Flutter for Desktop 的时候发现了一个小 Bug，就是拖动窗口调整大小时，窗口整体会出现红色的闪烁。我猜可能是窗口绘制刷新导致的。

## Flutter for Web

说完了 Flutter for Mobile/Desktop，我们来请出今天的最后一位嘉宾，Flutter for Web。

与其说是 Flutter for Web 倒不如说是 Dart for Web，从 Dart 这个语言诞生之初，它就一直在尝试编译成 JavaScript。谷歌怎么想的，咱也不知道，咱也不敢问。在 Flutter 刚诞生的时候其实并没有针对 web 的计划，不过后来谷歌的工程师大笔一挥，干脆重写了新的 dart:ui，这也就导致不可能将所有的 Flutter 代码都运行到 Web 端（有些特性是平台独有的），因此这里我们仅仅是跑通官方 Demo。

### 安装 Dart SDK

篇幅原因，这里就只给出在 Mac 上安装 Dart SDK 的过程了。因为要开发 web 工程，我们还需要安装 Dartium 和 Content Shell，好在 Mac 下的 homebrew 非常强大，这里直接执行下面命令就可以了。

```
brew tap dart-lang/dart
brew install dart --with-content-shell --with-dartium
```

如果觉得 brew 下载太慢的话，可以考虑更换中科大的镜像源，只需执行下面的命令：

```
替换brew.git:
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git

替换homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
```

之后再执行`brew install`就会快很多了。

安装完成后。在终端中执行下述命令来检查 Dart SDK 的版本：

```
brew info dart
```

### 安装 flutter_web 开发工具包

由于 Flutter for Web 采用的库和 Flutter 有所差异，所以我们还需要安装 webdev 包，终端执行下面语句即可：

```bash
flutter pub global activate webdev
```

确保`$HOME/.pub-cache/bin`路径在你的环境变量中，这样你就可以直接在终端中执行`webdev`命令了。

### 新建一个 Flutter for Web 项目

在 VS Code 中打开命令面板后输入`flutter web`则会自动提示你让你新建一个 web 程序，然后输入项目名即可创建一个 web 项目。

![](https://cdn.ytools.xyz/uPic/1240-20230116112647491.jpeg)

然后执行`flutter packages get`即可安装依赖。

### 启动你的第一个 web 项目

现在来执行最后一个命令来运行项目：

```
webdev serve
```

终端的输出结果如下：

![](https://cdn.ytools.xyz/uPic/1240-20230116112654364.jpeg)

我们打开浏览器并输入：`http://127.0.0.1:8000`，然后我们就可以在浏览器上看到神奇的结果了：

![](https://cdn.ytools.xyz/uPic/1240-20230116112656344.jpeg)

> 目前发现 Firefox 和 Chrome 均可运行，Safari 无法显示界面，原因还有待查找。

回顾代码我们可以发现 Flutter for Web 项目的 main.dart 和普通的 Flutter 项目的代码几乎一致：

![](https://cdn.ytools.xyz/uPic/1240-20230116112700033.jpeg)

唯一的区别就是第一行中引入的 fltter_web 库了。

因为对 Flutter for Web 我也没过多了解，这一部分推荐你去查看[官方文档](https://github.com/flutter/flutter_web)了解更多关于我们上面执行的命令或者网页的信息。

## 总结

Flutter 的发展势头和谷歌想要让 Flutter 一统所有平台的野心大家是有目共睹的，就在前不久，Fuchsia OS 的官网也悄然上线，作为新系统的指定开发工具，Flutter 自然备受关注。目前 Flutter for Mobile 已经发展的挺好了，虽然配置 Desktop 应用和 Web 应用时仍有些繁琐，开发时仍会有许多 bug，但冰冻三尺非一日之寒，我们应该给予足够的耐心。

**参考文章**

1. [在 macOS 上运行 Flutter 桌面端项目](https://zhuanlan.zhihu.com/p/52218677)
2. [在 macOS 上安装和配置 Flutter 开发环境](https://flutter-io.cn/docs/get-started/install/macos)
3. [Flutter for Desktop: Create and Run a Desktop Application](https://medium.com/flutter-community/flutter-for-desktop-create-and-run-a-desktop-application-ebeb1604f1e0)
4. [Getting started with Flutter Web](https://blog.usejournal.com/getting-started-with-flutter-web-e187829c9dd3)
5. [Desktop Embedding for Flutter](https://github.com/google/flutter-desktop-embedding)
6. [在 Mac 安装 Dart](http://dart.goodev.org/install/mac)

---

![](https://cdn.ytools.xyz/uPic/1240-20230116112707934.jpeg)
