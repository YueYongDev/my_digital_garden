---
{"title":"Python 管理哪家强？","categories":"实战教学","tags":["python,环境管理"],"dg-publish":true,"permalink":"/实战教学/Python 管理哪家强/","dgPassFrontmatter":true}
---


![Pipenv - The Officially Recommended Python Packaging Tool](https://cdn.ytools.xyz/uPic/006tKfTcgy1g13dkeueaoj30k009g75s.jpg)

之前的文章[[实战教学/mac下利用pyenv管理多个版本的python\|《利用pyenv管理多个版本的python》]]提过pyenv 是一个非常好用的 Python 版本管理工具，利用它我们可以在同一台电脑上安装多个版本的 Python ，这个过程非常简单。Mac系统的电脑一行命令就可以安装了：

```bash
brew install pyenv
```

pyenv 的安装和使用详见开篇提到的文章，这里不再赘述。

今天主要是想介绍另一个非常好用的 Python 工具——**"Pipenv"**

## Pipenv是什么？

> **Pipenv** is a tool that aims to bring the best of all packaging worlds (bundler, composer, npm, cargo, yarn, etc.) to the Python world. *Windows is a first-class citizen, in our world.*
>
> It automatically creates and manages a virtualenv for your projects, as well as adds/removes packages from your `Pipfile` as you install/uninstall packages. It also generates the ever-important `Pipfile.lock`, which is used to produce deterministic builds.

上段引用自官方文档，简单来说就是 Pipenv 是希望成为 Python 界最好的包管理工具，成为 Python 界的 *npm*。你可以使用 Pipenv 这一个工具来安装、卸载、跟踪和记录依赖性，并创建、使用和组织你的虚拟环境。
当你使用它启动一个项目时，如果你还没有使用虚拟环境的话，Pipenv 将自动为该项目创建一个虚拟环境。
Pipenv 使用 `Pipfile` 和 `Pipfile.lock` 来管理依赖包，并且在使用 pipenv 添加或删除包时，自动维护 Pipfile 文件，同时生成 Pipfile.lock 来锁定安装包的版本和依赖信息，避免构建错误。相比pip需要手动维护requirements.txt 中的安装包和版本，具有很大的进步。

## 为什么要使用 Pipenv

设想一个场景：你手头有多个开发项目，其中项目 A 和项目 B 要求用 Python3，项目C需要用 Python2，而项目 A 和项目 B 又要求第三方依赖包相互独立，互不干扰。

这个时候，为了满足版本的不同我们需要使用 Pyenv 安装多个版本的 Python ，同时为了使不同项目之间隔离开来，我们可以使用 Pipenv 创建虚拟的开发环境。

那有的人就要问了，`virtualenv`不就可以创建虚拟环境了吗，为什么还要使用 Pipenv ，这里先卖个关子，文末我再来比较这些常见工具之间的差异性。

## 使用 Pipenv

### 安装 Pipenv

如果你是Mac电脑，那么推荐使用Homebrew来安装。

```bash
brew install pipenv
```

如果不是Mac电脑，建议 使用Python3的pip3 安装：

```bash
pip3 install pipenv
```

执行pipenv，可以查看pipenv的帮助信息：

```bash
pipenv
```

如果出现如下信息，就说明安装成功了：

![](https://cdn.ytools.xyz/uPic/006tKfTcgy1g13bfa496rj30vo0kadqb.jpg)

### 为项目创建虚拟环境

默认地，虚拟环境会创建在`~/.local/share/virtualenvs`目录里面。
如果我们希望在每个项目的根目录下保存虚拟环境目录（.venv），需要在 `.bashrc` 或 `.bash_profile` 中配置如下：

```
export PIPENV_VENV_IN_PROJECT=1
```

要想使配置生效，执行下`source ~/.bashrc`或者`source ~/.bash_profile`即可使环境变量生效。
接下来我们为项目创建虚拟环境。

```
mkdir pipenv_demo		
cd pipenv_demo
pipenv --python 3.6.7 # 为当前初始化一个版本为 3.6.7 的python环境
```

### 首次运行

如果是第一次在项目中运行 `pipenv` 命令的话，会在项目中创建一个名为 `Pipfile` 的文件，文件内容类似下面这样。

```
[[source\|source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]
requests-html = "*"

[dev-packages]

[requires]
python_version = "3.7"
```

这个文件就类似于 maven 中的 `pom.xml`、npm 中的 `package.json` 文件。如果运行过install、update等命令的话，还会创建一个`Pipfile.lock`文件，类似npm中的lock文件。这两个文件就是pipenv用于管理第三方库的配置文件，如果同时使用版本控制软件的话，需要将它们也加入进去。

> Tips：对于这个文件我们需要先做一件事，就是更改 **[[source]]** 中的 url 为国内镜像。
>
> ```
> [[source]]
>  url = "https://pypi.tuna.tsinghua.edu.cn/simple"
> ```

### 常用命令

#### 安装项目依赖

如果我想在项目中安装requests这个包，运行：

```
pipenv install requests
```

如果需要指定具体版本号，可以这样：

```
pipenv install requests==2.13.0
```


如果是第一次运行pipenv的话，会先创建Pipfile文件，否则会修改Pipfile文件。

该命令还有一个常用参数-d或--dev，用于安装仅供开发使用的包。

> Tips：用 git 管理项目时候，要把 `Pipfile` 和 `Pipfile.lock` 加入版本跟踪。这样 clone 了这个项目的同学，只需要执行
>
> ```
> pipenv install
> ```
>
> 就可以安装所有的Pipfile中 [packages]部分的包了，并且自动为项目在自己电脑上创建了虚拟环境。

#### 卸载某个依赖

相应的还有命令来卸载第三方包，该命令还有两个参数`--all`和`--all-dev`用于卸载所有包和所有开发包。

```
pipenv uninstall requests
```

#### 更新项目依赖

查看所有需要更新的包：

```
pipenv update --outdated
```

更新所有包：

```
pipenv update
```

更新指定的包：

```
pipenv update <包名>
```

#### 其他命令

```
pipenv -h  # 查看帮助
pipenv graph  # 查看已安装的包
pipenv shell  # 启动shell
pipenv run python hello.py  # 运行脚本
```

### Python 管理哪家强？

还记得在最开始提出的问题吗？明明`virtualenv`就可以创建虚拟环境了吗，为什么还要使用 Pipenv 呢？

接下来我们来横向对比下几个常见的容易弄混的 python 管理工具（我在初学的时候总是分不清😂）

| 工具              | 介绍                                                         | 原理         |
| ----------------- | ------------------------------------------------------------ | ------------ |
| pip               | 包管理工具                                                   |              |
| virtualenv        | 虚拟环境管理工具                                             | 切换目录     |
| virtualenvwrapper | 虚拟环境管理工具加强版                                       |              |
| pyenv             | python版本管理工具                                           | 修改环境变量 |
| pyenv-virtualenv  | 虚拟环境管理工具                                             |              |
| pipenv            | 项目环境管理工具                                             |              |
| anaconda          | 一个包含180+的科学包及其依赖项的发行版本（不只是 Python 可用） |              |

同时在推荐一个我在 stackoverflow 上看到的回答：[What is the difference between pyenv, virtualenv, anaconda?](https://stackoverflow.com/questions/38217545/what-is-the-difference-between-pyenv-virtualenv-anaconda/39928067#39928067)

### 实践建议：

建议团队内开发人员，在自己电脑上都安装 pyenv 和 pipenv 。Pipfile 和 Pipfile.lock 加入版本跟踪，.venv 不要加入版本管理。为自己的每一个项目建立独立的虚拟环境。

### 参考

1. [What is the difference between pyenv, virtualenv, anaconda?](https://stackoverflow.com/questions/38217545/what-is-the-difference-between-pyenv-virtualenv-anaconda/39928067#39928067)
2. [利用pipenv和pyenv管理多个相互独立的Python虚拟开发环境](https://blog.csdn.net/liuchunming033/article/details/79582617)
3. [pipenv快速入门](https://blog.csdn.net/u011054333/article/details/82891847)

