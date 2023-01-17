---
{"title":"你真的需要Apple TV吗？不如自制一个Android TV！","tags":["树莓派"],"dg-publish":true,"permalink":"/一些想法/你真的需要Apple TV吗？不如自制一个Android TV！/","dgPassFrontmatter":true}
---


没有一个树莓派能逃离吃灰的命运。

去年我写了一篇[[实战教学/树莓派家用指北\|《树莓派家用指北》]]，介绍了树莓派是如何作为家庭服务器改善我的生活的，指路链接 👇

https://mp.weixin.qq.com/s/JZR2gFnIFoYDG7Z9HwsS3A

	今天我们的主角依旧是我的那个树莓派，只是以另一种形式在我的家里发光发热——电视盒子。

看到这可能有人会好奇，这个树莓派用作电视盒子后，原先的家庭服务器怎么办？事实上我之所以把这个树莓派做成电视盒子，第一个原因是我搬家了，需要重新升级规划下家里的软件系统，第二是我用 NAS 替代了原先的树莓派用作家庭服务器（有机会的话以后讲一下）。

所以为了不让这个“理财产品”就这么吃灰下去，我一直积极探索可能的用处，终于，我发现了一个最适合他的场景——Android TV。

## 为什么会想到 Android TV？

搬家之后，新屋子里有一个电视 + 办宽带送的中国移动的电视盒子，第一次打开它的时候，卡顿的系统、上古的 UI、繁杂的广告让我不禁感叹，这真的是 2022 年的东西吗？

心想那要不买一个 Apple TV 吧？可再看看价格，不免囊中羞涩，算了算了，还是留点老婆本吧。况且 Apple TV 这么好的盒子用在一个只有 1080P 的电视里属实是有些浪费了。

