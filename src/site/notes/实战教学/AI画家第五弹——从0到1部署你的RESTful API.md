---
{"title":"AI画家第五弹——从0到1部署你的RESTful API","tags":["AI画家,TensorFlow,Flask"],"dg-publish":true,"permalink":"/实战教学/AI画家第五弹——从0到1部署你的RESTful API/","dgPassFrontmatter":true}
---


![](https://cdn.ytools.xyz/uPic/1240-20230116113943718.jpeg)

上一篇文章中我们已经[[实战教学/AI画家第四弹——利用Flask发布风格迁移API\|利用 Flask 制作了一个 RESTful API]]，然而文中末尾我们我们将代码运行起来的时候确发现是 localhost:xxxx，这也就意味着我们没法通过外网访问我们提供的 API，所以这样做出来的程序即使部署到服务器上也没有任何用处。这一篇我们就来详细的说下怎么部署到服务器上然后实现外网访问吧。

## 环境准备

工欲善其事，必先利其器。还是老样子，既然要选择部署到服务器上，我们首先肯定得要有一个服务器，这里我选择 XX 云的学生机（别问为什么，问就是便宜）。系统选择 Ubuntu18.04，需要安装的东西有：

- Python 3.6
- Nginx
- pyenv
- pipenv
- Flask + Gunicorn

这里我们用 pyenv 管理不同的 python 版本，这里可能就有人要问了，Ubuntu 不是自带 python2 和 python3 吗，为什么还要用 pyenv 来管理，之前的文章我们也说过，用 pyenv 管理 python 会非常的方便，方便到只需要一行代码就可以安装切换系统的 Python 版本，废话不多说，接下来就开始我们的操作吧。

### 连接到远程服务器

如何购买 XX 云的云主机我就不多说了，当你买好之后我们在后台查看到我们购买的云主机的外网 IP，比如我的服务器的外网 IP 地址就是`94.191.9.43`（希望各位大佬手下留情，别弄他）：

![](https://cdn.ytools.xyz/uPic/1240-20230116113954394.jpeg)

接下来就是连接到服务器了，工具有很多，xshell、putty 或者 terminal 都可以，这里我推荐一个工具**Termius**，推荐他的原因有两个：**颜值**，作为一个颜控，Termius 是我见过的最好看的 SSH 客户端，没有之一。同时还是跨平台的，win/mac/linux/android/ios 都支持，这点不知道比 xshell 高到哪里去了。

![5c7eb7590c28be6ee5c8afdb_termius](https://cdn.ytools.xyz/uPic/1240-20230116113958411.jpeg)

连接也很简单，我们在 termius 中添加需要连接的 host，然后输入用户名和密码即可登录了，出现如下字样说明登录成功。

![](https://cdn.ytools.xyz/uPic/1240-20230116114000652.jpeg)

### 利用 pyenv 安装制定版本的 python

之前的文章介绍过如何在 mac 下安装 pyenv，虽然 linux 下的安装方法类似，但毕竟本文的题目是是从 0 到 1 部署，还是再详细说一下吧。

- 下载安装 pyenv

```shell
curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash
```

- 先查看是否存在`~/.bash_profile`文件

```shell
cat ~/.bash_profile
```

- 如果没有就新建该文件

```shell
touch ~/.bash_profile
```

- 然后执行以下命令将 pyenv 添加到系统变量中

```shell
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
source ~/.bash_profile
```

接下来执行`pyenv`命令，如果出现下图则说明安装成功。

![](https://cdn.ytools.xyz/uPic/1240-20230116114008574.jpeg)

接下来我们利用 pyenv 来安装不同版本的 python，只需要一行命令：

```
pyenv install xxx（版本号）
```

比如说我们想安装 python 3.6.7，只需要执行`pyenv install 3.6.7`即可。这一步可能有点慢，主要看服务器的网速和性能了。

如果出现安装失败的话，可以尝试执行下面代码安装一些依赖库后再重新安装 python：

```shell
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
xz-utils tk-dev libffi-dev liblzma-dev
```

出现如下字样说明安装成功。

![image-20190527155431084](https://cdn.ytools.xyz/uPic/1240-20230116114014293.jpeg)

接下来我们执行`pyenv versions`查看我们系统中目前已有的 python

```
* system (set by /home/ubuntu/.pyenv/version)
  3.6.7
```

执行`pyenv global 3.6.7`将 3.6.7 作为系统默认的 python 环境，执行下 python 命令查看系统 python 版本，如下图所示我们已经将系统的默认 python 版本切换为 3.6.7 了。

![image-20190527155707337](https://cdn.ytools.xyz/uPic/1240-20230116114019000.jpeg)

### 安装 pipenv 管理项目的虚拟环境

- 先升级 pip

```shell
pip install --upgrade pip
```

- 然后安装 pipenv

```shell
pip install pipenv
```

最后执行 pipenv 查看是否安装成功。

他的具体用法参考我之前的一篇文章 👉[[实战教学/Python 管理哪家强\|《Python 管理哪家强？》]]

### 将项目上传至服务器

其实到这里项目环境基本算是搭建完成了，不过也只是刚刚能用，离我们最终需要搭建的环境还差点东西，这个后期再说，先来把项目上传到我们的服务器上，这里我选择的工具是 FileZilla，一个开源免费的 FTP 客户端（而且颜值也不错），新建站点后添加服务器 IP，输入用户名和密码后点击连接即可连接到远程服务器，如下图所示。

![](https://cdn.ytools.xyz/uPic/1240-20230116114022828.jpeg)

连接成功后我们可以在工具右侧看到服务器的文件列表，我个人比较喜欢在/home/ubuntu 下新建一个 dev 文件夹专门用来存放项目的代码，方便管理，如下所示。

![](https://cdn.ytools.xyz/uPic/1240-20230116114027227.jpeg)

我们在工具左侧选择要上传的文件，拖到指定文件夹中即可实现文件上传。这里我讲代码存放到`/home/ubuntu/dev/python/stylize`路径下了，我们也可以通过 terminal 查看到文件。

![](https://cdn.ytools.xyz/uPic/1240-20230116114034233.jpeg)

> 这个代码在上篇文章中有提到，可在微信公众号「01 二进制」后台回复「风格迁移 API」获得

## 实现外网访问

### 最原始的办法

我们已经将项目代码上传至服务器了，命令行进入该文件夹，然后执行`pipenv install`创建虚拟环境并下载所需依赖。接下来再执行`pipenv shell`进入虚拟环境，这时我们可以看见路径前会有一个括号，里面是虚拟环境的名字，如下所示。

```
(stylize) ubuntu@VM-0-10-ubuntu:~/dev/python/stylize$
```

对上一节提供的`main.py`代码，我们需要对 main 函数进行一些修改：

```python
if __name__ == '__main__':
    app.run(host='0.0.0.0',port=8080, debug=app.config['DEBUG']
```

相对于上一节内容，此次修改添加了`host='0.0.0.0'`这一参数，这一参数的的含义就是实现 WSGI 服务器的外网访问，如果不添加这个 host，默认只能内网访问，这时我们再执行`python main.py`启动整个项目，出现如下效果说明服务已经启动了。

![](https://cdn.ytools.xyz/uPic/1240-20230116114040336.jpeg)

这时候我们在浏览器中输入`ip:8080`即可看到效果，例如我这的输入的是http://94.191.9.43:8080，出现的结果如下：

![](https://cdn.ytools.xyz/uPic/1240-20230116114046793.jpeg)

> ⭐️ 需要注意的是，你需要在你购买的云服务的控制台设置安全组，以便给你的服务器开放相应的端口

至此我们就实现了 flask 的应用在服务端的部署以及实现外网访问了。

### 这样就结束了吗？

但是这样就结束了吗？当然不是，如果我们把连接断掉，这个服务就没办法执行。这时可能就有人会说了，那在执行`python main.py`这一命令前加一个`nohup`不就可以实现后台运行了吗？没错，这样确实可以实现后台运行，但是在实际的生产环境中是不会有人使用这样的方式部署的，原因很简单，**Flask 是一个 web 框架，而非 web server，直接用 Flask 拉起的 web 服务仅限于开发环境使用，生产环境不够稳定，也无法承受大量请求的并发，在生产环境下需要使用专业的服务器软件来处理各种请求**。既然 flask 自带的服务器性能很差，那我们自然要清楚专业的服务器，这里的可选项有 Gunicorn、 Nginx 或 Apache，通常情况下，我们选择 Gunicorn+Nginx 这样的组合。

### Gunicorn 是什么？

这里简单回答两个问题，Gunicorn 是什么以及为什么是 Gunicorn。

第一个问题，Gunicorn 是什么？

Gunicorn 中文名"独角兽"，一个开源 Python WSGI UNIX 的 HTTP 服务器。

第二个问题，为什么是 Gunicorn？

效率高，使用方便。

### 为什么是 Gunicorn+Nginx？

在[[实战教学/AI画家第三弹——毕业设计大杀器Flask#应用服务器、Web服务器、Restful\|《AI 画家第三弹——毕业设计大杀器之 Flask》]]这篇文章中我们曾提过[[实战教学/AI画家第三弹——毕业设计大杀器Flask#^fc8b19\|web 服务器]]和[[实战教学/AI画家第三弹——毕业设计大杀器Flask#^a9e73a\|应用服务器]]这两种服务器，其中应用服务器既可以解析静态资源，也可以解析动态资源，但是解析静态资源的能力不如 web 服务器。而 Nginx 就是 web 服务器，Gunicorn 是应用服务器，采用这样的组合，一方面基于 Nginx 转发 Gunicorn 服务，在生产环境下能补充 Gunicorn 服务在某些情况下的不足，另一方面，如果做一个 Web 网站，除了服务外，还有很多静态文件需要被托管，这是 Nginx 的强项，也是 Gunicorn 不适合做的事情。所以，基于 Flask 开发的网站，部署时用 Gunicorn 和 Nginx，是一个很好的选择。

### 那为什么需要 Nginx 转发 Gunicorn 服务？

按照上面的解释，其实直接使用 Gunicorn 就完全可以实现外网访问了，为什么非要加一层 Nginx 呢？

原因其实也很简单，Nginx 功能强大，用 Nginx 转发 Gunicorn 服务，重点是解决“慢客户端行为”给服务器带来的性能降低问题；另外，在互联网上部署 HTTP 服务时，还要考虑的“快客户端响应”、SSL 处理和高并发等问题，而这些问题在 Nginx 上一并能搞定，所以在 Gunicorn 服务之上加一层 Nginx 反向代理，是个一举多得的部署方案。

> 如果想深入了解 Nginx/Gunnicorn 在项目中扮演的角色，推荐阅读 👇
>
> Nginx、Gunicorn 在服务器中分别起什么作用？ - 知乎
>
> https://www.zhihu.com/question/38528616

### 开始实现

#### 启动 Gunicorn

上面说了那么多基础知识和原理，赶紧来实际操作下吧，首先安装 Gunicorn，安装方法很简单，在 pipenv 虚拟环境中，我们直接运行以下代码就可以了。

```python
pipenv install gunicorn
```

然后运行

```
gunicorn -D -w 3 -b 127.0.0.1:8080 main:app
```

这样就可以启动一个 web 服务，这里我们来解释下这些参数。

1. gunicorn：命令的名称，不多解释
2. -D 表示后台运行
3. -w 表示线程
4. -b 指定 ip 和端口（建议使用本地端口，方便 nginx 进行代理。）
5. main 是项目的启动文件（main.py）
6. app 是全局变量 （`app = Flask(__name__)`）

虽然说上述命令也不是很难，但是每次都输这么长的命令难免会出错，有没有什么简单的方法呢？答案自然是有的，我们可以将这些命令的参数以**配置文件**的形式保存到本地。

首先在项目根目录下新建一个`gunicorn.conf`，然后写入下面的信息：

```
# 并行工作线程数
workers = 4
# 监听内网端口8080【按需要更改】
bind = '127.0.0.1:8080'
# 设置守护进程【关闭连接时，程序仍在运行】
daemon = True
# 设置超时时间120s，默认为30s。按自己的需求进行设置
timeout = 120
# 设置访问日志和错误信息日志路径
accesslog = './logs/acess.log'
errorlog = './logs/error.log'
```

作用就如注释所言。这时我们再启动项目只需要执行下面的命令即可

```
gunicorn -c gunicorn.conf main:app
```

> 记得提前创建好 logs 这个文件夹哦 😊

#### 关闭 Gunicorn

经历过上述操作后我们就可以启动一个项目了，那该如何关闭呢？

很简单，分两步

**step1：**

```bash
pstree -ap|grep gunicorn
```

显示结果：

```bash
(stylize) ubuntu@VM-0-10-ubuntu:~/dev/python/stylize$ pstree -ap|grep gunicorn
  |-gunicorn,9349/home/ubuntu/.local/share/virtualenvs/stylize-rKP1K2-f/bin/gun
  |   |-gunicorn,9352/home/ubuntu/.local/share/virtualenvs/stylize-rKP1K2-f/bin/gun
  |   |-gunicorn,9353/home/ubuntu/.local/share/virtualenvs/stylize-rKP1K2-f/bin/gun
  |   `-gunicorn,9354/home/ubuntu/.local/share/virtualenvs/stylize-rKP1K2-f/bin/gun
  |                       |-grep,9365 --color=auto gunicorn
```

**step2：**

```bash
kill -9 9349
```

-9 表示强制关闭，9349 表示运行的进程号（根据你自己的服务器情况来定）

### 安装配置 Nginx

在 Ubuntu 上安装 Nginx 很简单，一条命令就可以了：

```shell
sudo apt-get install nginx
```

接下来就是配置了，nginx 的默认配置文件在`/etc/nginx/sites-enabled/default`中，执行`sudo vim default`，修改下面部分的代码

```
server {
    listen 80;
    server_name localhost;
    location /{
        proxy_pass http://127.0.0.1:8080;
    }
}
```

这里我们解释下，修改上述内容的作用是：将外部通过**80 端口**发来的请求，代理给 127.0.0.1:8080 端口。还记得我们在`gunicorn.conf`中设置了 flask 的启动地址为`127.0.0.1:8080`，到这里终于派上了用场。

然后执行以下命令重启 nginx

```shell
sudo nginx -s reload
```

这时我们再访问http://94.191.9.43，就不用添加端口号了。

![](https://cdn.ytools.xyz/uPic/1240-20230116114059376.jpeg)

如果你有域名的话，在后台把域名解析到这个 ip 上就可以实现域名访问了，是不是很有趣呢？

当然在 Gunicorn 上再加一层 Nginx 服务，除了实现反向代理，另一个原因就是用它配置 HTTPS 非常方便，这里我就不多说了，如果读者有兴趣的话可以查阅相关资料，或者在评论区/后台留言我，如果想了解的人数多的话我就找个机会挑出来单独写下。

### 日志！日志！日志！

除了上述 HTTPS 的问题，不知道你们有没有发觉还少了点东西。没错，就是日志！重要的事情说三遍！作为一个后台程序，怎么可以少了日志呢？既然我们已经使用 Gunicorn 接管了 Flask 项目，那怎么才能获得 Flask 的日志信息呢？别急，Gunicorn 早就帮你想好了，很简单，只需要在启动文件也就是`main.py`中添加一段代码即可。

```python
import logging
if __name__ != '__main__':
    gunicorn_logger = logging.getLogger('gunicorn.error')
    app.logger.handlers = gunicorn_logger.handlers
    app.logger.setLevel(gunicorn_logger.level)
```

这样的写法主要做了 4 件事：

1. `if __name__ != '__main__' `这种写法很少见，意思是仅当该文件作为模块被导入时内部的代码才会被执行，有个好处，避免了硬编码，当执行`gunicorn main:app`跑的时候`main.py`会被当成模块导入到 gunicorn 内部, 因为 gunicorn 实际上就是纯 pyhton 写的
2. 获取 gunicorn 的 logger
3. 将 gunicorn 的 logger 和 flask app 的 logger 绑定在一起
4. 将绑定的 logger 的 level 设置成 gunicorn logger 的 level, 因为最终输出的 log level 是在 gunicorn 中配置的

然后重新启动 gunicorn 就可以了，日志存放的路径我们之前也已经在 gunicorn.conf 中定义过了，用 cat 命令查看就 OK 了。

## 最后

不知不觉已经码了将近 4000 字，这次写这么多一方面是因为最近用这东西的时候发现可以扩展的东西太多了，另一方面也是希望可以和其他博客的内容有所区别，不仅仅把实现过程表现出来，还可以把自己使用过程中的一些习惯、心得体会分享出来和大家交流。当然部署这里能挖的还有很多，比如如何获取来访者的 IP 地址并加以分析，如何用 Nginx 配置 HTTPS 等等等等，有兴趣的可以相关资料，或者在留言区/后台和我讨论，你们的支持才是我前进的最大动力！
