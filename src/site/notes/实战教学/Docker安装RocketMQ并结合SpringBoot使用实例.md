---
{"title":"Docker 安装 RocketMQ 并结合 SpringBoot 使用实例","tags":["docker,rocketmq,springboot"],"dg-publish":true,"permalink":"/实战教学/Docker安装RocketMQ并结合SpringBoot使用实例/","dgPassFrontmatter":true}
---


在[[技术科普/浅入浅出消息队列\|《浅入浅出消息队列》]]一文中，我们了解了消息队列的作用、优缺点和使用场景，相信你对消息队列已经有了一个大致的概念，文末给自己埋的坑说日后会写一篇实战教程，正好现在实习结束了，也许久没有写实战教程了，于是这就来填坑了。

## 前置知识

阅读本文前，建议有一些前置知识，包括且不限于：

- 常见的 Linux 命令
- 消息队列的相关知识
- Docker 的基本使用
- docker-compose 的基础知识
- SpringBoot 的基本使用

那废话不多说，我们就开始吧。

> 本文的所涉及到的代码可在微信公众号「01 二进制」后台回复「rocketmq」获得。

## 为什么要以 RocketMQ 为例？

本文主要是为了通过实例的方式直观的了解消息队列。那么问题来了，消息队列那么多（ActiveMQ、RabbitMQ、Kafka），**我**为什么要选择 RocketMQ 呢？这里我们不谈原理，只说说体验，**仅是个人选择，不喜勿喷。**

1. 背靠阿里，不看测评，纯粹看他经历过多次双十一的检验就已经知道其性能是处于第一批次的。
1. 作为一个 Java 程序员，如果选择一个纯 Java 编写的软件，后期阅读其源码难度也会小很多。（RabbitMQ 底层是 Erlang，kafka 底层是 Scala）
1. 在阿里实习的时候一直都是使用 RocketMQ 的内部版本，于我而言，RocketMQ 更熟悉。

## 初识 RocketMQ

在使用消息队列前，我们要知道消息队列是什么，这一块内容参考之前的文章[[技术科普/浅入浅出消息队列\|《浅入浅出消息队列》]]，这里不再赘述。

本段节来讲解 RocketMQ 所涉及到的相关概念，我们先来简单看下官方给出的 RocketMQ 架构图