![](https://cdn.ytools.xyz/uPic/e6c9d24ely1h5erszgg0oj213g0jo424.jpg)

既然用不了苹果的电视服务，用安卓的总可以吧。

于是我又去搜索了一些国内的电视盒子，什么小米的、当贝的、荣耀的，横向比较了一下，不是性能孱弱（通常都是 2GB+32GB）就是广告遍地，而且还不能看海外电视。当然了，最关键的是还要多花一笔钱，想了想还是放弃了。

既然国内的安卓盒子不行，为什么不试试原生的 Android TV 呢？2022 年了，原生的 Android TV 应该有不少的发展了吧。

抱着试一试的心态，我打开了 Android 开发者的官网，发现 2022 年的 Android TV 无论是 UI 还是体验，都比以前有了长足进步，现在就差一个载体，而我刚好有这个最好的载体——树莓派。

![](https://cdn.ytools.xyz/uPic/e6c9d24ely1h5erwzzb3vj21e20ow44a.jpg)

## 行动起来

本文不是一个教学贴，因此不会手把手的记录整个流程，简单介绍一些我在自制这一过程中的关键点以及可能出现的问题。参考的帖子：[https://konstakang.com/devices/rpi4/LineageOS18-ATV/](https://konstakang.com/devices/rpi4/LineageOS18-ATV/)

### 准备工作

你需要准备的东西有：

- 一个树莓派 3B/4B，至少有 2GB RAM，建议 32GB+ 的 SD 卡（我的是 8GB RAM + 256GB ROM）
- 一根 mini HDMI 转 HDMI/DP/VGA 数据线（根据你家的电视接口定）
- 树莓派风扇（如果有最好，毕竟是 24 小时不关机的，散热还是有必要的）

我选择的是 konstakang 提供的 LineageOS 18.1 Android TV (Android 11)，没有选择上 Android 12 的原因是当时还没有出 12 的 GApps（谷歌提供的一些套件），再加上一个电视盒子也没必要追求那么新的操作系统（国内还有不少手机停在 Android 10 万年不更新）

rom 地址在 👉[https://www.androidfilehost.com/?fid=17825722713688273838](https://www.androidfilehost.com/?fid=17825722713688273838)

给树莓派刷入安卓系统的方法和刷入其他系统的方法基本一致，建议直接使用 Raspberry Pi Imager 烧录系统。工具地址 👉https://www.raspberrypi.com/software/

然后就和安卓刷机没什么区别了。

### resize 你的 SD 卡

![](https://cdn.ytools.xyz/uPic/e6c9d24ely1h5erzjw9byj217o07ign6.jpg)
刚烧录的 Android TV 系统会出现不正常分区的问题，我们需要将 SD 卡上的空白空间都利用起来，执行 resize 的流程也很简单，只需要使用 TWRP 将提供的 resize.zip 刷入系统即可。流程和安卓刷机是一样的，需要借助一个叫做 TWRP 的工具，有安卓刷机经验的小伙伴应该很了解这个步骤。有关 TWRP 的介绍这里就不展开了，移步 👉https://twrp.me

![](https://cdn.ytools.xyz/uPic/e6c9d24ely1h5erzegcufj20zk0k03zs.jpg)

resize.zip 的下载地址：[https://androidfilehost.com/?fid=2981970449027577728](https://androidfilehost.com/?fid=2981970449027577728)

### root & GApps 的安装

![](https://cdn.ytools.xyz/uPic/e6c9d24ely1h5es2au9qij20xc0m8tae.jpg)

既然选择了自建 Android TV，肯定是希望可以享受到一些海外的优质媒体服务，那么谷歌套件就必不可少了。

我们需要借助 magisk 实现 root，然后刷入一些谷歌套件 GApps（需要科学上网），通常我们会选择 OpenGApps（感谢开源 🙏）

rpi-magisk 地址：[https://androidfilehost.com/?fid=2981970449027577730](https://androidfilehost.com/?fid=2981970449027577730)

OpenGApps for andriod tv：[https://opengapps.org/?arch=arm64&api=11.0&variant=tvstock](https://opengapps.org/?arch=arm64&api=11.0&variant=tvstock)

## 其他实用技巧

### SSH 连接你的 Android TV

#### 打开开发者选项

#### 连接电视盒子

```bash
adb connect 192.168.2.134
```

#### 以 root 方式访问 adb

![](https://cdn.ytools.xyz/uPic/e6c9d24ely1h5es6esg7aj219q08yac0.jpg)

```bash
adb root
```

#### 从 android tv 获取 ssh 访问的 private key

```bash
adb pull /data/ssh/ssh_host_rsa_key my_private_key
```

#### 文件添加权限

> 因为该文件是利用 adb 下载得到的，利用该文件执行 ssh 命令时会提示权限过高，因此需要设置权限为 400
>
> ![](https://cdn.ytools.xyz/uPic/e6c9d24ely1h5es5nkga9j20wo08q40n.jpg)

```bash
chmod 400 my_private_key
```

#### ssh 连接树莓派

![](https://cdn.ytools.xyz/uPic/e6c9d24ely1h5es5acp79j215o0ojjvu.jpg)

```bash
ssh -i my_private_key root@<你的树莓派地址>
```

### 查看当前 cpu 温度

通过查看你的树莓派运行时的温度，来决定是否需要为其加装一个风扇

```bash
cat /sys/class/thermal/thermal_zone0/temp
```

![](https://cdn.ytools.xyz/uPic/e6c9d24ely1h5es4wufsaj20o2028q2y.jpg)

## 最后

目前这个 Android TV 服务在我家里已经运行了有半年了，配合着家里的软路由实现入口级的科学上网以及 NAS 用作家庭媒体中心，最后利用小爱音箱红外版接入了小米智能家居，已经可以实现不逊色于 Apple TV 的娱乐服务。不信，你看看我家的猫咪都爱上了看电视呢！

![](https://cdn.ytools.xyz/uPic/e6c9d24ely1h5es4jvahdj20qs0mywgq.jpg)
