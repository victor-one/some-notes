# 01 Node.js介绍

## 什么是 Node.js

### 官方定义

[Node.js](https://nodejs.org/zh-cn/) 是一个基于 **Chrome V8 引擎**的 JavaScript 运行环境。Node.js 使用了一个**事件驱动**、**非阻塞式 I/O**的模型，使其轻量又高效。Node.js 的包管理工具 npm 是全球最大的开源库生态系统。

Node.js 不是一门语言，也不是 JavaScript 的框架，也不是像Nginx一样的Web服务器 ，**Node.js 是 JavaScript 在服务器端的运行环境（平台）**。

### Node.js 的组成

在 Node.js 里运行 JavaScript，跟在 Chrome 里运行 JavaScript 有什么不同？

二者采用的是同样的 JS 引擎。在 Node.js 里写 JS，和在前端写 JS，几乎没有不同。在写法上的区别在于：Node.js 没有浏览器、页面标签相关的 API，但是新增了一些 Node.js 相关的 API。通俗来说，对于开发者而言，在前端写 JS 是用于控制浏览器；而 Node.js 环境写 JS 可以控制整个计算机。

JavaScript 的组成分为三个部分：

- ECMAScript
- DOM：标签元素相关的API
- BOM：浏览器相关的API

ECMAScript 是 JS 的语法；DOM 和 BOM 浏览器端为 JS 提供的 API。

而 Node.js 的组成分为：

- **ECMAScript**。ECMAScript 的所有语法在 Node 环境中都可以使用。
- **Node 环境**提供的一些**附加 API**(包括文件、网络等相关的 API)。

### 补充

与 PHP、JSP、Python、Perl、Ruby 的“既是语言，也是平台”不同，Node.js 的使用 JavaScript 进行编程，运行在 Chrome 的 V8 引擎上。

与 PHP、JSP 等相比（PHP、JSP、.net 都需要运行在服务器程序上，Apache、Nginx、Tomcat、IIS。 ），Node.js 跳过了 Apache、Nginx、IIS 等 HTTP 服务器，它自己不用建设在任何服务器软件之上。Node.js 的许多设计理念与经典架构（LAMP = Linux + Apache + MySQL + PHP）有着很大的不同，可以提供强大的伸缩能力。Node.js 没有 web 容器。

JS 语言非常灵活，使得它在严谨性方面不如 Java 等传统的静态语言。JS 是一门动态语言，而且融合了面向对象和函数式编程这两种编程范式。

## Node.js 的架构和依赖

Node.js 的架构如下：

![Node.js的架构](http://img.smyhvae.com/20180301_1540.png)

Node.js 内部采用 Google Chrome 的 V8 引擎，作为 JavaScript 语言解释器；同时结合自行开发的 libuv 库，**扩展了 JS 在后端的能力（比如 I/O 操作、文件读写、数据库操作等）**。使得 JS 既可以在前端进行 DOM 操作（浏览器前端），又可以在后端调用操作系统资源，是目前最简单的全栈式语言。

其次，Node 生态系统活跃，提供了大量的开源库，使得 JavaScript 语言能与操作系统进行更多的交互。

### Node.js 运行环境的核心：V8 引擎 和 libuv 库

Node.js 是 JavaScript 在服务器端的运行环境，在这个意义上，Node.js 的地位其实就是 JavaScript 在服务器端的虚拟机，类似于 Java 语言中的 Java 虚拟机。

* [V8 引擎](https://v8.dev/) ：编译和执行 JS 代码、管理内存、垃圾回收。V8 给 JS 提供了运行环境，可以说是 JS 的虚拟机。V8 引擎本身是用 C++ 写的。
* [libuv](https://zh.wikipedia.org/wiki/Libuv)： libuv 是一个专注于异步 I/O 的跨平台类库，目前主要在 Node.js 上使用。它是 Node.js 最初的作者 Ryan Dahl 为 Node.js 写的底层类库，也可以称之为虚拟机。libuv 本身是用 C 写的。

### V8 的内存限制

在一般的后端开发语言中，在基本的内存使用上没有什么限制，然而在 Node 中通过 JavaScript 使用内存时就会发现只能使用部分内存（64 位系统下约为 1.4GB，32 位系统下约为 0.7GB）。在这样的限制下，将会导致 Node 无法直接操作大内存对象。

造成这个问题的主要原因在于 Node 基于 V8 构建，所以在 Node 中使用的 JavaScript 对象基本上都是通过 V8 自己的方式来进行分配和管理的。V8 的这套内存管理机制在浏览器的应用场景下使用起来绰绰有余，足以胜任前端页面中的所有需求。但在 Node 中，这却限制了开发者随心所欲使用大内存的想法。

## Node.js 的应用

### 1、BFF 中间层

BFF，即 Backend For Frontend（服务于前端的后端）。玉伯在《[从前端技术进化到体验科技](https://mp.weixin.qq.com/s/IYddaaw2ps1wR2VT1dZWPg)》这篇文章中点出了 BFF 层的概念：

> BFF 模式下，整体分工很清晰，**后端通过 Java/C++ 等语言负责服务实现，理想情况下给前端提供的是基于领域模型的 RPC 接口，前端则在 BFF 层直接调用服务端 RPC 接口拿到数据**，按需加工消费数据，并实现人机交互。基于 BFF 模式的研发，很适合拥有前端技术背景的全栈型工程师。这种模式的好处很明显，后端可以专注于业务领域，更多从领域模型的视角去思考问题，页面视角的数据则交给前端型全栈工程师去搞定。**领域模型与页面数据是两种思维模式，通过 BFF 可以很好地解耦开，让彼此更专业高效**。

在 Web 服务里，搭建一个中间层，前端访问中间层的接口，中间层再访问后台的 Java/C++ 服务。这类服务的特点是不需要太强的服务器运算能力，但对程序的灵活性有较高的要求。这两个特点，正好和 Node.js 的优势相吻合。Node.js 非常适合用来做 BFF 层，优势如下：

- 对于前端来说：让前端**有能力自由组装后台数据**，这样可以减少大量的业务沟通成本，加快业务的迭代速度；并且，前端同学能够**自主决定**与后台的通讯方式。
- 对于后台和运维来说，好处是：安全性（不会把主服务器暴露在外面）、降低主服务器的复杂度等。

### 2、服务端渲染

**客户端渲染**（CSR / Client side render）：前端通过一大堆接口请求数据，然后通过 JS 动态处理和生成页面结构和展示。优点是**前后端分离**、减小服务器压力、局部刷新。缺点是不利于 SEO（如果你的页面然后通过 Ajax 异步获取内容，抓取工具并不会等待异步完成后再行抓取页面内容）、首屏渲染慢。

**服务端渲染**（SSR / Server Side Render）：服务器返回的不是接口数据，而是一整个页面（或整个楼层）的 HTML 字符串，浏览器直接显示即可。也就是说，在服务器端直接就渲染好了，然后一次性打包返回给前端。优点是**有利于 SEO、首屏渲染很快**。

**总结： 搜索引擎优化 + 首屏速度优化 = 服务端渲染**。

备注：这里的「服务端渲染」只是让 Node.js 做中间层，不会替代后端的，后台同学请放心。

### 3、做小型服务、小型网站的后端（基于 Express、Koa 框架）

现在很多公司的后台管理系统，都是用 Node.js 来开发接口，毕竟，后台管理系统对性能和并发的要求不是太高。有了 Node.js 之后，通过 JS 直接操作 DB，做增删改查，生成接口，极大降低了前端同学的学习门槛。

当然，有时候做 Node.js 开发，是因为：后台人力不够，所以把后台开发的一部分工作量，转移给前端同学。

### 4、做项目构建工具

前端正在广泛使用的构建工具 gulp、Webpack，就是基于 Node.js 来实现的。

### 5、 做 PC 客户端软件（基于 Electron 框架）

Electron 框架就是基于 Node.js 的，可以用来开发客户端软件。

Electron 原名为 Atom Shell，是由 GitHub 开发的一个开源框架。Electron 以 Node.js 作为运行时（runtime），以 chromium 作为渲染引擎，使开发者可以使用 JS 这种前端技术栈来开发跨平台的桌面GUI应用程序。

有一点你可能会感到惊讶：程序员们都在用的代码编辑器 VS Code 软件， 就是基于 Electron 框架来开发的。其他使用 Electron 进行开发的知名应用还有：Skype、GitHub Desktop、Slack、WhatsApp等。

还有一个例子是：电子游戏直播网站 [Twitch](https://www.twitch.tv/)，号称是国外游戏直播的鼻祖，它在 PC 端的客户端软件，就是用 Electron 框架的。你会发现，Twitch 的网站视觉，和 PC 端的视觉，几乎是一样的。如果两端都采用 JS 语言，就可以极大的复用现有的工程。

### 总结

或许，能用 Node.js 做的后台应用，Java/C++ 也能做；但是 Node.js 可以让我们多一种选择。

短期来看，Node.js 很难像 Java/C++ 那样，成为后台的主力开发语言。这并非是因为 Node.js 的性能问题，主要是因为，Node.js 还比较年轻，经验积累太少，框架的支持度不够。搞企业级服务，Node.js 敌不过 Java/C++，所以目前只能搞「轻量级」；但未来可期。

限制语言能力的不是语言本身，而是生态。

# 02 Node.js的特点

## Node.js 的特点

- 异步、非阻塞 IO 模型
- 事件循环
- 单线程
- 总结：轻量和高效

Node.js如何处理高并发？

Node.js采用的是异步的、非阻塞的模型。“单线程”指的是Node的主线程只有一个。为了确保主线程不被阻塞，主线程用于接收客户端的请求，但不会处理具体的任务。Node背后的线程池来进行长时间运行的任务（如IO操作、网络操作）的处理，线程池里的任务通过队列和事件循环的机制来执行。

## 使用 Node.js 时的劣势

- 程序运行不稳定，可能会出现服务不可用的情况
- 程序运行效率较低，每秒的请求数维持在一个较低的水平
- 前端同学对服务器端的技术不太熟悉。

# 03 Node.js开发环境安装

## 建议通过NVM进行安装

安装方法见[此处](https://web.qianguyihao.com/11-Node.js/03-Node.js%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E5%AE%89%E8%A3%85.html#node-js-%E8%BF%90%E8%A1%8C%E7%8E%AF%E5%A2%83%E5%AE%89%E8%A3%85%EF%BC%9A%E9%80%9A%E8%BF%87-nvm%EF%BC%88%E6%8E%A8%E8%8D%90%EF%BC%89)

注意，安装前需卸载原本安装的npm和node，不然无法做到使用NVM进行版本切换。

安装node之前可使用以下指令进行版本的确认和选择：

```shell
nvm list available
```

## NVM 的常用命令

> 注意，这一段说的是 NVM 的常用命令，不是 Node 的常用命令。

查看当前使用的 nvm 版本：

```shell
nvm version
```

查看本地安装的所有的 Node.js 版本：

```shell
# 方式1
nvm ls

# 方式2
nvm list
```

**安装指定版本的 Node.js：**

```shell
nvm install 版本号

# 举例
nvm install 8.10.0
```

卸载指定版本 Node.js：

```shell
nvm uninstall 版本号
```

**切换使用指定版本的 node**：

```shell
nvm use 版本号
```

**设置node的默认版本**：

```shell
nvm alias default 版本号
```

**查看全局npm包的安装路径**：

```shell
npm root -g
```

查看远程服务器端的所有 Node 版本：

```shell
nvm ls-remote
```

执行上面的命令后，在列出的版本清单中，凡是用 `Latest LTS`标注的版本，则表明是**长期维护**的版本。我们在安装时，建议安装这些版本。当然，我们也可以在网址 https://nodejs.org/en/download/releases/ 查看 LTS 的历史版本。

## Node.js 的常用命令

查看 node 的版本：

```shell
$ node -v
```

执行脚本字符串：

```shell
$ node -e 'console.log("Hello World")'
```

运行脚本文件：

```shell
$ node index.js

$ node path/index.js

$ node path/index
```

查看帮助：

```shell
$ node --help
```

**进入 REPL 环境：**

```shell
$ node
```

REPL 的全称：Read、Eval、 Print、Loop。类似于浏览器的控制台。

![img](http://img.smyhvae.com/20180301_1900.png)

如果要退出 REPL 环境，可以输入`.exit` 或 `process.exit()`。

## 包和 NPM

### 什么是包

由于 Node 是一套轻内核的平台，虽然提供了一系列的内置模块，但是不足以满足开发者的需求，于是乎出现了包（package）的概念： 与核心模块类似，就是将一些预先设计好的功能或者说 API 封装到一个文件夹，提供给开发者使用。

Node 本身并没有太多的功能性 API，所以市面上涌现出大量的第三方人员开发出来的 Package。

### 包的加载机制

如果 Node 中自带的包和第三方的包名冲突了，该怎么处理呢？原则是：

- 先在系统核心（优先级最高）的模块中找；
- 然后到当前项目中 node_modules 目录中找。

比如说：

```shell
requiere(`fs`);
```

那加载的肯定是系统的包。所以，我们尽量不要创建一些和现有的包重名的包。

### NPM 的概念

**NPM**：Node Package Manager。官方链接： https://www.npmjs.com/

随着时间的发展，NPM 出现了两层概念：

- 一层含义是 Node 的开放式模块登记和管理系统，亦可以说是一个生态圈，一个社区。
- 另一层含义是 Node 默认的模块管理器，是一个命令行下的软件，用来安装和管理 Node 模块。

Node 附带的 NPM 可能不是最新版本，可以用下面的命令，更新到最新版本：

```shell
$ npm install npm -g
```

### 配置 NPM 的全局目录

详情见[此处](https://blog.csdn.net/qq_42707594/article/details/111182778)

## NPM 的常用命令

查看 npm 当前版本：

```shell
npm -v
```

更新 npm：

```shell
npm install npm@latest -g
```

项目初始化：（执行完成后，会生成`package.json`文件）

```shell
npm init

# 快速跳过问答式界面，选择默认配置
npm init --yes
```

只在当前工程下安装指定的包：

```shell
npm install [package]
```

在全局安装指定的包：

```shell
npm install -g [package]
```

安装的包只用于开发环境，不用于生产环境：（会出现在 package.json 文件中的 devDependencies 属性中）

```shell
npm install [package] --save-dev

# 或者
npm install [package] -D
```

安装的包需要发布到生产环境：（会出现在 package.json 文件中的 dependencies 属性中）

```shell
npm install [package] --save

# 或者
npm install [package] -S
```

查看当前目录下已安装的 node 包：

```shell
npm list
```

查看全局已经安装的 node 包：

```shell
npm list -g
```

查看 npm 帮助命令：

```shell
npm --help
```

查看指定命令的帮助：

```shell
npm [指定命令] --help
```

更新指定的包：

```shell
npm update [package]
```

卸载指定的包：

```shell
npm uninstall [package]
```

查看配置信息：

```shell
npm config list
```

查看本地安装的指定包的信息，没有则显示 empty：

```shell
npm ls [package]
```

查看全局安装的指定包的信息，没有则显示 empty：

```shell
npm ls [package] -g
```

查看远程 npm 上指定包的所有版本信息：

```shell
npm info [package]
```

查看当前包的安装路径：

```shell
npm root
```

查看全局包的安装路径：

```shell
npm root -g
```

## 配置 npm 镜像源

由于 npm 默认的下载地址在国外（npmjs.com），有时候会被墙，导致无法下载或者下载很慢。因此，我们可以尝试切换成，从其他的镜像源下载 npm 包。

切换镜像源，有下面这几种方式：

- 方式 1：临时切换镜像源。
- 方式 2：切换镜像源
- 方式 3：通过 NRM 切换镜像源（最为推荐的方式）。
- 方式 4：cnpm。

### 方式 1：临时切换镜像源

安装指定包的时候，通过追加 `--registry`参数即可。格式如下：

```shell
# 格式
npm install [package] --registry [https://xxx]

# 举例：在下载安装 express 这个包的时候，临时指定镜像源为 https://registry.npm.taobao.org
npm install express --registry https://registry.npm.taobao.org
```

### 方式 2：切换镜像源

```shell
npm config set registry https://registry.npm.taobao.org
```

执行上述命令后，以后下载所有 npm 包的时候，都会改为使用淘宝的镜像源。

### 方式 3：通过 NRM 切换镜像源（推荐）

**NRM**：Node Registry Manager。作用是：**切换和管理 npm 包的镜像源**。

- 项目地址：https://www.npmjs.com/package/nrm
- GitHub 地址： https://github.com/Pana/nrm

**安装 NRM**：

```shell
	npm install -g nrm
```

**NRM 的常用命令：**

```shell
# 显示全部的镜像
nrm ls

# 使用淘宝的镜像
nrm use taobao
```

## 方式 4：安装 cnpm

- 项目地址：https://npm.taobao.org/

安装`cnpm`替换 npm（npm 由于源服务器在国外，下载包的速度较慢，cnpm 会使用国内镜像）：

```shell
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

以后我们就可以通过 cnpm 命令去安装一个包。举例如下：

```shell
# 安装 vue 这个包
cnpm install vue
```

这里的单词 `install` 可以简写成 `i`。

## Node.js 的简单使用

我们可以输入`node`命令，然后在里面写 js 的代码。

或者，也可以 通过 node 运行 指定的 js 文件。比如，编写好一个 js 文件`01.js`，然后在命令行输入：

```shell
	node 01.js
```

就可以执行这个 js 程序，直接在命令行查看运行结果。

# 04 Node.js模块化规范：CommonJS

## 前言

网站越来越复杂，js代码、js文件也越来越多，会遇到**一些问题**：

- 文件依赖
- 全局污染、命名冲突

程序模块化包括：

- 日期模块
- 数学计算模块
- 日志模块
- 登陆认证模块
- 报表展示模块等。

所有这些模块共同组成了程序软件系统。

一次编写，多次使用，才是提高效率的核心。

## 模块化的理解

### 什么是模块化

**概念**：将一个复杂的程序依据一定的规则（规范）封装成几个块（文件），并组合在一起。

模块的内部数据、实现是私有的, 只是向外部暴露一些接口(方法)与外部其它模块通信。

### 模块化的好处

- 避免命名冲突，减少命名空间污染
- 降低耦合性；更好地分离、按需加载
- **高复用性**：代码方便重用，别人开发的模块直接拿过来就可以使用，不需要重复开发类似的功能。
- **高可维护性**：软件的声明周期中最长的阶段其实并不是开发阶段，而是维护阶段，需求变更比较频繁。使用模块化的开发，方式更容易维护。
- 部署方便

## 模块化规范

### 模块化规范的引入

假设我们引入模块化，首先可能会想到的思路是：在一个文件中引入多个js文件。如下：

```html
<body>
    <script src="zepto.js"></script>
    <script src="fastClick.js"></script>
    <script src="util/login.js"></script>
    <script src="util/base.js"></script>
    <script src="util/city.js"></script>
</body>
```

但是这样做会带来很多问题：

- 请求过多：引入十个js文件，就有十次http请求。
- 依赖模糊：不同的js文件可能会相互依赖，如果改其中的一个文件，另外一个文件可能会报错。

以上两点，最终导致：**难以维护**。

于是，这就引入了模块化规范。

### 模块化的概念解读

模块化起源于 Node.js。Node.js 中把很多 js 打包成 package，需要的时候直接通过 require 的方式进行调用（CommonJS），这就是模块化的方式。

那如何把这种模块化思维应用到前端来呢？这就产生了两种伟大的 js：RequireJS 和 SeaJS。

### 模块化规范

服务器端规范：

- [**CommonJS规范**](http://www.commonjs.org/)：是 Node.js 使用的模块化规范。

CommonJS 就是一套约定标准，不是技术。用于约定我们的代码应该是怎样的一种结构。

浏览器端规范：

- [**AMD规范**](https://github.com/amdjs/amdjs-api)：是 **[RequireJS](http://requirejs.org/)** 在推广过程中对模块化定义的规范化产出。

```
- 异步加载模块；

- 依赖前置、提前执行：require([`foo`,`bar`],function(foo,bar){});   //也就是说把所有的包都 require 成功，再继续执行代码。

- define 定义模块：define([`require`,`foo`],function(){return});
```

- **[CMD规范](https://github.com/qianguyihao/Web/blob/master/11-Node.js)**：是 **[SeaJS](http://seajs.org/)** 在推广过程中对模块化定义的规范化产出。淘宝团队开发。

```
  同步加载模块；

  依赖就近，延迟执行：require(./a) 直接引入。或者Require.async 异步引入。   //依赖就近：执行到这一部分的时候，再去加载对应的文件。

  define 定义模块， export 导出：define(function(require, export, module){});
```

PS：面试时，经常会问AMD 和 CMD 的区别。

另外，还有ES6规范：import & export。

## CommonJS 的基本语法

### CommonJS 的介绍

[CommonJS](http://www.commonjs.org/)：是 Node.js 使用的模块化规范。也就是说，Node.js 就是基于 CommonJS 这种模块化规范来编写的。

CommonJS 规范规定：每个模块内部，module 变量代表当前模块。这个变量是一个对象，它的 exports 属性（即 module.exports）是对外的接口对象。加载某个模块，其实是加载该模块的 module.exports 对象。

在 CommonJS 中，每个文件都可以当作一个模块：

- 在服务器端：模块的加载是运行时同步加载的。
- 在浏览器端: 模块需要提前编译打包处理。首先，既然同步的，很容易引起阻塞；其次，浏览器不认识`require`语法，因此，需要提前编译打包。

### 模块的暴露和引入

Node.js 中只有模块级作用域，两个模块之间的变量、方法，默认是互不冲突，互不影响，这样就导致一个问题：模块 A 要怎样使用模块B中的变量&方法呢？这就需要通过 `exports` 关键字来实现。

Node.js中，每个模块都有一个 exports 接口对象，我们可以把公共的变量、方法挂载到这个接口对象中，其他的模块才可以使用。

### 暴露模块的方式一： exports

`exports`对象用来导出当前模块的公共方法或属性。别的模块通过 require 函数调用当前模块时，得到的就是当前模块的 exports 对象。

**语法格式**：

```js
// 相当于是：给 exports 对象添加属性
exports.xxx = value
```

这个 value 可以是任意的数据类型。

**注意**：暴露的关键词是`exports`，不是`export`。其实，这里的 exports 类似于 ES6 中的 export 的用法，都是用来导出一个指定名字的对象。

**代码举例**：

```js
const name = 'qianguyihao';

const foo = function (value) {
	return value * 2;
};

exports.name = name;
exports.foo = foo;
```

### 暴露模块的方式二： module.exports

`module.exports`用来导出一个默认对象，没有指定对象名。

语法格式：

```js
// 方式一：导出整个 exports 对象
module.exports = value;

// 方式二：给 exports 对象添加属性
module.exports.xxx = value;
```

这个 value 可以是任意的数据类型。

代码举例：

```js
// 方式1
module.exports = {
    name: '我是 module1',
    foo(){
        console.log(this.name);
    }
}

// 我们不能再继续写 module.exports = value2。因为重新赋值，会把 exports 对象 之前的赋值覆盖掉。

// 方式2
const age = 28;
module.exports.age = age;
```

`module.exports` 还可以修改模块的原始导出对象。比如当前模块原本导出的是一个对象，我们可以通过 module.exports 修改为导出一个函数。如下：

```js
module.exports = function () {
    console.log('hello world')
}
```

### exports 和 module.exports 的区别

最重要的区别：

- 使用exports时，只能单个设置属性 `exports.a = a;`
- 使用module.exports时，既单个设置属性 `module.exports.a`，也可以整个赋值 `module.exports = obj`。

其他要点：

- Node中每个模块的最后，都会执行 `return: module.exports`。
- Node中每个模块都会把 `module.exports`指向的对象赋值给一个变量 `exports`，也就是说 `exports = module.exports`。
- `module.exports = XXX`，表示当前模块导出一个单一成员，结果就是XXX。
- 如果需要导出多个成员，则必须使用 `exports.add = XXX; exports.foo = XXX`。或者使用 `module.exports.add = XXX; module.export.foo = XXX`。

### 问题: 暴露的模块到底是谁？

**答案**：暴露的本质是`exports`对象。【重要】

比如，方式一的 `exports.a = a` 可以理解成是，**给 exports 对象添加属性**。方式二的 `module.exports = a`可以理解成是给整个 exports 对象赋值。方式二的 `module.exports.c = c`可以理解成是给 exports 对象添加属性。

Node.js 中每个模块都有一个 module 对象，module 对象中的有一个 exports 属性称之为**接口对象**。我们需要把模块之间公共的方法或属性挂载在这个接口对象中，方便其他的模块使用。

### 引入模块的方式：require

require函数用来在一个模块中引入另外一个模块。传入模块名，返回模块导出对象。

**语法格式**：

```js
const module1 = require('模块名');
```

解释：

- 内置模块：require的是**包名**。
- 下载的第三方模块：require的是**包名**。
- 自定义模块：require的是**文件路径**。文件路径既可以用绝对路径，也可以用相对路径。后缀名`.js`可以省略。

**代码举例**：

```js
const module1 = require('./main.js');

const module2 = require('./main');

const module3 = require('Demo/src/main.js');
```

**require()函数的两个作用**：

- 执行导入的模块中的代码。
- 返回导入模块中的接口对象。

### 主模块

主模块是整个程序执行的入口，可以调度其他模块。

```shell
# 运行main.js启动程序。此时，main.js就是主模块
$ node main.js
```

### 模块的初始化

一个模块中的 JS 代码仅在模块**第一次被使用时**执行一次，并且在使用的过程中进行初始化，然后会被缓存起来，便于后续继续使用。

代码举例：

（1）calModule.js:

```js
var a = 1;

function add () {
  return ++a;
}

exports.add = add;
```

（2）main.js：（在 main.js 中引入 hello.js 模块）

```js
var addModule1 = require('./calModule')
var addModule2 = require('./calModule')

console.log(addModule1.add());
console.log(addModule2.add());
```

在命令行执行 `node main.js` 运行程序，打印结果：

```
2
3
```

从打印结果中可以看出，`calModule.js`这个模块虽然被引用了两次，但只初始化了一次。

## CommonJS 在服务器端的实现举例

### 1、初始化项目

在工程文件中新建如下目录和文件：

```
modules
    | module1.js
    | module2.js
    | module3.js

app.js
```

然后在根目录下新建如下命令：

```
  npm init
```

然后根据提示，依次输入如下内容：

- **包名**：可以自己起包名，也可以用默认的包名。注意，包名里不能有中文，不能有大写。
- **版本**：可以用默认的版本 1.0.0，也可以自己修改包名。

其他参数，一路回车即可。效果如下：

![init效果](http://img.smyhvae.com/20180410_1425.png)

于是，根目录下会自动生成`package.json`这个文件。点进去看一下：

```json
{
  "name": "commonjs_node",
  "version": "1.0.0",
  "description": "",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "smyhvae",
  "license": "ISC"
}
```

### 2、导入第三方包

`uniq`这个第三方包的作用是保证唯一性（我们拿它来举例）。我们在当前工程目录下，输入如下命令进行安装：

```shell
  npm install uniq
```

安装成功后，根目录下会自动生成相应的文件：

![导入第三方包效果](http://img.smyhvae.com/20180410_1450.png)

当输入完`npm install uniq`之后，`package.json`中就会自动添加`uniq`包的依赖：

![json添加依赖](http://img.smyhvae.com/20180410_1855.png)

如果有些童鞋的npm版本较低，就需要手动去添加依赖；另一种方式是，可以使用`npm install uniq --save`命令，这个多出来的`--save`就可以自动添加依赖。

我们去[官网](https://www.npmjs.com/package/uniq)看一下`uniq`的用法：

```js
  let uniq = require('uniq');

  let arr = [1, 1, 2, 2, 3, 5];
  uniq(arr);
  console.log(arr);  //输出结果：[ 1, 2, 3, 5 ]
```

可以看出，这个包可以起到数组去重的作用。

### 3、自定义模块

（1）module1.js：

```js
//暴露方式一：module.exports = value

//暴露一个对象出去
module.exports = {
    name: '我是 module1',
    foo(){
        console.log(this.name);
    }
}

//我们不能再继续写 module.exports = xxx。因为重新赋值，会把之前的赋值覆盖掉。
```

（2）module2.js：

```js
//暴露方式一：module.exports = value

//暴露一个函数出去
module.exports = function(){
    console.log('我是 module2');
}
```

注意，此时暴露出去的 exports 对象 等价于整个函数。

（3）module3.js：

```js
//暴露方式二：exports.xxx = value

//可以往 export 对象中不断地添加属性，进行暴露

exports.foo1 = function(){
    console.log('module3 中的 foo1 方法');
}

exports.foo2 = function(){
    console.log('module3 中的 foo2 方法');
}

exports.arr = [1,1,2,2,3,5,11];
```

（4）app.js：（将其他模块汇集到主模块）

```js
//将其他模块汇集到主模块

let uniq = require('uniq'); //引入时，第三方模块要放在自定义模块的上面

let module1 = require('./modules/module1');
let module2 = require('./modules/module2');
let module3 = require('./modules/module3');

//调用module1对象的方法
module1.foo();

//调用module2的函数
module2();  //注意，在定义时，module2对象等价于整个函数function。所以，module2()的意思是，直接调用了函数。

//调用module3中的属性
module3.foo1();
module3.foo2();

uniq(module3.arr); //将module3中的数组进行去重操作
console.log(module3.arr); //打印数组去重后的结果
```

这样的话，我们的代码就写完了。

我们在命令行中输入`node app.js`，就可以把代码跑起来了。打印结果如下：

```
我是 module1
我是 module2
module3 中的 foo1 方法
module3 中的 foo2 方法
[ 1, 11, 2, 3, 5 ]
```

## CommonJS 基于浏览器端的实现举例

### 1、初始化项目

在工程文件中新建如下目录和文件：

```
js
    dist //打包生成文件的目录
    src //源码所在的目录
      | module1.js
      | module2.js
      | module3.js
      | app.js //应用主源文件
index.html    //因为CommonJS是基于浏览器端，js文件要跑在浏览器的页面上，所以要有这个html页面
```

然后在根目录下新建如下命令：

```shell
  npm init
```

然后根据提示，依次输入如下内容：

- **包名**：可以自己起包名，也可以用默认的包名。注意，包名里不能有中文，不能有大写。
- **版本**：可以用默认的版本 1.0.0，也可以自己修改包名。

其他的参数，一路回车即可。

于是，根目录下会自动生成`package.json`这个文件。点进去看一下：

```json
{
  "name": "commonjs_browser",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

### 2、下载第三方包：Browserify

这里需要用到[Browserify](http://browserify.org/)这个工具进行编译打包。Browserify 称为 CommonJS 的浏览器端的打包工具。

输入如下命令进行安装：（两个命令都要输入）

```shell
    npm install browserify -g          //全局
    npm install browserify --save-dev  //局部。
```

上面的代码中，`-dev`表示开发依赖。这里解释一下相关概念：

- 开发依赖：当前这个包，只在开发环境下使用。
- 运行依赖：当前这个包，是在生产环境下使用。

### 3、自定义模块 & 代码运行

（1）module1.js：

```js
//暴露方式一：module.exports = value

//暴露一个对象出去
module.exports = {
    name: '我是 module1',
    foo(){
        console.log(this.name);
    }
}

//我们不能再继续写 module.exports = xxx。因为重新赋值，会把之前的赋值覆盖掉。
```

（2）module2.js：

```js
//暴露方式一：module.exports = value

//暴露一个函数出去
module.exports = function(){
    console.log('我是 module2');
}
```

注意，此时暴露出去的 exports 对象 等价于整个函数。

（3）module3.js：

```js
//暴露方式二：exports.xxx = value

//可以往export对象中不断地添加属性，进行暴露

exports.foo1 = function(){
    console.log('module3 中的 foo1 方法');
}

exports.foo2 = function(){
    console.log('module3 中的 foo2 方法');
}
```

（4）app.js：（将其他模块汇集到主模块）

```js
let module1 = require('./module1');  // ./ 指的是当前路径
let module2 = require('./module2');
let module3 = require('./module3');

module1.foo();
module2();
module3.foo1();
module3.foo2();
```

引入的路径解释：

- `./`是相对路径，指的是当前路径（app.js的当前路径是src）

到此，我们的主要代码就写完了。

但是，如果我们直接在index.html中，像下面这样写，是不行的：（因为浏览器不认识 require 关键字）

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <script src="./js/src/app.js"></script>
</body>
</html>
```

为了能够让index.html引入app.js，我们需要输入如下命令：

打包处理js:

```shell
    browserify js/src/app.js -o js/dist/bundle.js
```

然后在index.html中引入打包后的文件：

```html
    <script type="text/javascript" src="js/dist/bundle.js"></script>
```

# 05 Node.js内置模块：fs文件模块

## Node.js 的官方API文档

- Node.js 的API文档（英文）： https://nodejs.org/docs/latest-v8.x/api/index.html
- Node.js 的API文档（中文）：http://nodejs.cn/api/

关于 Node.js 的内置模块和常见API，可以看官方文档。

查阅文档时，稳定指数如下：

- 红色：废弃。
- 橙色：实验。表示当前版本可用，其他版本不确定。也许不向下兼容，建议不要在生产环境中使用该特性。
- 绿色：稳定。与 npm 生态系统的兼容性是最高的优先级。

## Node.js 中模块的分类

Node.js 应用由模块组成，采用 CommonJS 模块规范。Node.js中的模块分为三种：

- 内置模块
- 第三方模块
- 自定义模块

### 1、内置模块

```js
const process = require('process');
const path = require('path');

console.log(process.version);
console.log(path.resolve('../'));
```

require方法用于加载模块。

常见的内置模块包括：

- FS：文件系统模块
- path：路径模块
- OS：操作系统相关
- net：网络相关
- http
- ...

JS本身是没有能力获取底层系统资源的，这一切都是 JS虚拟机在和底层做交互，然后通过 JS 的表现形式，暴露给应用层。

另外，还有很多库，是直接使用C/++编写的，通过编译之后，再提供给 JS 应用层调用，或者直接提供给 Node.js层使用。

**所有的编程语言底层都会回归C/C++**，甚至是汇编语言。

### 2、require 加载第三方包的机制

```js
const express = require('express');
```

require 加载第三方包的机制：

（1）第三方包安装好后，这个包一般会存放在当前项目的 node_modules 文件夹中。我们找到这个包的 package.json 文件，并且找到里面的main属性对应的入口模块，这个入口模块就是这个包的入口文件。

（2）如果第三方包中没有找到package.json文件，或者package.json文件中没有main属性，则默认加载第三方包中的index.js文件。

（3）如果在 node_modules 文件夹中没有找到这个包，或者以上所有情况都没有找到，则会向上一级父级目录下查找node_modules文件夹，查找规则如上一致。

（4）如果一直找到该模块的磁盘根路径都没有找到，则会报错：can not find module xxx。

### 3、自定义模块（module）：

每个文件就是一个模块，有自己的作用域。在一个文件里面定义的变量、函数、类，都是私有的，对其他文件不可见。

举例：

```js
var example = require('./example.js');
console.log(example.x); // 5
console.log(example.addX(1)); // 6
```

## 读取文件

今天这篇文章，重点讲一下 Node 内置模块中的 **fs（文件处理模块）**。

在使用文件模块之前，记得先导入：

```js
// 导入文件模块
const fs = require('fs');
```

fs 的英文全称是 File System。fs 模块提供了很多 api 方法，我们首先应该学习的方法是**文件读取**。

Node中文件读取的方式主要有以下几种。

### 异步读取文件 fs.readFile()

语法格式：

```js
fs.readFile(file[, options], callback(error, data))
```

代码举例：

```js
const fs = require('fs');

fs.readFile('hello.txt', 'utf8', (err, data) => {
    if (err) {
        // 失败
        console.log(err)
    } else {
        // 成功
        console.log('异步读取数据：' + data2)
    }
});
```

如果需要嵌套读取多个文件，可以用 promise 或者 async ... await 进行封装。代码举例如下。

### promise 封装 fs.readFile()

```js
const fs = require('fs');

function fsRead(path) {
    return new Promise((resolve, reject) => {
        fs.readFile(path, { flag: 'r', encoding: "utf-8" }, (err, data) => {
            if (err) {
                //失败执行的内容
                reject(err)
            } else {
                //成功执行的内容
                resolve(data)
            }
        })
    })
}

var promise1 = fsRead('hello1.txt')
promise1.then(res1 => {
    console.log(res1);
    return fsRead('hello2.txt');
}).then(res2 => {
    console.log(res2);
    return fsRead('hello3.txt');
}).then(res3 => {
    console.log(res);
})
```

### async ... await 封装 fs.readFile()

这个写法更为简洁，推荐。

```js
var fs = require('fs');

function fsRead(path) {
    return new Promise((resolve, reject) => {
        fs.readFile(path, { flag: 'r', encoding: "utf-8" }, (err, data) => {
            if (err) {
                //失败执行的内容
                reject(err)
            } else {
                //成功执行的内容
                resolve(data)
            }
        })
    })
}

async function ReadList() {
    var res1 = await fsRead('hello1.txt');
    var res2 = await fsRead('hello2.txt');
    var res3 = await fsRead('hello3.txt');
}

// 执行方法
ReadList();
```

### 同步读取文件 fs.readFileSync()

语法格式：

```js
fs.readFileSync(file[, options])
```

代码举例：

```js
const fs = require('fs');

try {
  const data = fs.readFileSync('hello.txt', 'utf8');
  console.log(data);
} catch(e) {
  // 文件不存在，或者权限错误
  throw e;
}
```

### Node.js 中的同步和异步的区别

fs模块对文件的几乎所有操作都有同步和异步两种形式。例如：readFile() 和 readFileSync()。

区别：

- 同步调用会阻塞代码的执行，异步则不会。
- 异步调用会将 读取任务 下达到任务队列，直到任务执行完成才会回调。
- 异常处理方面：同步必须使用 try catch 方式，异步可以通过回调函数的第一个参数。【重要】

## 写入文件

语法格式：

```js
fs.write(fd, string[, position[, encoding]], callback)
```

async ... await 封装：

```js
let fs = require('fs')

function writeFs(path, content) {
    return new Promise(function (resolve, reject) {
        fs.writeFile(path, content, { flag: "a", encoding: "utf-8" }, function (err) {
            if (err) {
                //console.log("写入内容出错")
                reject(err)
            } else {
                resolve(err)
                //console.log("写入内容成功")
            }
        })
    })
}


async function writeList() {
    await writeFs('1.html', "<h1>qianguyihao</h1>");
    await writeFs('2.html', "<h1>hello world</h1>");
    await writeFs('3.html', "<h1>永不止步</h1>");
}

writeList()
```

## 删除文件

语法格式：

```js
fs.unlink(path, callback)
```

参数说明：

- path：文件路径。
- callback：回调函数。

代码举例：

```js
fs.unlink('path/file.txt', (err) => {
    if (err) throw err;
    console.log('文件删除成功');
});
```

备注：`fs.unlink()` 不能用于删除目录。 如果要删除目录，可以使用 `fs.rmdir()`。

## Buffer

通过 Buffer 开辟的内存空间，都是连续的内存空间，所以效率比较高。

代码举例1：

```js
// 将字符串转成 buffer 对象
const str = 'qianguyihao';
let buffer = Buffer.from(str);

console.log(buffer); // 输出16进制编码
console.log(buffer.toString()); // 输出字符串：qianguyihao
```

代码举例2：

```js
// 从内存中开辟一个新的缓冲区
let buffer = Buffer.alloc(20);
buffer[0] = 'a';

console.log(buffer);
```

## 读取目录

语法格式：

```js
fs.mkdir(path[, options], callback)
```

参数说明：

- path：文件路径。
- options参数可以是：
  - recursive：是否以递归的方式创建目录，默认为 false。
  - mode：设置目录权限，默认为 0777。

代码举例：

```
var fs = require("fs");

console.log("查看 /tmp 目录");
fs.readdir("/tmp/",function(err, files){
   if (err) {
       return console.error(err);
   }
   files.forEach( function (file){
       console.log( file );
   });
});
```

其他的还有：（暂略）

- 删除目录
- 输入输出

# 06 Node.js内置模块：path路径模块

## path 路径模块

Node.js 通过`path`这个内置模块，提供了一些路径操作的API，具体可以参考官方的api文档。这里列举一些常用的API。

### path.extname() 获取文件/路径的扩展名

语法格式：

```js
 path.extname(myPath);
```

代码解释：

- 获取 `myPath` 这个文件或者路径的扩展名。
- `myPath` 这个参数要求是字符串。如果 `myPath` 不是字符串，则抛出 TypeError。

代码举例：

```js
const path = require('path');

path.extname('hello.txt'); // 返回 '.txt'

path.extname('www.qianguyihao.com'); // 返回 '.com'

path.extname('index.coffee.md');  // 返回 '.md'

path.extname('index.');  // 返回 '.'

path.extname('index');  // 返回 ''

path.extname('.index');  // 返回 ''

path.extname('.index.md');  // 返回 '.md'
```

### path.resolve() 生成完成的绝对路径

语法格式：

```js
path.resolve([...myPaths])
```

解释：

- 将路径或路径片段的序列解析为绝对路径。
- 返回的路径是**从右往左**处理，后面的每个 myPath 被依次解析，直到构造出一个完整的绝对路径。

代码举例：

```js
const path = require('path');

let arr1 = ['/foo1/foo2', 'qianguyihao', 'foo3'];
let result1 = path.resolve(...arr1);
console.log(result1); // 打印结果：/foo1/foo2/qianguyihao/foo3

let arr2 = ['/foo1/foo2', '/qianguyihao', 'foo3'];
let result2 = path.resolve(...arr2);
console.log(result2); // 打印结果：/qianguyihao/foo3
```

### 几个常见路径

- `__dirname`：这是一个常量，表示：当前执行文件所在**完整目录**。
- `__filename`：这是一个常量。表示：当前执行文件的**完整目录 + 文件名**。
- `process.cwd`：获取当前执行 Node命令 时的目录名。

代码举例：

```js
console.log(__dirname);

console.log(__filename);

console.log(process.cwd());
```

运行结果：

```
$ node app.js

/Users/smyhvae/qianguyihao
/Users/smyhvae/qianguyihao/app.js
/Users/smyhvae/qianguyihao
```

### path.join() 将多个路径进行拼接

如果是我们手动拼接路径，容易出错。这个时候，可以利用 path.join() 方法将路径进行拼接。

语法格式：

```js
path.join([...paths]);
```

解释：使用平台特定的分隔符作为定界符将所有给定的 path 片段连接在一起，然后规范化生成的路径。

代码举例：

```js
const path = require('path');

const result1 = path.join(__dirname, './app.js');
console.log(result1); // 返回：/Users/smyhvae/qianguyihao/app.js

const result2 = path.join('/foo1', 'foo2', './foo3');
console.log(result2); // 返回：/foo1/foo2/foo3

const result3 = path.join('/foo1', 'foo2', '/foo3');
console.log(result3); // 返回：/foo1/foo2/foo3
```

## OS 系统模块

- os.networkInterfaces() 查看网络地址

# 07 Node.js操作MySQL数据库

## Node.js 连接 MySQL

（1）安装 mysql 包：

```shell
$ npm install mysql
```

（2）引入 mysql 包：

```js
const mysql = require("mysql");
```

（3）建立连接：

```js
let mysql = require("mysql");
let options = {
  host: "localhost",
  //port:"3306", //可选，默认3306
  user: "root",
  password: 'xxx', // 这里改成你自己的数据库连接密码
  database: "qiangu_database",
};
//创建与数据库进行连接的连接对象
let connection = mysql.createConnection(options);

//建立连接
connection.connect((err) => {
  if (err) {
      // 数据库连接失败
    console.log(err);
  } else {
      // 数据库连接成功
    console.log("数据库连接成功");
  }
});
```

正常来说，运行程序后，应该会提示`数据库连接成功`。

如果在运行时提示错误`Client does not support authentication protocol requested by server`，解决办法如下：(在终端进入 sql 之后，输入如下命令)

```mysql
# 注意，这里的 'root' 请填你的user账号， 'localhost' 请填 你的 host， 'password' 请填你的密码
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';

# 然后执行如下命令
flush privileges;
```

## Node.js 增删改查 MySQL

### 1、查询表

```js
let mysql = require('mysql');
let options = {
    host: 'localhost',
    //port:"3306",//可选，默认3306
    user: 'root',
    password: 'xxx', // 这里改成你自己的数据库密码
    database: 'qiangu_database'
}
//创建与数据库进行连接的连接对象
let connection = mysql.createConnection(options);

//建立连接
connection.connect((err) => {
    if (err) {
        // 数据库连接失败
        console.log(err)
    } else {
        // 数据库连接成功
        console.log('数据库连接成功')
    }
});


// 1、查询表
let strSql1 = 'select * from qiangu_student_table';
connection.query(strSql1, (err, result, fields) => {
    if (err) {
        // 表查询失败
        console.log(err);
    } else {
        // 表查询成功
        console.log('qiangu_student_table 表查询结果：' + JSON.stringify(result));
        console.log('fields:' + JSON.stringify(fields));
    }
})
```

打印结果如下：

```
qiangu_student_table 表查询结果：
[{"id":1,"name":"千古壹号","age":28},{"id":2,"name":"许嵩","age":34},{"id":3,"name":"邓紫棋","age":28}]

fields:[
    {"catalog":"def","db":"qiangu_database","table":"qiangu_student_table","orgTable":"qiangu_student_table","name":"id","orgName":"id","charsetNr":63,"length":11,"type":3,"flags":0,"decimals":0,"zeroFill":false,"protocol41":true},
    {"catalog":"def","db":"qiangu_database","table":"qiangu_student_table","orgTable":"qiangu_student_table","name":"name","orgName":"name","charsetNr":33,"length":765,"type":253,"flags":0,"decimals":0,"zeroFill":false,"protocol41":true},
    {"catalog":"def","db":"qiangu_database","table":"qiangu_student_table","orgTable":"qiangu_student_table","name":"age","orgName":"age","charsetNr":63,"length":11,"type":3,"flags":0,"decimals":0,"zeroFill":false,"protocol41":true}
]
```

### 删除表

```js
// 2、删除表
let strSql2 = 'drop table test2_table';
connection.query(strSql2, (err, result) => {
    if (err) {
        // 表删除失败
        console.log(err);
    } else {
        // 表删除成功
        console.log('表删除成功：' + result);
    }
});
```

打印结果：

```
表删除成功：
OkPacket {
    fieldCount: 0,
    affectedRows: 0,
    insertId: 0,
    serverStatus: 2,
    warningCount: 0,
    message: '',
    protocol41: true,
    changedRows: 0
}
```

### 删除数据库

将上方的sql语句换一下即可：

```js
let strSql3 = 'drop database qiangu_database';
```

### 2、新建数据库

```js
let mysql = require('mysql');
let options = {
    host: 'localhost',
    //port:"3306",//可选，默认3306
    user: 'root',
    password: 'smyhvae001',
    // database: 'qiangu_database'  // 注意，因为代码里是创建新的数据库，所以这里不需要填其他的数据库名
}
//创建与数据库进行连接的连接对象
let connection = mysql.createConnection(options);

//建立连接
connection.connect((err) => {
    if (err) {
        // 数据库连接失败
        console.log(err);
    } else {
        // 数据库连接成功
        console.log('数据库连接成功')
    }
});

// 创建新的数据库
const strSql4 = 'create database qiangu_database3';
connection.query(strSql4, (err, result) => {
    if (err) {
        console.log(err);
    } else {
        console.log('新建数据库成功：' + JSON.stringify(result));
    }

});
```

打印结果：

```
数据库连接成功
新建数据库成功：{
    "fieldCount":0,"affectedRows":1,"insertId":0,"serverStatus":2,"warningCount":0,"message":"","protocol41":true,"changedRows":0
}
```

### 3、新建表

新建表的sql语句举例：

```mysql
CREATE TABLE `qiangu_table5` (
    `id` int NOT NULL AUTO_INCREMENT,
    `name` varchar(255) DEFAULT NULL,
    `age` int DEFAULT NULL,
    PRIMARY KEY (`id`)
);
```

如果是在 js 代码中执行上面这样命令的话，要记得把 sql 语句存放在字符串里的同一行。

代码举例如下：

```js
let mysql = require('mysql');
let options = {
    host: 'localhost',
    //port:"3306",//可选，默认3306
    user: 'root',
    password: 'smyhvae001',
    database: 'qiangu_database'
}
//创建与数据库进行连接的连接对象
let connection = mysql.createConnection(options);

//建立连接
connection.connect((err) => {
    if (err) {
        // 数据库连接失败
        console.log(err);
    } else {
        // 数据库连接成功
        console.log('数据库连接成功')
    }
});


// 新建表
// 注意，在 js 代码中，sql 语句要存放在字符串里的同一行。
const strSql5 = 'CREATE TABLE `qianguyihao_table5` (`id` int NOT NULL AUTO_INCREMENT,`name` varchar(255) DEFAULT NULL,`age` int DEFAULT NULL,PRIMARY KEY (`id`));';

connection.query(strSql5, (err, result) => {
    if (err) {
        // 新建表失败
        console.log(err);
    } else {
        // 新建表成功
        console.log('qianguyihao 新建表成功：' + JSON.stringify(result));
    }
})
```

打印结果：

```
数据库连接成功
qianguyihao 新建表成功：
{
    "fieldCount":0,"affectedRows":0,"insertId":0,"serverStatus":2,"warningCount":0,"message":"","protocol41":true,"changedRows":0
}
```

### 在指定的表中插入数据

在指定的表中插入数据：

```js
// 在指定的表中插入数据
const strSql6 = "insert into qianguyihao_table5 (name, age) values ('千古壹号', '28')";

connection.query(strSql6, (err, result) => {
    if (err) {
        // 插入数据失败
        console.log(err);
    } else {
        // 在指定的表中插入数据成功
        console.log('qianguyihao 在指定的表中插入数据成功：' + JSON.stringify(result));
    }
});
```

打印结果：

```
qianguyihao 在指定的表中插入数据成功：
{
    "fieldCount":0,"affectedRows":1,"insertId":1,"serverStatus":2,"warningCount":0,"message":"","protocol41":true,"changedRows":0
}
```

如果插入的数据是变量（比如是用户提交上来的数据），那么，sql 语句可以这样写：

```js
// 在指定的表中插入数据（数据作为变量）
const strSql7 = "insert into qianguyihao_table5 (name, age) values (?, ?)";

connection.query(strSql7, ['许嵩', '34'], (err, result) => {
    if (err) {
        // 插入数据失败
        console.log(err);
    } else {
        // 在指定的表中插入数据成功
        console.log('qiangauyihao 在指定的表中插入数据成功：' + JSON.stringify(result));
    }
});
```
