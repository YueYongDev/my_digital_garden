---
{"title":"ISE头条号海报生成器","tags":["python"],"dg-publish":true,"permalink":"/实战教学/ISE头条号海报生成器/","dgPassFrontmatter":true}
---


![](https://cdn.ytools.xyz/uPic/006tKfTcgy1g0vtglskw9j30zk0npn4j.jpg)

> 完整源码可在公众号：「01二进制」后台回复：「海报生成器」获取

今天来分享一个日常生活中经常见到，但是制作起来又可能会一时没有思路的东西，主要功能就是生成带二维码的卡片或者海报。

之前莫名其妙的被导师安排负责管理实验室的头条号，任务不难，就是接收实验室学长学姐翻译转述的论文，然后再发布到今日头条的头条号上，最后再生成如下所示的宣传图即可：

![](https://cdn.ytools.xyz/uPic/006tKfTcgy1g0vtqxipsfj30jp0pz400.jpg)

当时觉得，不就是发发文章，然后再用**ps**做个图这么简单吗。可接手之后才发现我毕竟图样图森破啊，从去年11月我开始发文章到今天，期间从未有一天断过，但是这头条号的编辑器也从未更新过，一个这么大的自媒体企业，文章的编辑器竟然烂的跟坨💩一样，不支持外部图片，不支持markdown，不支持数学公式，不支持多级标题。（别跟我说什么可以把markdown转成html然后再复制进头条号的编辑器里面，样式都变成鬼了）

扯远了扯远了，回到正题。之前这么多天实现上述需要的主要流程如下：

1. （采取各种方式优化排版）把文章发布到（不支持各种常用功能的）头条号上
2. 文章发布后，获取其文章链接，并到草料二维码生成器网站，上传实验室logo后生成二维码下载至本地
3. 利用PhotoShop将封面图、文章标题和文章二维码合成在一起后发给老师。

在经历了100天上述这样重复的操作之后，我厌烦了。难道就没有一个工具可以让我只输入文章链接和标题就自动生成海报的吗？

苦苦寻觅半天无果，也罢，有条件要上，没条件创造条件也要上。没有现成的轮子，那就只能自己打造一个了，Python无疑是开发这个小工具的首选。

一般用于推广的海报或卡片样式都差不多，需要改变的主要就是二维码，所以只需要准备好海报的背景图，然后根据用户提供的二维码，将其贴在海报指定的位置上即可。

此次实验的项目结构如下：

![](https://cdn.ytools.xyz/uPic/006tKfTcgy1g0vusz8kw8j30ak07wt8y.jpg)

> assets文件夹中包含一些资源文件，例如`msyhl.ttc（字体文件）`、`template.jpg（背景模版图片）`。output是生成的海报存放的路径

## 生成带logo的二维码

本次生成二维码依赖于 **PIL 模块**和 **qrcode 库**，官方地址为：https://pypi.org/project/qrcode/5.1/，这里不解释用法，感兴趣的自己去官方文档下了解。这里就直接上代码了，具体代码的用意详见注释：

```python
# 生成一个带logo的二维码
def generateQRCode(url):
    # 初始化
    qr = qrcode.QRCode(
        version=5, error_correction=qrcode.constants.ERROR_CORRECT_H, box_size=8)
    # 添加内容
    qr.add_data(url)
    qr.make(fit=True)

    img = qr.make_image()
    img = img.convert("RGBA")

    # 读取logo
    icon = Image.open("assets/logo.jpg")
    # 设置logo
    img_w, img_h = img.size
    factor = 4
    size_w = int(img_w / factor)
    size_h = int(img_h / factor)

    icon_w, icon_h = icon.size
    if icon_w > size_w:
        icon_w = size_w
    if icon_h > size_h:
        icon_h = size_h
    icon = icon.resize((icon_w, icon_h), Image.ANTIALIAS)

    # 将logo并入原二维码中
    w = int((img_w - icon_w)/2)
    h = int((img_h - icon_h)/2)
    icon = icon.convert("RGBA")
    img.paste(icon, (w, h), icon)
    rgb_im = img.convert('RGB')

    # 保存到指定路径下
    today = datetime.date.today()
    folder_path = 'output/'+str(today)
    mkdir(folder_path)
    rgb_im.save(folder_path+'/qr.jpg')
```

## 生成海报

我们先来梳理下，想要生成一张满足我们需求的海报需要哪些元素：

* 二维码（qrImg）
* 背景模版图片（template.jpg）
* 文章标题（postTitle）
* 和文章有关的封面图（postPic）

> 换一种方式呈现代码，推荐一个将代码转换成图片的美化工具**Carbonize**

![](https://cdn.ytools.xyz/uPic/006tKfTcgy1g0vvdgbhbyj30u00xxqep.jpg)

其实仔细阅读过这段代码之后才觉得整体的思路一目了然：先读取需要的素材文件（二维码、标题），然后将素材粘贴到背景图片的指定位置。唯一复杂点的就是要找到一个合适的粘贴点，这个没办法，只能自己去试。

> 完整源码可在公众号：「01二进制」后台回复：「海报生成器」获取

## Tips

这里有点我要提下，就是利用`PIL`更改图片大小那块，也就是`postPic.thumbnail((width/1.5, height/1.5))`这个地方，其实PIL中还有一个方法叫做`resize`也是用来更改图片的大小的，那两者有何区别呢？

使用PIL生成缩略图用两种方式，`resize`和`thumbnail`,区别在于使用`reszie`会返回一个新对象，
而使用`thumbnail`则会在原对象上进行修改，即`thumbnail`会覆盖原图。

```python
>>> from PIL import Image
>>> im = Image.open('a.jpg')
>>> im.size   # 原图尺寸
(3264, 2448)
>>> id(im)
140253860921640
>>> resize_im = im.resize((100,100))
>>> resize_im
<PIL.Image.Image image mode=RGB size=100x100 at 0x7F8F65A0A518>
>>> id(resize_im)
140253862077720
>>> thumb_im = im.thumbnail((100, 100))
>>> thumb_im
>>> im.size   # 使用thumbnail后的原图尺寸改变，resize后的结果不一定等于指定的尺寸，因为是按比例缩放的
(100, 75)
```

