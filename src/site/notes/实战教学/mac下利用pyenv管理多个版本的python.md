---
{"title":"mac下利用pyenv管理多个版本的python","categories":"实战教学","tags":["其他,pyenv"],"dg-publish":true,"permalink":"/实战教学/mac下利用pyenv管理多个版本的python/","dgPassFrontmatter":true}
---


![](https://ws4.sinaimg.cn/large/006tNc79ly1fzpydn9g1ij313s0u0acm.jpg)

## 前言
经常遇到这样的情况：
* 系统自带的Python是2.x，自己需要Python 3.x；
* 某些机器学习的框架（如PaddlePaddle/Tensorflow）需要的版本是python3.5，但是你的系统支持的python版本较高，且无法删除（因为某些软件会和python产生依赖）

此时需要在系统中安装多个Python，但又不能影响系统自带的Python，即需要实现Python的多版本共存。[pyenv](https://github.com/yyuu/pyenv)就是这样一个Python版本管理器。

**pyenv**可以进行全局的 Python 版本切换，也可以给单个项目提供对应的 Python 版本。用了 「pyenv」以后，就可以很容易的安装不同的 Python 版本，不同版本之间的切换也变得 so easy。

## 注意
> Pyenv只会管理通过Pyenv安装的Python版本，你自己在Python官网上下载的直接安装的Pyenv`并不能被管理！！！`同样除了系统自带的python包外,`其他直接安装`的python包是`识别不出来`的,即使使用的brew安装的也识别不出来.

## pyenv的安装
1. 安装工具：[brew](https://brew.sh/index_zh-cn)
2. 系统环境：![](https://upload-images.jianshu.io/upload_images/5666077-b573ed4d6756ec14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 通过homebrew安装

```
brew install pyenv
```

但是github提示了一句话
> After installation, you'll need to `add eval "$(pyenv init -)" to your profile` (as stated in the caveats displayed by Homebrew — to display them again, usebrew info pyenv). You only need to add that to your profile once.

意思就是说我们需要在profile文件里面添加一句

```
eval "$(pyenv init -)" 
```

博主亲测，如果没有这一步，后面执行`pyenv global [version]`是不会成功的。

#### 编辑.bash_profile文件
----
在终端中输入如下命令，进入当前用户的Home目录

```
cd ~ 
```

输入如下命令，打开.bash_profile文件

```
open .bash_profile
```

如不存在，则输入如下命令，创建文件

```
touch .bash_profile
```

编辑文件

```
open -e .bash_profile
```

在弹出的.bash_profile文件中新增

```
eval "$(pyenv init -)"
```

command + s 保存文件,然后在终端中输入如下命令，刷新之前配置的.bash_profile文件.

```
source .bash_profile
```

### 验证pyenv是否安装成功
执行如下命令：

```
pyenv --help
```

上面命令行的意思是获取 prenv 的帮助信息。

![](https://upload-images.jianshu.io/upload_images/5666077-e0358401b5843a4b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## pyenv的常用命令
![](https://upload-images.jianshu.io/upload_images/5666077-1dc672875082b73b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图是官方文档中的例子，以下是整理的一些pyenv的常用命令，如果想要查看完整命令列表，可以点击查看[pyenv命令列表](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md#command-reference)

* 查看pyenv支持哪些Python版本

```
pyenv install --list
```

![查看可以安装的版本](https://upload-images.jianshu.io/upload_images/5666077-34180201b67c6a5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/540)

* 查看已经安装的python版本

```
pyenv versions
```

![查看已经安装的版本](https://upload-images.jianshu.io/upload_images/5666077-b3cd0f47a0508a22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/540)

* 查看当前使用的python版本

```
pyenv version
```

![查看当前使用的Python版本](https://upload-images.jianshu.io/upload_images/5666077-5a4b5af4f711a865.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/540)

* 安装一个python版本如3.5.6

```
pyenv install 3.5.6
```

* 安装完成之后需要对数据库进行更新：

```
pyenv rehash
```

* 卸载一个python版本如3.5.6

```
pyenv uninstall  3.5.6
```

* 设置全局python版本如3.5.6

```
pyenv global 3.5.6
//很多人不推荐这么做,说是mac操作系统的文件也会调用原生的2.7的python版本
//这种说法感觉有点:恐惧来自未知的感觉.持保留意见
```

* 这个时候确认一下当前python的版本

![确认当前python的版本](https://upload-images.jianshu.io/upload_images/5666077-4b911293e5787e60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/540)
发现已经更改为3.5.6了

* 设置目录级python版本如3.5.6

```
pyenv local 3.5.6
```

* 为当前shell会话设置python版本如3.5.6

```
pyenv shell 3.5.6
```

## 常见问题解决

1. `pyenv install [version]`下载太慢
只需要在python的官网下载你需要的python版本的`tar.xz`文件然后放到 `/User/.pyenv/cache`中然后再执行`pyenv install [version]`就可以了

![下载第二个文件](https://upload-images.jianshu.io/upload_images/5666077-4c9dbf9675a8f1c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/540)

> 在这里提一下：`/.pyenv` 在mac中是隐藏文件夹，mac显示隐藏文件夹的快捷键是：`shift+command+.`

2. 出现 `zipimport.ZipImportError: can't decompress data; zlib not available` 的问题

```
~ pyenv install 3.5-dev
Cloning https://hg.python.org/cpython...
Installing Python-3.5-dev...
 
BUILD FAILED (OS X 10.11.6 using python-build 20150818)
 
Inspect or clean up the working tree at /var/folders/23/4kbs9t712jv1mvmw6cpjwr2m0000gn/T/python-build.20160815000934.22185
Results logged to /var/folders/23/4kbs9t712jv1mvmw6cpjwr2m0000gn/T/python-build.20160815000934.22185.log
 
Last 10 log lines:
  File "/private/var/folders/23/4kbs9t712jv1mvmw6cpjwr2m0000gn/T/python-build.20160815000934.22185/Python-3.5-dev/Lib/ensurepip/__main__.py", line 4, in <module>
    ensurepip._main()
  File "/private/var/folders/23/4kbs9t712jv1mvmw6cpjwr2m0000gn/T/python-build.20160815000934.22185/Python-3.5-dev/Lib/ensurepip/__init__.py", line 209, in _main
    default_pip=args.default_pip,
  File "/private/var/folders/23/4kbs9t712jv1mvmw6cpjwr2m0000gn/T/python-build.20160815000934.22185/Python-3.5-dev/Lib/ensurepip/__init__.py", line 116, in bootstrap
    _run_pip(args + [p[0] for p in _PROJECTS], additional_paths)
  File "/private/var/folders/23/4kbs9t712jv1mvmw6cpjwr2m0000gn/T/python-build.20160815000934.22185/Python-3.5-dev/Lib/ensurepip/__init__.py", line 40, in _run_pip
    import pip
zipimport.ZipImportError: can't decompress data; zlib not available
make: *** [install] Error 1
```

解决方案参考[#451](https://github.com/pyenv/pyenv/issues/451)
用如下命令就可以解决了：

```
~ CFLAGS="-I$(brew --prefix openssl)/include -I$(xcrun --show-sdk-path)/usr/include" \
  LDFLAGS="-L$(brew --prefix openssl)/lib" \
  pyenv install -v 3.5-dev
```

3. `pyenv global [verion]`命令失效
这个问题我已经在上面说过了，需要在`bash_profile`文件里面添加一句

```
eval "$(pyenv init -)" 
```

添加方式上面已经详细介绍过了。

## 最后

本文参考：
1. [Simple Python Version Management: pyenv](https://github.com/pyenv/pyenv)
2. [Mac下 Pyenv 的安装使用](https://www.jianshu.com/p/cea9259d87df)
