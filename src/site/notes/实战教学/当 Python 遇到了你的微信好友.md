---
{"title":"当 Python 遇到了你的微信好友","categories":"实战教学","tags":["数据分析"],"dg-publish":true,"permalink":"/实战教学/当 Python 遇到了你的微信好友/","dgPassFrontmatter":true}
---


临近毕业，慢慢的也感伤起来，回想大学这几年，除了技术的成长，最值得庆幸的就是结交了一帮志同道合的好友。后期自己做了公众号，微信好友的数量也越来越多，身边人所扮演的角色也越来越丰富，有早已结婚生子为人父母的同学，有沉迷科研学术的教师，当然也少不了一众还在 996 的程序猿。事实上，你所处圈子的质量很大程度上就决定了你的人生质量，那么今天我们就来看看当 Python 遇到了你的微信好友后能擦出怎样的火花。

## 前言

这次我们直奔主题，本文要做的是以下几件事：

1. 分析微信好友的总人数、男生数、女生数、男女比
2. 分析好友的地域分布
3. 利用 **自然语言处理** 的方法分析出你好友的情感倾向
4. 获取微信好友的头像并拼接成指定图片

### 准备

还是老样子，做实验前，先做好准备工作，实验环境如下：

- Python 3.6 （虚拟环境的管理为 Pipenv）

- Pycharm

- 主要使用到的包有：
  - itchat
  - pyecharts
  - baidu-aip
  - photomosaic
  - pillow