![](https://cdn.ytools.xyz/uPic/007S8ZIlgy1gjtpeiatlrj313y0msgoe.jpg)

从上图我们可以很直观的看出，一个完整的 RocketMQ 架构包含四个部分：**NameServer、Broker、Producer 和 Consumer**。

- NameServer：主要用作注册中心，用于管理 Topic 信息和路由信息的管理
- Broker：负责存储、消息 tag 过滤和转发。需将自身信息上报给注册中心 NameServer
- Producer：生产者
- Consumer：消费者

### 从寄信的角度理解

上面的解释可能难以理解，我们从寄信这一实例来看以下四个部分所承担的责任。

- Producer 和 Consumer 不必多说，消息的生产者和消费者，生产者负责投递消息，消费者负责接收消息，**是我们要编写的应用程序**。可以理解为寄信人和收信人。
- Broker 负责消息存储，以 Topic（主题）为维度，以队列的形式存储消息。可以理解为信箱，专门存储信件，收信人（Consumer）可以从这里获取信件。
- NameServer 负责对源数据进行管理，包括了对 Topic 和 Broker 的管理。可以理解为邮局，负责管理邮件的分发，维护信箱（Broker）的状态。

由上各部分角色的功能可知，我们需要先安装启动 NameServer，再启动 Broker 即可搭建完 RocketMQ

## 安装 RocketMQ

> 如果你的电脑上已经配置好了 rocketmq 的相关环境，可以跳过本章节。

从上面的介绍我们可以得知，在生产和消费消息之前，我们需要安装好**Broker 和 NameServer。**

### 准备工作

为了部署方便，我推荐使用 docker 搭建服务。此外，由于 rocketmq 需要分别部署 broker 与 nameserver ，考虑到分开部署比较麻烦，这里我将会使用 docker-compose。因此，你需要在你的宿主机中安装好 docker 和 docker-compose。

此外，我们还需要搭建一个 web 可视化控制台，用于监控 mq 服务状态，以及消息消费情况，这里使用 rocketmq-console，同样该程序也将使用 docker 安装。

> 如果对 docker 不熟悉的话，可以先阅读菜鸟教程的 docker 教程学习 👉[Docker 教程](https://www.runoob.com/docker/docker-tutorial.html)

### 安装

#### 安装 Docker

Linux：

执行以下命令

```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

Mac：

执行以下命令

```bash
brew cask install docker
```

Win:

下载对应的安装文件，然后双击运行安装。下载地址在：[https://hub.docker.com/editions/community/docker-ce-desktop-windows](https://hub.docker.com/editions/community/docker-ce-desktop-windows)

> 考虑到下载该文件需要科学上网，你可以在微信公众号「01 二进制」后台回复「docker」获取 docker 安装包的下载链接。

如果你的 win10 系统可以使用 winget，那就执行以下命令。(win 终于也有自己的包管理工具了 🙏)

```powershell
winget install Docker.DockerDesktop
```

国内从 DockerHub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。配置教程可参考 👉[Docker 镜像加速](https://www.runoob.com/docker/docker-mirror-acceleration.html)

#### 安装 RocketMQ 镜像

rocketmq 的 docker 镜像我们可以自己制作，官方文档中有详细介绍 👉[apache](https://github.com/apache)/**[rocketmq-docker](https://github.com/apache/rocketmq-docker)**

为了方便起见，这里我们直接使用别人已经制作好的镜像，镜像地址 👉 [foxiswho/rocketmq](https://hub.docker.com/r/foxiswho/rocketmq)

新建一个目录用于存放相关脚本，然后在终端执行下面的命令 👇

```shell
git clone https://github.com/foxiswho/docker-rocketmq.git
cd docker-rocketmq
cd rmq
chmod +x  start.sh
./start.sh
```

在经过一段时间的等待后，我们通过浏览器访问`localhost:8180`查看到以下页面则说明安装成功。

![](https://cdn.ytools.xyz/uPic/007S8ZIlgy1gjtwv48yc3j315g0np415.jpg)



### 安装脚本解析

通过脚本的方式一键安装确实很方便，但如果只是安装完成就万事大吉了自然是不行的，本着授人以渔的态度，我们来看看安装脚本里都有些啥：

#### start.sh

![](https://cdn.ytools.xyz/uPic/007S8ZIlgy1gjtwval912j30za0sun1s.jpg)

4-7 行在创建目录，10-13 行在给刚才创建的目录设置权限，至于原因我们之后再说。

我们看到 16 行使用 docker-compose 命令启动了容器，并设置为了后台自动启动，因此我们来看一下这个 docker-compose.yml 文件。

#### docker-compose.yml

```yaml
version: "3.5"

services:
  rmqnamesrv:
    image: foxiswho/rocketmq:4.7.0
    container_name: rmqnamesrv
    ports:
      - 9876:9876
    volumes:
      - ./rmqs/logs:/opt/logs
      - ./rmqs/store:/opt/store
    environment:
      JAVA_OPT_EXT: "-Duser.home=/opt -Xms512M -Xmx512M -Xmn128m"
    command: ["sh", "mqnamesrv"]
    networks:
      rmq:
        aliases:
          - rmqnamesrv
  rmqbroker:
    image: foxiswho/rocketmq:4.7.0
    container_name: rmqbroker
    ports:
      - 10909:10909
      - 10911:10911
    volumes:
      - ./rmq/logs:/opt/logs
      - ./rmq/store:/opt/store
      - ./rmq/brokerconf/broker.conf:/etc/rocketmq/broker.conf
    environment:
      JAVA_OPT_EXT: "-Duser.home=/opt -Xms512M -Xmx512M -Xmn128m"
    command:
      [
        "sh",
        "mqbroker",
        "-c",
        "/etc/rocketmq/broker.conf",
        "-n",
        "rmqnamesrv:9876",
        "autoCreateTopicEnable=true",
      ]
    depends_on:
      - rmqnamesrv
    networks:
      rmq:
        aliases:
          - rmqbroker

  rmqconsole:
    image: styletang/rocketmq-console-ng
    container_name: rmqconsole
    ports:
      - 8180:8080
    environment:
      JAVA_OPTS: "-Drocketmq.namesrv.addr=rmqnamesrv:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false"
    depends_on:
      - rmqnamesrv
    networks:
      rmq:
        aliases:
          - rmqconsole

networks:
  rmq:
    name: rmq
    driver: bridge
```

我们创建了三个服务，这三个服务的名字分别是 rmqnamesrv、rmqbroker 和 rmqconsole，分别对应我们之前所说的 nameserver、broker 和可视化控制台。并且对不同的服务做了不同的端口映射，同时将本地指定的文件目录挂载到 docker 容器中，并以网桥（bridge）的形式进行网络连接。

以`rmqnamesrv`为例，其基础镜像为`foxiswho/rocketmq:4.7.0`，创建的容器名为`rmqnamesrv`，并将其内部的 9876 端口映射到宿主机的 9876 端口，并将本地的`./rmqs/logs`文件挂载到 docker 容器的`/opt/logs`目录中。

```yaml
rmqnamesrv:
  image: foxiswho/rocketmq:4.7.0
  container_name: rmqnamesrv
  ports:
    - 9876:9876
  volumes:
    - ./rmqs/logs:/opt/logs
    - ./rmqs/store:/opt/store
```

> 如果对于 docker-compose 不熟悉的读者，可以先参考相关的教程学习一下 👉[Docker Compose](https://www.runoob.com/docker/docker-compose.html)

## SpringBoot 整合 RocketMQ 小实例

在完成了相对复杂的安装、配置后，我们终于可以实现一个小的 demo 来打通整个流程了。

### 创建消息主题和订阅组

使用 RocketMQ 进行发消息时，**必须要指定 topic**，对于 topic 的设置有一个开关`autoCreateTopicEnable`，一般在**开发测试**环境中会使用默认设置`autoCreateTopicEnable = true`，但是这样就会导致 topic 的设置不容易规范管理，没有统一的审核等等，所以在正式环境中会在 Broker 启动时设置参数`autoCreateTopicEnable = false`。这样当需要增加 topic 时就需要在 web 管理界面上添加即可。

在 web 界面添加 topic 的方式如下：

![](https://cdn.ytools.xyz/uPic/007S8ZIlgy1gjtwvp4it8j315g0l076f.jpg)

同理，在接受消息时，我们同样需要对消息订阅组进行配置，对于消息的订阅设置有一个开关`autoCreateSubscriptionGroup`，通常情况下，在生产环境下，我们需要设置为`autoCreateSubscriptionGroup=false`，这就要求了管理者必须去 web 管理界面上创建订阅组才可以收到消息。

在 web 界面添加订阅组的方式类似，如下图所示：

![](https://cdn.ytools.xyz/uPic/007S8ZIlgy1gjtwvxy8rfj315g0g876e.jpg)

> 如果只是测试环境，我们可以在配置文件中将这两个开关打开，配置文件在 `rmq/rmq/brokerconf` 目录下

### 编写代码

apache 官方已经提供了 rocketmq 对应的 springboot starter，这极大的简化了我们所需要做的配置工作，因此我们要做的就是先新建一个 springboot 项目，然后按照下面的方式着手实现。

### 导入依赖

首先先在 pom.xml 中导入 apache 官方提供的 starter

```
<!-- https://mvnrepository.com/artifact/org.apache.rocketmq/rocketmq-spring-boot-starter -->
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>2.1.0</version>
</dependency>
```

### 配置 application.yml

依赖导入后，我们需要在 application.yml 配置一个 name-server 地址，具体值看你的机器。

```yaml
rocketmq:
  name-server: localhost:9876
  producer:
    group: myGroup
```

### 创建一个生产者类

生产者发送消息：

```java
@RestController
public class RocketController {
    @Autowired
    private RocketMQTemplate rocketMQTemplate;
    // 发送给Broker，默认会自动创建topic，topic和tag用冒号分隔
    @GetMapping("/rocket/send")
    public String rocketSend() {
        LocalDateTime currentTime = LocalDateTime.now();
        rocketMQTemplate.convertAndSend("rocket-topic-2", currentTime.toString());
        return currentTime.toString();
    }
    // 延时消息，RocketMQ支持这几个级别的延时消息，不能自定义时长
    // 1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
    @GetMapping("/rocket/delayMsg/send")
    public String rocketDelayMsgSend() {
        LocalDateTime currentTime = LocalDateTime.now();
        rocketMQTemplate.syncSend("rocket-topic-2:tag-2", MessageBuilder.withPayload(currentTime.toString()).build(), 2000, 3);
        return currentTime.toString();
    }
}
```

### 创建一个消费者

消费者监听消息：

```java
@Component
@Slf4j
public class RokcetServiceListener {
    @Service
    @RocketMQMessageListener(consumerGroup = "consumer-group-1", topic = "rocket-topic-2")
    public class Consumer1 implements RocketMQListener<String> {
        @Override
        public void onMessage(String s) {
            log.info("consumer1 rocket收到消息：{}", s);
        }
    }
    // RocketMQ支持两种消费方式，集器消费和广播消费
    @Service
    @RocketMQMessageListener(consumerGroup = "consumer-group-2", topic = "rocket-topic-2",
            selectorExpression = "tag2", messageModel = MessageModel.BROADCASTING)
    public class Consumer2 implements RocketMQListener<String> {
        @Override
        public void onMessage(String s) {
            log.info("consumer2 rocket收到消息：{}", s);
        }
    }
}
```

### 测试

我们在浏览器中访问`localhost:8080/rocket/send`，即可看到返回的时间戳。

![](https://cdn.ytools.xyz/uPic/007S8ZIlgy1gjtww7r7bsj315g08bq46.jpg)

同时在控制台也可以看到消费者已经获取到这条信息了

![](https://cdn.ytools.xyz/uPic/007S8ZIlgy1gjtwwczwpjj315g0ac483.jpg)

同样的，我们也可以在可视化控制台查看到相应的消息

![](https://cdn.ytools.xyz/uPic/007S8ZIlgy1gjtwwim4qqj31230u0aem.jpg)

我们同样可以在可视化控制台查看消费者和生产者对于消息的生产与消费的情况，这些就留给读者自己探索了。至此，一个完整的利用 Docker 安装 RocketMQ 并结合 SpringBoot 使用的实例就结束了。

## 问题

### 问题 1：No route info of this topic: xxxxxx

通过翻译我们可以知道，这个错误产生的原因是因为消息队列中并未产生相对应的**topic**，所以我们要做的应该是去控制台新建一个 topic

![](https://cdn.ytools.xyz/uPic/007S8ZIlgy1gjtwwom5h8j315g0l076f.jpg)

### 问题 2：连接异常

如果出现类似下述这种连接异常的错误

```
com.alibaba.rocketmq.remoting.exception.RemotingConnectException: connect to <172.0.0.120:10909> failed
```

可能的原因是你并没有将项目放至 docker 容器中，因此你的项目代码不能直接与 rocketmq 容器访问，因此我们需要将`broker.conf`中的 `#brokerIP1=xxxxx` 前面`#`号去掉，并且把后面的`IP地址`改成你的`rocketmq`容器宿主机`IP地址`，配置文件在 `rmq/rmq/brokerconf` 目录下。

## 最后

为了填坑，我选择了 rocketmq 作为实例讲解的对象，并在第一节阐述了我为什么要使用 RocketMQ 的原因，之后解释了 RocketMQ 中几个重要的概念，然后利用 docker 快速的部署安装了一个 rocketmq 的单机实例，并分析了安装脚本。最后我们通过 springboot 这一目前主流的 web 框架实现了一个生产者与消费者的实例，并说明了可能会遇到的问题及解决方案。

以上就是本文的全部内容了，如果你觉得对你有所帮助，不放关注点赞支持一波，你们的支持是我更新的最大动力。
