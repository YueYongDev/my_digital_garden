---
{"title":"科普系列——如何解释什么是 AJAX？","categories":["技术科普"],"tags":["计算机网络"],"cover":"https://upload-images.jianshu.io/upload_images/5666077-9f45d4321e209d94.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240","dg-publish":true,"permalink":"//ajax/","dgPassFrontmatter":true}
---


## 前言

学妹这学期新开了一门课《Script 及 AJAX 开发技术》，然而临近学期末，她突然跑来问我：到底什么是 AJAX ？相信很多人（尤其是前端）在写代码的时候经常会用到 AJAX 技术，但是如果真要说出个所以然，可能还会有些困难。其实简单概括下，AJAX 就是一种利用 JavaScript 向服务端发起请求，并获得服务端响应的**技术**。它的特点是**异步请求，局部刷新。**

> **Tips：**这里我将**技术**二字加粗了，是因为很多初学者会以为 AJAX 是一个库/框架，类似于*JQuery/Vue*之类的，因而有很多初学者会提出该怎么安装 AJAX 的问题。事实上 AJAX 是一种技术。

虽然概括起来很简单，但是 AJAX 技术的一些细节仍然值得我们思考，接下来我会详细的介绍。

## AJAX 解决的问题

我们刚才说过了，AJAX 是一种发送**请求**的技术，那在 AJAX 被发明前，浏览器是如何请求的呢？

