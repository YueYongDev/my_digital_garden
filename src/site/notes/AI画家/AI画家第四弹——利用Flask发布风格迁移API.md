---
{"title":"AI画家第四弹——利用Flask发布风格迁移API","tags":["AI画家,TensorFlow,Flask"],"dg-publish":true,"permalink":"/AI画家/AI画家第四弹——利用Flask发布风格迁移API/","dgPassFrontmatter":true}
---


![](https://cdn.ytools.xyz/uPic/1240-20230116113745631.jpeg)

上篇文章介绍了python web开发中经常使用到的一个框架flask，如果有遗忘的，可以点此回顾👉[[AI画家/AI画家第三弹——毕业设计大杀器Flask\|《AI画家第三弹——毕业设计大杀器之Flask》]]，本文的主要任务就是完成上篇文章末尾的要求，利用Flask发布你自己的风格迁移API。

> 本文源码可在微信公众号「01二进制」后台回复「风格迁移API」获得

## 需求分析

我们知道软件工程的第一步就是需求分析，放在这里就是要知道我们需要实现的功能是什么样的。我画了一张简陋的图来描述这次的需求：

![](https://cdn.ytools.xyz/uPic/1240-20230116113749843.jpeg)

真的是很简陋的一张图啊，其实理解起来很容易，就是用户上传一张图片，Flask获取到这张图片，调用风格迁移的模型，然后生成结果图，在传递回前端即可。

## 环境准备

既然明白了需求，那么接下来要做的事情自然是环境搭建了，老样子，这里我们仍然使用Pipenv来创建虚拟环境，如何搭建pipenv环境我就不说了，在微信公众号「01二进制」后台回复「风格迁移API」获得源码之后直接在终端输入`pipenv install`即可。

## 开始

### hello world

我们首先在项目根目录创建一个main.py的文件作为整个项目的启动文件，上文我们说过，为了简化大型应用并为扩展提供集中的注册入口，我们并不会将所有的视图函数直接写在main.py，而是采用蓝图的方式分模块开发，因此我们需要在项目根目录新建`app/`文件夹，在其中的`__init__.py`中编写如下代码：

```python
from flask import Flask

def create_app():
    app = Flask(__name__)
    return app
```

这样我们就可以通过在main.py中编写如下代码实现一个hello world应用了。

```python
from app import create_app

app = create_app()

@app.route('/')
def hello():
    return 'hello,world'

if __name__ == '__main__':
    app.run(port=8080, debug=True)
```

启动main.py即可发现项目启动了，在浏览器输入`localhost:8080`即可看到`hello,world`字样。

### 蓝图编写

我们肯定是不能满足于小小的hello world的，既然说到了模块化开发，那怎么个模块法？

这里每个人的想法都是不一样的，其实也没有一个统一的标准，这里我就说下我自己的分级方法吧。

![](https://cdn.ytools.xyz/uPic/1240-20230116113753764.jpeg)

- stylize是项目根路径
- app是项目
- app/api是项目的api部分，一个项目肯定不只有api，还可能会有web等页面内容
- app/api/v1表明该api的版本是v1，当然日后也有可能会有v2、v3等等
- app/api/v1/img表明这里存放的都是和img有关的api
- app/api/v1/img/stylize.py表明这个文件存放的是风格迁移的视图函数

认识完结构的划分之后，就来编写我们的蓝图吧。

首先我们需要在`app/api/v1/img/__init__.py`中编写如下代码：

```python
from flask import Blueprint

# 定义一个蓝图
img = Blueprint('img', __name__)

from app.api.v1.img import stylize

# 这段代码用来测试该接口是否可用
@img.route('/')
def say_hello():
    return '这里是图片处理类的接口'
```

这样我们就定义了一个叫做img的蓝图，然后我们在`app/api/v1/__init__.py`中编写如下代码：

```python
from flask import Blueprint

# 定义一个蓝图
v1 = Blueprint('v1', __name__)

from app.api.v1.img import img
```

这样我们就实现了v1蓝图的编写。

那这样是不是就可以使用蓝图了呢？当然不是，我们还需要在app中配置这个蓝图，把蓝图加载到app中，否则flask是无法识别蓝图的。加载的方法也很简单，我们在`app/__init__.py`文件中添加一个函数：

```python
def register_blueprint(app):
    from app.api.v1 import v1
    from app.api.v1.img import img

    app.register_blueprint(v1, url_prefix='/api/v1')
    app.register_blueprint(img, url_prefix='/api/v1/img')
```

将这两个蓝图注册到app中，其中`url_prefix`这个参数用来标注路由的。有人可能不清楚，这里举个例子你就懂了。我们启动这个项目之后，在浏览器输入的是`localhost:8080`，如果加了`url_prefix`这个参数之后，我们访问img下的视图函数时就需要把路径改为`localhost:8080/api/v1/img/`了。

然后在`create_app()`函数中调用这个方法即可，main.py的代码如下：

```python
from flask import Flask


def create_app():
    app = Flask(__name__)

    register_blueprint(app)
    return app


def register_blueprint(app):
    from app.api.v1 import v1
    from app.api.v1.img import img

    app.register_blueprint(v1, url_prefix='/api/v1')
    app.register_blueprint(img, url_prefix='/api/v1/img')


if __name__ == '__main__':
    create_app()
```

接下来我们测试下这个蓝图是否真的注册成功了，我们启动该项目，并打开Postman输入：`localhost:8080/api/v1/img`，我们可以看到如下信息：

![](https://cdn.ytools.xyz/uPic/1240-20230116113801516.jpeg)

说明我们的蓝图已经注册

### 编写风格迁移工具类

既然蓝图都已经编写好了，那么视图函数的编写也就非常简单了，因此这里我们先把风格迁移的工具类写好，最后再编写视图函数。

上上篇文章我们介绍了图像风格迁移，记不清的可以看这篇文章👉[[AI画家/AI画家第二弹——图像风格迁移\|《AI绘画第二弹——图像风格迁移》]]，这篇文章介绍了最传统的图像风格迁移，想要生成一张图片的速度非常非常慢，肯定是没办法作为实际使用的，因此这篇文章所采用的生成风格迁移的图片的方法并不是这篇文章，而是基于李飞飞等人的一篇论文《Perceptual Losses for
Real-Time Style Transfer and Super-Resolution》所实现的快速图像风格迁移。这里只介绍如何拿训练好的模型去运用，有兴趣的自己下载这篇文章去研究。

我们新建一个文件夹：`app/utils/stylize`，这个文件夹中包含了风格迁移的工具类，项目结构如下：

![](https://cdn.ytools.xyz/uPic/1240-20230116113808314.jpeg)

其中output为最终的生成文件夹、src存放风格迁移的文件（可以不用管）、evaluate.py是之前用于评估模型性能的文件（其实也就是生成图片的文件，不用管+1）、trained_model文件夹存放了我们已经训练好的模型、temp存放的是我们从前端上传的图片，create_stylize_photo.py包含的是我们对外提供的风格迁移工具类。

因此这整个文件夹我们只需要关注`create_stylize_photo.py`这一个文件就可以了，其他的下载源码之后自己看就可以了。

#### 压缩图片

```python
# 压缩图片
def compress_image():
    im = Image.open(content_image)
    if content_image.endswith(".png"):
        im = im.convert('P')
    im.save(content_image, optimize=True)
```

#### 选择图片风格即所需要使用的模型

```python
style_list = ['la_muse', 'rain_princess', 'scream', 'udnie', 'wave', 'wreck']
style = style_list[int(image_style)]

# 模型的 checkpoint 的位置
check_point_dir = trained_models_path + style + '.ckpt'
```

#### 执行生成图片的操作

```python
# 最终生成的图片路径
result_image = path + '/output/' + 'output.jpg'

# 执行生成图片的操作
ffwd_to_img(content_image, result_image, check_point_dir)
```

执行完上述步骤后，我们就可以在`utils/stylize/output`中看到已经生成的风格迁移图片了。

### 利用七牛云存储结果图片

由于服务器带宽限制，我们最后返回给前端的结果肯定不能是我们自己服务器的url（毕竟学生机的带宽只有1M），所以这里我建议使用七牛云存储的功能将生成的结果保存到七牛云上，然后返回一个url即可。

```python
# 将生成的图片上传到七牛云
# 传入filename和filepath，返回图片的URL
def upload_pic_to_qiniu(filename, filepath):
    from app.secure import QINIU_AK
    from app.secure import QINIU_SK

    access_key = QINIU_AK
    secret_key = QINIU_SK

    q = Auth(access_key, secret_key)

    # 要上传的空间
    bucket_name = 'ytools'

    # 生成上传 Token，可以指定过期时间等
    token = q.upload_token(bucket_name, filename, 3600)

    ret, info = put_file(token, filename, filepath)
    return BASE_URL + ret['key']
```

然后我们在`create_stylize_photo.py`中加一句存储的代码即可。

```pytho
img_url = upload_pic_to_qiniu(filename, result_image)
```

### 编写风格迁移API

现在我们通过风格迁移工具类已经可以实现输入一张原始图片返回生成图片的URL的功能，现在我们来将目光聚焦到风格迁移API的编写上。

#### 定义路由

```python
@img.route('/stylize/create', methods=['POST'])
def create_style_changed_img():
```

#### 接收参数

我们定义一个函数`create_style_changed_img()`，方法采用POST方式，我们需要接受前端发来的两个参数，分别是img和type

```python
img = request.files.get('img')
type = request.form.get('type')
```

#### 生成图片及URL

然后我们将接收到的文件保存到之前新建的temp文件加中，然后调用工具类的方法返回图片的url

```python
img.save(path + '/temp/' + 'temp.jpg')

img_url = change_style(int(type))
```

#### 定义返回格式

作为一个好的api，我们肯定不只能返回一张图片的url就可以了，我们还需要记录下生成的时间，因此我们在代码执行的开始和结束的时候分别添加一段代码：

```python
start = datetime.datetime.now()
end = datetime.datetime.now()
```

然后我们再定义返回格式

```python
status = 200
msg = '图片生成成功'
info = [
    {
        'img_url': img_url,
        'created_time': get_date_now(),
        'finish_time': (end - start).seconds
    }
]
```

然后将结果返回

```python
res_json = Res(status, msg, info)

return jsonify(res_json.__dict__)
```

这里的Res是我定义的一个返回的实体信息类，长得是下面这样

```python
class Res:
    status = 200
    msg = ''
    info = []

    def __init__(self, status, msg, info):
        self.status = status
        self.msg = msg
        self.info = info
```

最后我们调用flask的jsonify方法，就可以返回json结果。

### 测试

到这里我们就已经完成了一个风格迁移API的编写，接下来我们测试下我们的API吧，首先先启动项目，然后打开Postman，将请求方法改为post，添加两个参数img和type，如下：

![](https://cdn.ytools.xyz/uPic/1240-20230116113818829.jpeg)

> 选择图片的时候，图片的质量尽量不要太大，否则可能会出现卡死的情况

最后的返回结果如下：

```json
{
    "info": [
        {
            "created_time": "2019-05-15 15:17:25",
            "finish_time": 2,
            "img_url": "https://user-gold-cdn.xitu.io/2019/5/15/16aba682a31521a7?w=640&h=360&f=jpeg&s=39700"
        }
    ],
    "msg": "图片生成成功",
    "status": 200
}
```

访问该URL即可看到如下图片（感觉还蛮好看的）：

![](https://cdn.ytools.xyz/uPic/1240-20230116113822316.jpeg)

至此我们已经实现了利用Flask发布一个风格迁移API了。

## 总结

最后总结下，在这篇文章中介绍了如何利用Flask发布一个风格迁移API，其中我们介绍了应该如何利用蓝图进行模块化开发，并给出了我自己认为的比较好的分层方法，同时利用七牛云存储为我们的服务器减压，最后利用postman请求该API完成测试。
