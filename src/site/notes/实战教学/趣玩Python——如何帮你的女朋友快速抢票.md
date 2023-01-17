---
{"title":"趣玩Python——如何帮女朋友快速抢票","tags":["其他,python"],"dg-publish":true,"permalink":"/实战教学/趣玩Python——如何帮你的女朋友快速抢票/","dgPassFrontmatter":true}
---


又到了半年一度的考试季，对于那些翻山越岭外出求学的莘莘学子们，相比于各显神通的考试，更紧张的莫过于买一张回家的车票，相信很多群最近都被下面这样的图占领了。

![](https://cdn.ytools.xyz/uPic/007S8ZIlgy1gibbgrfhdkj306o07qt8p.jpg)

![](https://cdn.ytools.xyz/uPic/007S8ZIlgy1gibbgsup3tj306o0833yl.jpg)

如今，随着 12306 的抗压能力越来越强，各种第三方抢票软件也是层出不穷，什么智行火车,携程旅游，就连官方都推出的了加速服务，这就导致了大量黄牛都开始感叹：这年头的生意不好做咯！

而且现在各家的抢票方式都是八仙过海，各显神通，这家让你消费买加速包，那家让你疯狂推销，以至于才出现了上述加速小程序的疯狂炸群（微信小程序恐成最大赢家）。

作为一个苦逼的学生党，花钱买加速包不大可能，毕竟买加速包的钱都快赶上半张火车票了；让我疯狂用小程序炸群也不大可能，毕竟关系到自己的社交信誉，而且现如今的群成员各个都是大爷，不发红包不点加速。

那么难道就没有一种 geek 风的抢票软件吗？

## 12306 购票小助手

想找各种骚操作的软件，第一想法自然是去最大的同性交友网站啊，无意中发现了一个名为 12306 购票小助手的项目，试了下竟然真的抢到了票，项目已经开源，地址 👉[https://github.com/testerSunshine/12306](https://github.com/testerSunshine/12306)

### 思路图

![思路图](https://cdn.ytools.xyz/uPic/007S8ZIlgy1gibbh7duetj30nn0ixwfc.jpg)

作者也很用心的把程序的思路给画了出来，我们可以简单的看一下。整个思路其实就是模拟一个正常人购票的方式，首先查询下车票剩余的票数，如果有座位提交订单，出现验证码这识别验证码，随后就循环点击提交按钮，这里作者就做了很多的条件判断，比如出现异常则重新查询，提交订单失败也重新查询，直至获取订单成功。订票成功之后还有一个通知机制，即发送到你的邮箱里。

### 使用

说了这么多，应该如何使用呢？详细的可以参考作者的 README，这里我用最简单的方式讲述下需要注意的地方以及如何使用用这个购票小助手抢到票：

**注意事项**

1. python 版本为**2.7.10-2.7.15**
2. 推荐使用 MacOS/Linux
3. 使用时一定要以 root 用户运行

**准备工作**

1. 注册若快图像识别<http://www.ruokuai.com/client/index?6726>，记住用户名和密码，然后充值 1 块钱兑换 2500 快豆即可，该步骤是为识别验证码做准备。
2. 下载项目：执行`git clone https://github.com/testerSunshine/12306.git`将代码下载至本地。
3. 安装 Python2.7：此处推荐使用 pyenv 管理你的 python 版本，Mac 用户可以参考[[实战教学/mac下利用pyenv管理多个版本的python\|《mac 下利用 pyenv 管理多个版本的 python》]]安装制定版本的 python 版本，这里我使用的是 python 2.7.15
4. 安装依赖库：命令行进入项目根目录后，执行`sudo python2 -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple -r requirements.txt`

**项目配置**

![配置文件](https://cdn.ytools.xyz/uPic/007S8ZIlgy1gibbhqzcqpj30d80de0tc.jpg)

上图中的**`ticket_config.yaml`**是运行整个项目最重要的配置文件，所有的购票信息都在该文件中，比如车票时间，12306 账号密码，乘车人信息，通知邮箱等等，文件中都有详细的注释，根据要求进行更改即可。

![](https://cdn.ytools.xyz/uPic/007S8ZIlgy1gibbhq3sdtj30f00j6my6.jpg)

把这个配置文件按你的需求填写完毕之后，就可以开始运行了。

**开始抢票**

命令行进入项目根目录后，执行`sudo python run.py`即可开始抢票了。

![](https://cdn.ytools.xyz/uPic/007S8ZIlgy1gibbhw7kibj30vo0kan37.jpg)

如果抢到票了，就会输出类似下面的 log：

```
车次: DXXX 始发车站: 南京南 终点站: 合肥南 二等座: 16
设置乘车人数为: 1
查询到有余票，尝试提交订单
车票提交通过，正在尝试排队
排队成功, 你排在: 0位, 当前余票还剩余: 16 张
不需要验证码
提交订单成功！
排队等待时间预计还剩 -4 ms
恭喜您订票成功，订单号为：XXXXXX, 请立即打开浏览器登录12306，访问‘未完成订单’，在30分钟内完成支付！
```

然后再登录 12306 的官方网站，访问‘未完成订单’即可看到你的购票信息了。

![](https://cdn.ytools.xyz/uPic/007S8ZIlgy1gibbi28j7tj30yg08y74n.jpg)

最后祝愿大家都能抢到回家的票！
