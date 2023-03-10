---
{"dg-publish":true,"cover":"https://cdn.ytools.xyz/uPic/xzlGcO1240-20230113010350410.jpeg","permalink":"/技术科普/从电商入手理解什么是CDN/","dgPassFrontmatter":true}
---

## 前言

相信很多人在制作自己的第一个网站的时是很激动的。我们知道，在一个网站项目中，页面里经常会有许多 JavaScript 以及 CSS 的引用，如果是直接引用项目内文件的话，他们可能是这样的：

![](https://cdn.ytools.xyz/uPic/6ZgivE1240-20230113010317615.jpeg)

这种方式的优点是开发省力，发布省力，对服务器要求小，省钱，没有具体公网接入需求。

然而如果你的网站里面有很多图片或者视频并且需要部署到公网上时，网站的访问速度一定会让你倍感崩溃。就像下面这张图 👇

![](https://cdn.ytools.xyz/uPic/T3ZS1Zstrip-20230113010328520.gif)

这时候肯定会有推荐你使用 CDN 来加速网站里的一些 JavaScript 和 CSS 文件，如下所示：

![](https://cdn.ytools.xyz/uPic/aDE09D1240-20230113010331689.jpeg)

其实上面的图片就已经使用到 CDN 了。那到底什么是 CDN 呢？

在解释什么是 CDN 之前，我们先来看一个身边非常常见的案例—— **网购**。

## 京东自营与淘宝的购物体验

相信现在应该没有人没用过淘宝和京东的，在说 CDN 之前我们先来说下我在淘宝和京东的购物体验。下面是我在使用这两个电商平台时的情况：

- 在淘宝买第三方店家商品
- 在京东购买自营商品

之前我在淘宝买了一个雷电 3 的扩展坞，发货地是深圳，花了三天才到南京，如果收货地是河南呢？新疆呢？我想时间就更长了。可是我在河南的同学在京东（自营）买了一个手机下午购买的第二天早晨就收到货了（并不是给京东打广告）。这是为什么呢？

我们在用京东购物的时候，如果仔细观察的话可以发现，京东自营会根据我们的收货地点，在全国范围内找离我们最近、送达最快的仓库，比如我在南京下的订单，他可能就会从上海甚至直接从南京发货；如果是在洛阳下单可能就会从郑州发货。这样做的好处就是不管我们在南京，还是乌鲁木齐，我们的收货时间会大大减少。**CDN** 就类似于京东建立的这种仓储系统。

## 从网购到 CDN

不知道上面的描述是否清楚，这里为了加深理解，我制作了下面的流程对比图：

![](https://cdn.ytools.xyz/uPic/QoZCQ81240-20230113010340783.jpeg)

为了让货物更快的送到买家手中，京东建立了这种仓储系统；类比到网络中，为了让用户更快地加载网页（可以理解为服务器给浏览器送页面），CDN 横空出世了。

CDN 的全称是 **Content Delivery Network**，即**内容分发网络**。其基本思路是尽可能避开互联网上有可能影响数据传输速度和稳定性的瓶颈和环节，使内容传输的更快、更稳定。通过在网络各处放置节点服务器所构成的在现有的互联网基础之上的一层智能虚拟网络，CDN 系统能够实时地根据**网络流量和各节点的连接、负载状况以及到用户的距离和响应时间**等综合信息将用户的请求**重新导向**离用户**最近**的服务节点上（如下图所示）。其目的是使用户可就近取得所需内容，解决 Internet 网络拥挤的状况，提高用户访问网站的响应速度。

![](https://cdn.ytools.xyz/uPic/xzlGcO1240-20230113010350410.jpeg)

## 从“直播”理解一些和 CDN 有关的名词

从上面的描述中我们得知了 CDN 的作用以及大概原理，但是其中的细节并没有展开来说。其实 CDN 的一些细节通常会和一些名词联系上，例如**负载均衡**、**源站**之类的。同样的，我们以一个身边的例子——**“直播”**——来讲解这些和 CDN 有关的名词。

我们知道，视频其实是由一帧一帧的图片组成的，所以直播的时候我们收到的视频画面的流程可以近似理解为下面这样 👇

![](https://cdn.ytools.xyz/uPic/ReMzI41240-20230113010353694.jpeg)

然而事实是这样的吗？当然不是！一个主播怎么可能只有一个观众，所以应该是下面这样 👇

![](https://cdn.ytools.xyz/uPic/GZNh5p1240-20230113010357061.jpeg)

上图的方式是主播把相同的数据同时传给多个不同的观众，这当然是非常愚蠢的方式，**同样的数据**被传了多次，主播端的瓶颈非常明显，比如有 1000 个观众同时观看的时候，主播端根本无法承担这么多的数据传输。

所以很容易想到的一个方式就是在主播和用户之间增加一个性能**非常强悍**的服务器充当**中间人**的角色，从服务器把数据发给不同用户，也就是下面这样 👇

![](https://cdn.ytools.xyz/uPic/dcca2v1240-20230113010400669.jpeg)

这里的服务器主要有两个作用：1. 接收来自主播的数据（推流）；2. 将收到的数据分发给用户（分发）。

> 当然，如果这里的服务器性能过于强悍，那么它除了可以执行推流和分发的作用外，还可以实现美颜、特效、鉴黄等功能。这时，这台服务器就又多了一个身份——**流媒体处理中心**。

可是一台服务器的性能也是有上限的，假设一台服务器最多可以支持 1000 个用户同时观看的话，如果用户数远远大于 1000 的话又该怎么办呢？相信读到这里的小伙伴一定都知道可以怎么做了，没错，那就是再添加一层服务器（集群），如下图所示 👇

![](https://cdn.ytools.xyz/uPic/UCo1FX1240-20230113010404360.jpeg)

在上图中，服务器 0 负责接收主播的视频数据，然后传递给服务器 1、2、3……，然后再由这些服务器分发给用户。考虑到用户之后有可能还会访问这些数据，所以他们就干脆把数据在服务器 1、2、3……上都存储了一份。

### 相关名词

接下来我们从上面的描述中来理解一些概念。

**负载均衡**

当观众人数不太多的时候，例如总共只有 1000 人，那么是选择让某一台服务器服务这 1000 人，还是 3 台服务器分担 1000 人，还是 2 台？机器也会有新旧之分，老机器只能抗 800 数量，那要怎么来分配呢？等等问题。这里就需要有一个策略来做资源的分配。这个策略叫做：负载均衡。负载均衡通常可以利用**重定向、反向代理等**方式实现，常用的负载均衡算法有**轮询法、随机法、最小连接数法等**（篇幅问题，这里不再阐述）。

**CDN 缓存**

考虑到用户之后有可能还会访问这些数据，所以他们就干脆把数据在服务器 1、2、3……上都存储了一份（最简单的例子就是多个用户可能会在不同的时间段访问同一张图片）。这个概念叫做：CDN 缓存。

**回源、源站、边缘节点**

当分配到服务器 1 的第一个观众进入时，服务器 1 是没有存储数据的，它会向服务器-0 获取数据，这个过程叫做：回源；相应的，服务器-0 被称为：源站；服务器 1、2、3……这些负责内容分发的被称为边缘节点。

**缓存命中/缓存命中率**

观众请求的数据如果由 CDN 缓存提供，叫做缓存命中，所有用户请求的缓存命中比例叫做缓存命中率，它是衡量 CDN 质量的关键指标。

**就近原则**

一名新进入的观众会被分配到哪一台服务器上呢？理论上，这台服务器距离用户的网络链路越短、不跨网，数据的传输的稳定性就越好，这个叫做：就近原则。

## CDN？对象存储？

通过上面的介绍，我们知道 CDN 的主要目的就是为了加速访问，服务对象主要是**_直播、点播、网页静态文件、小文件等_**。这时候可能就会有人问了，为了加速一些小文件的访问，我也会使用一些厂家的对象存储服务，例如阿里的 OOS，百度的 BOS 等。那对象存储和 CDN 又有什么区别呢？

的确，这两者的目的其实都是加速用户的访问，但是侧重点完全不同。CDN 的重点在于**分发**，对象存储的重点在于**存储**。可以把对象存储简单理解为网盘，CDN 是高速公路。

以图片存储为例，**对象存储是存图片的，CDN 是加速下载图片的。**所以在很多情况下，二者是配合使用的，目前这一套组合也已经成为互联网应用的一个必不可少的组成部分。

## 使用 CDN 的好处

说了这么多，如果只是为了加速网站的访问速度，完全可以选择其他方式，为什么一定要用 CDN 呢？或者说，除了可以加速，CDN 还有什么好处？

1. 有利于搜索排名。谷歌等搜索引擎已经把网站访问速度作为一个结果排名的重要指标了。
2. 网站不容易宕机。其实这就和把鸡蛋放在很多篮子里是一个道理，多个服务器分流之后，源站的压力就会小很多。
3. 减少托管成本。大多数服务器的带宽都是有限制的，分流之后不同的文件被存放在不同的服务器上，可以减少带宽产生的费用。

## 怎么使用 CDN

怎么使用 CDN 是个比较难回答的问题，因为如果要自己搭建一套 CDN 服务难度非常大，但如果只是想要使用的话，有很多大厂都有自己的 CDN 服务，不同厂家都有不同的收费标准和特性，这个就因人而异了，具体使用看各家的文档即可。一般在 html 中使用的时候我会直接去 BootCDN 上复制粘贴下需要使用的库。

## One More Thing

虽然有句话叫做“凡是能通过怼硬件来实现的都别浪费时间做软件优化”，但是从上面的解释中我们也能得知，CDN 的能力和那些边缘节点有很大关系，假设硬件投入已经饱和了，还有什么方式可以加速整个访问流程呢？

压缩传输数据！如果可以实现将传输过程中的数据更进一步的压缩同时保持信息不变，那传输过程中各个节点所需要承担的压力就会小很多了，自然而然就可以加速访问了。

我最喜欢的美剧之一《硅谷》讲述的就是主人公 Richard 开发出的一种开创性的“通用压缩算法”，并以此创业的故事。按照剧里的描述，该算法可以改变已有的互联网世界。遗憾的是该剧中的算法现实中并不存在，但这并不影响这是一部十分优秀的美剧，推荐每一个从事 IT 领域的人观看。

## 最后

鉴于笔者能力有限，如果有什么疑问或建议可以在评论区留言。新来的小伙伴不妨给个关注，你们的支持我前进的最大动力 💪

![](https://cdn.ytools.xyz/uPic/vxguLo1240-20230113010412219.jpeg)