1. 地址栏。用户在地址栏输入 [http://baidu.com](https://link.zhihu.com/?target=http%3A//baidu.com) ，按回车，就向 [http://baidu.com](https://link.zhihu.com/?target=http%3A//baidu.com) 发起了一个请求。（同时页面刷新）
2. a 标签。用户点击页面中的 a 链接，也会发起一个请求。（同时页面刷新）
3. img 标签。页面中如果有 img 标签，那么就会发起一个对此图片的请求（页面没有刷新，但是只能请求图片）类似的还有 link 标签、script 标签，都可以对一类文件的请求。

在这三种方式中，除了第三种，其他两种方式想要发送一个请求，就必须要刷新页面，如果页面只有展示内容的话刷新一下自然无所谓，但倘若一个页面有很多的表单内容需要填写，而你在最后填写完成提交的时候才告诉你，其中某一个地方不符合要求，要你回去重填，然后刷新一下页面，内容都消失了，怕是当时就可能会气的暴走了吧。

也正是这种极端的用户体验让微软创新地开发了一个接口 ActiveXObject("Microsoft.XMLHTTP")，并在 IE 5.0 中开放给开发者用。通过该接口，浏览器可以向服务器发送请求并取回所需的数据，并在客户端采用 JavaScript 处理来自服务器的回应。这就是 AJAX 的前身。随后这种技术被谷歌的开发人员发现并运用在 Gmail 中，再然后就是 W3C 制定了一个标准用来规范 AJAX，至此 AJAX 算是正式成为每一个前端开发者的必备技能了。

通过 AJAX 技术，服务器和浏览器之间交换的数据大量减少，服务器回应更快了。同时，很多的处理工作可以在发出请求的客户端机器上完成，因此服务端的负荷也减少了许多。

## AJAX 的原理

那 AJAX 的实现原理又是什么呢？我们先来看一下 AJAX 的定义，以下内容摘自维基百科：

> **AJAX**即“**Asynchronous JavaScript and XML**”（异步的[JavaScript](https://zh.wikipedia.org/wiki/JavaScript)与[XML](https://zh.wikipedia.org/wiki/XML)技术），指的是一套综合了多项技术的[浏览器](https://zh.wikipedia.org/wiki/瀏覽器)端[网页](https://zh.wikipedia.org/wiki/網頁)开发技术。

这里又出现了一个新的名词：**异步**。这个名词在计算机领域可以说是一个很重要的名词了，很多技术都离不开异步二字，比如 Nodejs 的**异步**非阻塞 I/O 模型，当然这就是题外话了。我们应该怎么理解这里的异步呢？

不急不急，我们先来看一个生活中非常常见的例子 🌰：

![](https://upload-images.jianshu.io/upload_images/5666077-abeef41042891d22.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这种场景在上学的时候很常见，其实 AJAX 的原理和上述流程很相似，不信你看下面：

![](https://upload-images.jianshu.io/upload_images/5666077-269d112e62ac5a37.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在上述例子中，核心是班长（也就是 HXR 对象），班主任可以通过他传递消息（客户端构建 XHR 对象发送请求）然后收到响应。在班长去通知小明的过程中，班主任仍然可以继续手头的工作，这就是一个异步的过程。（果然生活处处皆学问）

那么我们又该如何在代码中使用这个 XHR 对象呢？

## AJAX 的使用

XHR 的全称是 XMLHttpRequest，这是由微软首先引入的一个特性，其他浏览器提供商后来都提供了相同的实现。这跟以前的技术最大的不同点在于**「页面无需刷新」**，仅此而已。

想要使用 AJAX 发起一个请求很简单，一共 4 步。

1. 创建一个 XHR 对象（需要考虑浏览器差异）

```javascript
var request = null;
if (window.XMLHttpRequest) {
  // 兼容 IE7+, Firefox, Chrome, Opera, Safari
  xhr = new XMLHttpRequest();
} else {
  // 兼容 IE6, IE5
  xhr = new ActiveXObject("Microsoft.XMLHTTP");
}
```

2. 监听请求成功后的状态变化

```javascript
request.onreadystatechange = function () {
  if (this.readyState == 4 && this.status == 200) {
    console.log(request.responseText);
  }
};
```

第三行的 request.responseText 就是服务器返回的内容了（默认是字符串）

3. 设置请求参数

```javascript
request.open(method, url, async);
```

请求的三个参数分别是：请求的方法、请求的地址、和是否采用异步请求。

4. 发送请求

```javascript
request.send();
```

说实话，虽然只有 4 步，但是通过这种原生的方法发送请求还是觉得有些复杂，那有没有什么简单的方法呢？

## AJAX 的其他使用方式

### JQuery 使用 AJAX

JQuery 将上述过程封装的很好，使用起来也非常简单（只举出最简单的例子，详细还请移步官方文档）：

```javascript
$.get("url").then(function (response) {
  // 这里的 response 就是返回的内容
});
```

### Vue 使用 AJAX

vue 官方推荐使用 axios 来进行请求，这里同样举出一个最简单的例子

```javascript
getApiData:function() {
	//设置请求路径
	var url = "XXXXXX";
	// 发送请求:将数据返回到一个回到函数中
	// 并且响应成功以后会执行then方法中的回调函数
	axios.get(url).then(function(result) {
		// result是所有的返回回来的数据
		// 包括了响应报文行
		// 响应报文头
		// 响应报文体
		console.log(result.data);
	});
}
```

### 微信小程序使用 AJAX

微信小程序的请求就是 wx.request 这个 api，wx.request(一些对象参数)，微信小程序不同于浏览器的 ajax 请求，可以直接跨域请求不用考虑跨域问题。

```javascript
wx.request({
  url: "test.php",
  data: {
    x: "",
    y: "",
  },
  header: {
    "Content-Type": "application/json",
  },
  success: function (res) {
    console.log(res.data);
  },
});
```

## 拓展

### HTTP 状态码

看到这其实我们就可以发送数据了，那怎么接收返回的数据呢？事实上，这已经不是在 AJAX 的讨论范围了，但是作为一个拓展知识点，我还是想介绍下**状态码**这个东西。状态码的作用是服务器返回给客户端的用来描述 HTTP 请求的状态的。用来描述 HTTP 请求的状态码太多了，这里介绍一些常见的状态码。

- 200 表示从请求成功
- 301 表示永久性重定向。该状态码表示请求的资源已被分配了新的 URI，以后应使用资源现在所指的 URI。
- 302 表示临时性重定向。
- 404 表示服务器上找不到请求的资源。
- 500 表示服务器端在执行请求时发生了错误。多半是因为 Web 应用存在的 bug 或某些临时的故障。
- 503 表示服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。

### 获取网页中的 XHR 请求

这时就有人可能会问了，有没有什么办法可以获取一个网页中的 XHR 请求呢？当然是有的，这一过程其实说的宽泛点其实就是抓包，这里我以掘金为例，介绍下获取网页中的 XHR 请求。

首先我们打开 Chrome 浏览器，然后进入开发者工具（按 F12 或者网页右击选择“检查”），选择**Network**选项卡，我们可以发现下面有很多东西，比如 Filter、All、HXR、JS 等等，通过这个工具这里我们可以看见一个网页渲染过程中的所有请求（不只是 XHR，还有 JS、CSS 等）。

![](https://upload-images.jianshu.io/upload_images/5666077-f24f921a3097d63b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

随后我们选择 XHR，就会出现请求这个网页过程中的所有的 XHR 请求了。包括 name、status、size 等信息。

![](https://upload-images.jianshu.io/upload_images/5666077-4e94aa9a30590069.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

之前提到过了，我们通过 XHR 携带的数据返回给浏览器渲染页面，到底是怎么实现的呢？不急，我们先来看一下现在的页面是什么样的：

![](https://upload-images.jianshu.io/upload_images/5666077-0897c3b047592149.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其实这些东西都在其中一个 XHR 中，于是我们随便点击一个名为*query*的 XHR 对象（其实并不是随便点击的 😜），然后移到**Response**选项卡：

![](https://upload-images.jianshu.io/upload_images/5666077-a23ae5f4a574780c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们看到了很长的一段 JSON 数据，格式化后（这里我们可以直接切换到**Preview**选项卡）筛选出一部分可以看到：

![](https://upload-images.jianshu.io/upload_images/5666077-20648c5ef9c37eb7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

是不是刚才那个页面里面的东西都在这里面呢？

### 简单分析下

既然都获取到请求数据了，再不分析下都感觉对不起这么多的数据了，让我们把选项卡从**Response**移到**Headers**上，我们惊讶的发现竟然出现了好多东西：

![](https://upload-images.jianshu.io/upload_images/5666077-f13ba2a0b85a0801.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里我们简单说明下各个参数：

#### General 部分

首先是 General 部分：

![](https://upload-images.jianshu.io/upload_images/5666077-959b767f97475b12.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以看出，请求地址是`https://web-api.juejin.im/query`，请求方法为 POST 方法，请求状态是 200，也就是请求成功了，同时还可以知道这次请求的 IP 地址是`119.254.97.159:443`。

#### Referrer Policy

这里说下**Referrer Policy**这个字段，这个字段解释起来有点小麻烦，我们知道当用户在浏览器上点击一个链接时，会产生一个 HTTP 请求，用于获取新的页面内容，而在该请求的报头中，会包含一个 Referrer，用以指定该请求是从哪个页面跳转页来的，常被用于分析用户来源等信息。但是也有成为用户的一个不安全因素，比如有些网站直接将 sessionid 或是 token 放在地址栏里传递的，会原样不动地当作 Referrer 报头的内容传递给第三方网站。所以就有了 Referrer Policy，用于过滤 Referrer 报头内容，目前是一个候选标准，不过已经有部分浏览器支持该标准。这里为 _no-referrer-when-downgrade_ 的意思是指当发生降级（比如从 https:// 跳转到 http:// ）时，不传递 Referrer 报头。但是反过来的话不受影响。通常也会当作浏览器的默认安全策略。

#### Headers 部分

![](https://upload-images.jianshu.io/upload_images/5666077-76f391a98c9b1297.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5666077-d5149867577d3b8d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来是 Response Headers 和 Request Headers，这里说实话我觉得没什么好说的，稍微有些重要的就是请求体**Content-Type**，为什么说他重要呢？我们往下看。

![](https://upload-images.jianshu.io/upload_images/5666077-7575e5856ab16c69.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来一段是**Request Payload**，**Form Data**我们比较熟悉，那这个**Request Payload**又是个什么东西呢？我们知道前端开发中经常会用到 AJAX 发送异步请求，对于 POST 类型的请求会附带请求数据。而常用的传参方式有两种，其一是 Form Data，另一个就是 Request Payload 了。

那这两者有何区别呢？其实区别主要就是在**Content-Type**上，这也就是为啥我说他重要的原因。

#### Form Data 和 Request Payload 区别

1. 如果请求头里设置`Content-Type: application/x-www-form-urlencoded`，那么这个请求被认为是表单请求，参数出现在 Form Data 里，格式为 key=value&key=value&key=value...
2. 原生的 AJAX 请求头里设置`Content-Type:application/json`，或者使用默认的请求头`Content-Type:text/plain`参，数会显示在 Request payload 块里提交，参数格式为 JSON 格式：`{"key":"value","key":"value"…}`，可读性会更好。

### Fetch API

既然 XHR 这么方便，是不是就没有不足之处呢？当然不是。XHR 很实用，但并不是一个设计优良的 API，在设计上并不符合职责分离原则，输入、输出以及状态都杂糅在同一对象中，并用事件机制来跟踪状态变化。并且，基于事件的模型与最近流行的 Promise 和 generator 异步编程模型不太友好。因此 Fetch API 横空出世，它旨在修正上述缺陷，它提供了与 HTTP 语义相同的 JS 语法，简单来说，它引入了 `fetch()` 这个实用的方法来获取网络资源。

当然由于文章篇幅有限，这里仅仅只是引出 Fetch API，推荐阅读 [http://bubkoo.com/2015/05/08/introduction-to-fetch/](http://bubkoo.com/2015/05/08/introduction-to-fetch/)。

## 最后

其实刚开始只是想简单介绍下 AJAX 的原理，但是后来发现用了班主任让班长找小明这例子之后，AJAX 的原理似乎也就明白了，便想着要不就扩展点吧，以至于整篇文章有将近一半的篇幅在写扩展的知识了。不过也不算喧宾夺主，毕竟也是 AJAX 衍生出的知识点。

如果你对本篇文章的内容有所疑问，可以在评论区写下你的观点；如果你觉得本篇文章对你有所帮助，希望可以扫描下方二维码关注公众号「01 二进制」。您的支持是我前进的最大动力！

## 参考

1. [「每日一题」AJAX 是什么？](https://zhuanlan.zhihu.com/p/22564745)

2. [Ajax 原理一篇就够了](https://juejin.im/post/5b1cebece51d4506ae71addf)
3. [HTTP 请求中的 form data 和 request payload 的区别](https://www.cnblogs.com/btgyoyo/p/6141480.html)
4. [Form Data vs Request Payload](http://xiaobaoqiu.github.io/blog/2014/09/04/form-data-vs-request-payload/)
5. [微信开放文档](https://developers.weixin.qq.com/miniprogram/dev/api/network/request/wx.request.html)
6. [fetch API 简介](http://bubkoo.com/2015/05/08/introduction-to-fetch/)
7. [Referrer Policy 介绍](https://imququ.com/post/referrer-policy.html)

---

![](https://upload-images.jianshu.io/upload_images/5666077-e3a4d5b87f8ef4d4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
