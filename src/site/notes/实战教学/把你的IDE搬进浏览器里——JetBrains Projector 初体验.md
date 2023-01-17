---
{"title":"把你的IDE搬进浏览器里——JetBrains Projector 初体验","categories":"实战教学","tags":["云计算"],"abbrlink":6511,"dg-publish":true,"permalink":"/实战教学/把你的IDE搬进浏览器里——JetBrains Projector 初体验/","dgPassFrontmatter":true}
---


![](https://cdn.ytools.xyz/uPic/gR6ziC%E4%BA%91%E7%AB%AF%E7%BC%96%E7%A8%8B.png)

## 前言

对于云端编程，我想大多数人的第一想法应该是微软推出的 VSCode Remote，这个功能基于开源的 VSCode，通过 SSH 远程连接到服务器，开发者可以通过端口转发、SCP 等一系列实用功能快速实现**远程开发**。我曾体验过这种编程方式，极大减轻了电脑性能的压力，但我认为这并不是云端编程的最终形态，因为我仍然需要在自己的电脑上安装 VSCode 才可以使用这个功能。

最近 2021 款的 iPad Pro 上市了，这次搭载的是和 Mac 同款的 M1 芯片，性能强大到甚至有让人利用 iPad 编程的想法，只是迫于各大厂商没有推出适配 iPad 的 IDE，便也只能沦为“买钱生产力，买后爱奇艺”的工具了。那么有没有什么办法可以在不安装 IDE 的情况下使用 iPad 编程吗？自然是有的，JetBrains 公司提出了一种新的解决方案：把 IDE 搬进浏览器里。这便是本文的主角——JetBrains Projector。

![](https://cdn.ytools.xyz/uPic/ncqLVhBlog_1280x720.png)

## 发展

提起 JetBrains，你会想到什么？各路强大的 IDE，比如 Android Studio、IDEA、WebStorm……这些对于开发者来说耳熟能详的产品都出自这家公司，这些 IDE 的功能强大，但同时也只能运行在用户自己的电脑上，其“内存黑洞”的称号更是让开发者们又爱又恨。

![](https://cdn.ytools.xyz/uPic/wt6DBiiShot2021-05-04%2019.46.53.png)

事实上，目前所有的 JetBrains IDE 都使用 Java Swing 绘制 UI，其他基于 IntelliJ 的 IDE 也是如此，比如 Android Studio。鉴于 Swing 是 Java GUI 的一个库，而 Java 本身就是一门很吃内存的编程语言，虽然可以充分利用 Java 跨平台的特性，这也是这些 IDE 在 macOS、Windows 和 Linux 上 UI 几乎一致的原因。但现在，Swing **跨桌面平台**的特性却也成为阻碍其发展的一个原因了，在一些瘦客户端的情况下，“内存黑洞”屡屡被人诟病，Swing 也无法发挥其优势，于是 Projector 便应运而生了。

## 横向比较

![](https://cdn.ytools.xyz/uPic/AcZUVvAWS_Cloud9_Asset01_R3_P.22c006faf1258710ffbdd756ec83ea97449e9da3.png)

JetBrains Projector 是 JetBrains 提出的“远程开发”解决方案，基于 Client + Server 架构，虽然对标的是微软 VSCode 的[Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)方案，但是二者的原理和体验效果还是相差很多的。

- VSCode 通过 SSH 等技术，只传输代码、索引等数据，仅将计算匀给服务器，而渲染显示等还是依赖本地的 VSCode 客户端，这种情况下，你**仍然需要安装 VSCode**。
- Projector 改动了 Swing 的渲染机制，通过网络传输渲染指令，最终使用 Web 技术将界面展现出来。这样做的好处是，你可以直接使用**浏览器**访问安装在服务器上的 IDEA。

笔者在查阅资料的过程中发现，经常有很多人将这两者弄混，通过上述内容，相信你也有一个直观的感受了，这两者使用体验的差距类似于**VNC 与 SSH**之间使用体验的差距。因此笔者认为这其实是对于「云端编程」的两种不同的解决方案，针对的使用场景虽有交叉，但很多情况下是不一样的，因此并不会有哪一方会完全取代另一方的情况出现。

## 使用场景

既然上文已经提到了，JetBrains Projector 和 VSCode Remote 的使用场景并不相同，那这一节就来简单说说 Projector 特别适合解决的问题：

1. 在**运行时或数据库**附近运行代码以减少往返次数。
1. **高度安全**的企业环境。
1. 真正的**大型项目**。
1. 禁止**源代码**本地复制。
1. 用户**硬件约束**。
1. **瘦客户端**。
1. 需要在 Windows 机器甚至是 ChromeOS 等**非传统操作系统**上的 GNU/Linux 环境中运行 IDE。
1. 需要在**关闭计算机**后让应用在服务器上继续运行。
1. **远程调试**服务器端（devtest、devprod）。
1. 具有调试源和预配置 IDE 的**VM 或 Docker 映像**。
1. 需要**远程访问的配置**。

Note：Projector **不支持**协作开发。

## 初体验

前文说了 JetBrains Projector 是基于 Client + Server 架构的，因此为了体验 Projector，我们需要分别安装 Client 端和 Server 端。

### Server 端

![](https://cdn.ytools.xyz/uPic/MzUr2viShot2021-05-04%2019.57.11.png)

官方给出了三种搭建 Server 端的方式，分别是：

1. [Docker 镜像](https://github.com/JetBrains/projector-docker)：Docker 是在云环境中运行 Projector 的最简单的方法，需要额外安装 Docker 环境。不需要额外安装 IDEA，如果只是为了体验，推荐该方式。
1. [Python 脚本](https://github.com/JetBrains/projector-installer)：通过 PyPi 安装，这是一个独立的发行版，目前仅适用于 GNU/Linux 主机。
1. [IDE 插件](https://github.com/JetBrains/projector-server)：需要有图形界面的电脑支持并运行 Jetbrains IDE，通过安装 Projector 插件来作为服务端。
   > PS：个人觉得第三种方式有多此一举的嫌疑，既然远程服务器都已经具备图形界面了，那我直接使用 VNC 不就好了吗？

搭建过程很简单，这里选择 Docker 搭建 Projector 服务，直接选择以下几个命令安装指定 IDE 即可

```bash
docker pull registry.jetbrains.team/p/prj/containers/projector-clion
docker pull registry.jetbrains.team/p/prj/containers/projector-datagrip
docker pull registry.jetbrains.team/p/prj/containers/projector-goland
docker pull registry.jetbrains.team/p/prj/containers/projector-idea-c
docker pull registry.jetbrains.team/p/prj/containers/projector-idea-u
docker pull registry.jetbrains.team/p/prj/containers/projector-phpstorm
docker pull registry.jetbrains.team/p/prj/containers/projector-pycharm-c
docker pull registry.jetbrains.team/p/prj/containers/projector-pycharm-p
docker pull registry.jetbrains.team/p/prj/containers/projector-webstorm
```

例如，这个代码段可以拉取 IntelliJ IDEA Community Edition：

`docker pull registry.jetbrains.team/p/prj/containers/projector-idea-c`

然后运行镜像，执行以下命令，将<IMAGE_ID>换成刚刚下载完成的镜像 ID 即可。

`docker run --rm -p 8887:8887 -it <IMAGE_ID>`

出现以下内容说明 Server 端安装成功 👇

![](https://cdn.ytools.xyz/uPic/udOlTlimage.png)

### Client 端

JetBrains 官方给出两种 Client 端的使用方式，一种是直接通过浏览器访问，另一种是使用官方开发的客户端。

官方客户端的地址在：[https://github.com/JetBrains/projector-client](https://github.com/JetBrains/projector-client)

#### 浏览器访问

我们先通过浏览器访问[http://localhost:8887/](http://localhost:8887/.)，同意 Policy 后便可以看到如下页面 👇

![](https://cdn.ytools.xyz/uPic/RPXJopiShot2021-05-04%2017.03.25.png)

显示效果和本地的 IDEA 几乎没有差别，当然了，使用体验也还是和服务器的性能有很大关系。

#### 官方 Client App

我们打开官方提供的客户端后填入刚才的地址便会显示同样的效果。

![](https://cdn.ytools.xyz/uPic/Fkl5VeiShot2021-05-04%2017.11.55.png)

在简单阅读了这个官方 App 的源码后发现这个 Desktop App 其实是基于 Electron 的，有趣的是，虽然使用的是自家的 Kotlin 语言编写，但不知道为什么不顺便使用自家的**Compose for Desktop**。

#### 浏览器访问的一些缺点

虽然通过官方 App 使用 Projector 很方便，但说到底我还是要下载一个应用程序，既然都这样了，和 VSCode Remote 也没什么区别，我为什么不直接使用浏览器访问呢？
其实官方文档中已经针对这个疑问做了详细的[说明](https://jetbrains.github.io/projector-client/mkdocs/latest/ij_user_guide/accessing/#known-issues)：

1. iPad 不支持 self-signed WebSockets，即不安全的 Websockets 连接（较新的安卓其实也不支持），因此想利用 iPad 访问局域网内的 Projector 会有些麻烦，当然了，你给服务器添加 HTTPS 访问也是可以的。
1. 一些快捷键会被浏览器拦截，例如，Windows/Linux 中的 Ctrl+Q 或 Mac 中的 Cmd+N 是由浏览器处理的。这可能会导致你在使用 Projector 无法使用一些快捷键。
1. 剪切板不同步，服务端的剪切板会有一些限制，使得开发过程中的复制与粘贴会出现一些问题。

也正是因为上述这些问题，官方才推出了自己的客户端 App。

## 头脑风暴——VSCode on Browser

通过上述的介绍，相信你对 JetBrains Projector 已经有了一定的了解了，其原理就是改变 Swing 的渲染方式，使其最终可以使用 Web 技术将界面展现出来。

这时候，我们可以头脑风暴一下，既然 VSCode 基于的 Electron 技术本质上是让运行在浏览器中的网页可以顺畅的运行在桌面端，那么是不是可以进行一个**“逆向”**，将运行在桌面的 VSCode 反向运行在浏览器中呢？这种方式的思路不同于 VSCode Remote，反而和 JetBrains Projector 有些类似，答案当然是可以的，国外也已经有大神将源码开源出来了，详情参考 👉[https://github.com/cdr/code-server](https://github.com/cdr/code-server)，这里就不再多说了，感兴趣的读者们可以自行查阅相关资料。

![](https://cdn.ytools.xyz/uPic/eSZbixiShot2021-05-04%2017.55.49.png)

## 最后

先前我曾在[《云游戏在革谁的命？》](https://mp.weixin.qq.com/s/yvprXpCfIIVvm3P6eaO1vg)一文中讨论过云游戏对于传统游戏的影响以及是否会取代 PC 游戏，其实云端编程和云游戏类似，都需要高速且稳定的网络，在这个万物上云的时代，只有解决了这两点才可以真正将云游戏和云编程推广，我相信这一天很快就会到来。

以上就是本篇文章的全部内容了，如果觉得对你有所帮助，不妨点个赞，新来的读者不妨给个关注支持一下，你们的支持是我继续更文的最大动力。