> 对 Pipenv 这个虚拟环境管理工具不熟悉的可以去看我之前的文章：[《Python 管理哪家强？》](https://mp.weixin.qq.com/s/KBLgdsL2UaXfayT0Hdi04A)，里面对于 Pipenv 这个虚拟环境管理工具有一些介绍。
>
> itchat 是一个开源的微信个人号接口，可以让我们使用 python 来调用微信
>
> pyecharts 是 python+echarts 的结合，用于进行数据的可视化
>
> baidu-aip 是百度推出的一个 nlp 的包
>
> photomosaic 是用来生成蒙太奇马赛克图片的

大家获取到源码之后只需要将 **Pipfile** 复制到你们的项目根路径下，然后再终端执行 `pipenv install` 即可创建一个安装好所有包的虚拟环境了（前提是你的电脑上已经安装了 pipenv 了）

做好准备工作后我们就开始吧。

## 开始

### 1. 初始化 itchat

只需一行代码即可初始化 itchat：

```python
itchat.auto_login(hotReload=True)
```

hotReload(热加载)，True 表示其短时间内不需要再次扫码登陆

### 2. 获取好友列表

同样的也只需要一行代码即可获取：

```python
friends = itchat.get_friends(update=True)[0:]
```

返回的数据是类 JSON 格式的，我们用 Python 可以很方便的解析，因为篇幅原因，返回的示例我就不展示了，你们自己输出查看就可以了。

### 3. 分析男女分布情况

首先我们需要获取好友的性别信息，通过分析返回的 JSON 字符串我们发现，在好友的信息中有 Sex 标签，其规律是当其值为 1 是表示男生，2 表示女生，0 表示没有填写的，因而我们可以这样

```python
# 对好友数进行分析
def analyze_friends_num(friends):
    # 初始化性别的变量(男、女、其他，其他表示的是注册时没有填写性别信息的)
    male = female = others = 0
    # 循环得到的全部好友
    # 在好友的信息中有Sex标签,发现规律是当其值为1是表示男生,2表示女生,0表示没有填写的
    for i in friends[1:]:
        sex = i['Sex']
        if sex == 1:
            male += 1
        elif sex == 2:
            female += 1
        else:
            others += 1
    # 总人数
    total = len(friends[2:])
    print("总人数为", total, "其中男性", male, "人，女性", female, "人，男女比为", round((male / female), 2), ":1")
```

执行的结果为：

```
总人数为 387 其中男性 228 人，女性 116 人，男女比为 1.97 :1
```

更加直观的显示如下（可视化的代码在 **utiils** 包下，这里就不放出了，有需要的自己看源码）：

![friends_num](https://cdn.ytools.xyz/uPic/1240-20230116151219715.jpeg)

我的好友里男女比竟然是 2:1（那些没填性别信息的人里面不知道还有多少男生），活该没有女朋友啊，看来下次要多加一些女生的微信了。

### 4. 好友地域分布

这里我们只要获取到好友的地域信息，然后用两个 _dict_ （分别是省和市）保存即可，key 为地域， value 为该地域的好友数，循环遍历 friends 最后用饼图表示分布最多的 5 个省，用柱状图表示分布最多的 15 个市，代码如下：

```python
# 分析好友的地域分布
def analyze_friends_location(friends):
    province = {}
    city = {}
    for i in friends[1:]:
        if i['Province'] == '':
            i['Province'] = '其他'
        if i['City'] == '':
            i['City'] = '其他'

        province[i['Province']] = province.get(i['Province'], 0) + 1
        city[i['City']] = city.get(i['City'], 0) + 1
    sorted_province = sorted(province.items(), key=lambda item: item[1], reverse=True)
    sorted_city = sorted(city.items(), key=lambda item: item[1], reverse=True)

    # 画出分布图
    draw_friends_location(sorted_province[0:5], sorted_city[0:15])
```

结果如下：

![](https://cdn.ytools.xyz/uPic/1240-20230116151223459.jpeg)

![](https://cdn.ytools.xyz/uPic/1240-20230116151245365.jpeg)

看到这大家应该也能猜到我主要的活动区域是哪了吧，有兴趣的大家可以猜一猜然后在文末留言哦。

### 5. 好友情感分析

当你想要了解一个人心态（注意是心态而不是动态）的时候，你往往都会去看他的签名而不是朋友圈，因为签名更改的频率很低，很大程度上会反映这个人的情绪和心态。相比之下，朋友圈更新的频率较高，因为是要分享自己近期的动态的（我就见过有的女生一条朋友圈分成好几条发，每次只发几个字）。因此对好友的签名进行分析是可以分析出她的情绪的，那么我们该如何分析情感呢？

这里实名夸奖一下百度，作为国内技术的老大哥，很久之前百度就已经**免费开放**了他的一些人工智能接口，其中就有情感倾向分析，官网是<https://ai.baidu.com/tech/nlp_apply/sentiment_classify>，这些**免费的**人工智能接口的开放对于我们这些个人开发者无疑是个福音。下面是他的功能演示截图：

![](https://cdn.ytools.xyz/uPic/1240-20230116151252099.jpeg)

他的用法也很简单，安装好*baidu-aip*包之后，申请下 appkey、appid 和 secretkey 后即可使用：

```python
client = AipNlp(APP_ID, API_KEY, SECRET_KEY)

def analyze_text(text):
    res = client.sentimentClassify(text.strip())
    return res['items'][0]['sentiment']
```

因此我们要做的无非就是获取好友的签名，然后传入`analyze_text`函数即可：

```python
# 分析好友的签名
def analyze_friends_signature(friends):
    positive = 0
    negative = 0
    others = 0

    print('签名情感分析中，请稍后......')
    for index, item in enumerate(friends[1:]):
        text = item['Signature']
        if text != '':
            try:
                print('正在分析第', index, '条签名：', text, '他的作者是：', item['NickName'], '你给他的备注是：', item['RemarkName'])
                res = analyze_text(text)
                if res == 0:
                    negative = negative + 1
                if res == 1:
                    others = others + 1
                if res == 2:
                    positive = positive + 1
            except:
                continue
```

看到这有人会有疑问了，我的好友人数有上千，免费的接口能用这么多次吗？事实上，他真的可以用这么多次 😂

![](https://cdn.ytools.xyz/uPic/1240-20230116151300156.jpeg)

看到这我突然想给百度打 call，这也太良心了吧。请问贵公司还缺实习生吗，我想去应聘 😂。然后我们来看看我的好友的情绪分析图吧。

![](https://cdn.ytools.xyz/uPic/1240-20230116151309408.jpeg)

没想到我的好友里面竟然还有 17.83%的人有消极情绪，看来必要的时候得"we need to talk"了。

### 6. 利用好友的微信头像生成指定的照片

看标题你们可能不懂是什么意思，直接放图你们就明白了：

![](https://cdn.ytools.xyz/uPic/1240-20230116151319638.jpeg)

这张图远看是一张一个人跳舞的图片，其实仔细看就知道了，构成这张图的是我的 300 多张微信好友的图像，这里我使用到了一个名为`photomosaic`的库，它是专门用来制作这种蒙太奇马赛克风格的图片的，是我无意中在知乎上看到的，所以大家有事没事还是逛逛知乎，多少能发现些好玩意。

接下来我们来看看如何生成上述图片。

**第一步，我们先获取好友的头像：**

```python
# 获取好友头像
def get_friends_avatar(friends):
    for index, item in enumerate(friends):
        print("正在下载第 %d 张头像" % index)
        img = itchat.get_head_img(userName=item["UserName"])
        file_image = open(os.getcwd() + "/app/temp/" + item["UserName"] + ".jpg", 'wb')
        file_image.write(img)
        file_image.close()
```

也很简单，直接调用 itchat 的`get_head_img`方法然后保存到本地指定文件夹下即可。

**第二步，利用 photomosaic 生成目标图片**

```python
# 利用好友头像生成蒙太奇马赛克图片
def draw_friends_mosaic_image():
    # 读取基准图，即要生成的蒙太奇马赛克图片的原始图
    image = pm.imread(os.getcwd() + '/assets/cxk.jpg')
    # 定义图片库
    pool = pm.make_pool(os.getcwd() + '/temp/*.jpg')
    # 制作50*50的拼图马赛克,(50, 50)是指每一行和每一列使用图片库中的图像的个数
    mosaic = pm.basic_mosaic(image, pool, (50, 50))
    # 保存制作好的图片
    pm.imsave(os.getcwd() + '/output/friends_mosaic_image.jpg', mosaic)
```

四行代码即可，原理的话知乎上有写，有兴趣的可以自己去搜一搜。

当然了，不是每个人的微信好友都有上千人，所以拼接出来的效果就不是很好，比如我自己的那个就不是很好，既然这样的话我就推荐另一个拼接头像的方法，不过效果要稍微差点，拼成的图长这样：

![friends_avators](https://cdn.ytools.xyz/uPic/1240-20230116151329947.jpeg)

这里的代码来自我关注的一个公众号，就不放出代码了，大伙感兴趣的可以去看下他的文章：[《一键拼出你的微信好友图片墙》](https://mp.weixin.qq.com/s/T22CF1y1urvlCQwv3yxRVA)

这张图虽然观赏效果不如上一张，但好在每个头像都很清楚，大伙儿看看能不能快速找到自己的头像呢？欢迎在留言区评论互动哦～

## 最后

方法教给大家了，图片素材可以优化，大家可以生成自己喜欢的蒙太奇图片，发到朋友圈，让代码骚动起来吧！
