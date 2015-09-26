---
layout: post
title: Learning Tornado
date: 2015-05-07 12:19:00
meta: Tornado
digest: "学习python web框架——Tornado。主要是从 Tornadoweb 上的文档中的重要部分记录下来。<br/>
Tornado 和现在的主流 Web 服务器框架（包括大多数 Python 的框架）有着明显的区别：它是非阻塞式服务器，而且速度相当快。得利于其非阻塞的方式和对 epoll 的运用，Tornado 每秒可以处理数以千计的连接，因此 Tornado 是实时 Web 服务的一个理想框架。"
tags: [Python]
category: "Web development"
---

学习python web框架——Tornado。主要是从 [Tornadoweb][1] 上的文档中的重要部分记录下来。

> Tornado 和现在的主流 Web 服务器框架（包括大多数 Python 的框架）有着明显的区别：它是**非阻塞式**服务器，而且速度相当快。得利于其非阻塞的方式和对 **epoll** 的运用，Tornado 每秒可以处理数以千计的连接，因此 Tornado 是实时 Web 服务的一个理想框架。

开发 Tornado 主要是为了实现 Facebook 的 [FriendFeed][2] 功能——在 FriendFeed 的应用里每一个活动用户都会保持着一个服务器连接。

如下是 Tornado 的“hello world”程序：

<pre><code class="python">
import tornado.ioloop
import tornado.web

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("Hello, world")

application = tornado.web.Application([
    (r"/", MainHandler),
])

if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
</code></pre>

从上面代码中可以看出一个 tornado 程序中由 RequestHandler 来对 HTTP Request 请求进行处理，对请求路径与 RequestHandler 进行封装后传给 Application 来进行响应。


### 模块索引

最重要的一个模块是**web**， 它就是包含了 Tornado 的大部分主要功能的 Web 框架。其他模块都是工具性质的，以扩充 tornado 功能。

- 主要模块
    + web - FriendFeed 使用的基础 Web 框架，包含了 Tornado 的大多数重要的功能
    + escape - XHTML, JSON, URL 的编码/解码方法
    + database - 对 MySQLdb 的简单封装，使其更容易使用
    + template - 基于 Python 的 web 模板系统
    + httpclient - 非阻塞式 HTTP 客户端，它被设计用来和 web 及 httpserver 协同工作
    + auth - 第三方认证的实现（包括 Google OpenID/OAuth、Facebook Platform、Yahoo BBAuth、FriendFeed OpenID/OAuth、Twitter OAuth）
    + locale - 针对本地化和翻译的支持
    + options - 命令行和配置文件解析工具，针对服务器环境做了优化
              
- 底层模块
    + httpserver - 服务于 web 模块的一个非常简单的 HTTP 服务器的实现
    + iostream - 对非阻塞式的 socket 的简单封装，以方便常用读写操作
    + ioloop - 核心的 I/O 循环


### 请求处理程序和请求参数

Tornado 的 Web 程序会将 URL 或者 URL 范式映射到 tornado.web.RequestHandler 的子类上去。在其子类中定义了 get() 或 post() 方法，用以处理不同的 HTTP 请求。

下面的代码将 URL 根目录 / 映射到 MainHandler，还将一个 URL 范式 /story/([0-9]+) 映射到 StoryHandler。*正则表达式匹配的分组会作为参数引入到相应方法中*：

<pre><code class="python">
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("You requested the main page")

class StoryHandler(tornado.web.RequestHandler):
    def get(self, story_id):
        self.write("You requested the story " + story_id)

application = tornado.web.Application([
    (r"/", MainHandler),
    (r"/story/([0-9]+)", StoryHandler),
])
</code></pre>


你可以使用 **get_argument()** 方法来获取查询字符串参数，以及解析 POST 的内容：

<pre><code class="python">
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("""&lt;html&gt;&lt;body&gt;&lt;form action="g" method="post"&gt;
                   &lt;input type="text" name="message"&gt;
                   &lt;input type="submit" value="Submit"&gt;
                   &lt;/form&gt;&lt;/body&gt;&lt;/html&gt;""")

    def post(self):
        self.set_header("Content-Type", "text/plain")
        self.write("You wrote " + self.get_argument("message"))
</code></pre>

上传的文件可以通过 **self.request.files** 访问到，该对象将名称（HTML元素 `<input type="file">` 的 name 属性）对应到一个文件列表。每一个文件都以字典的形式存在，其格式为 {"filename":..., "content_type":..., "body":...}。

如果你想要返回一个错误信息给客户端，例如“403 unauthorized”，只需要抛出一个 tornado.web.**HTTPError** 异常：

<pre><code class="python">
if not self.user_is_logged_in():
    raise tornado.web.HTTPError(403)
</code></pre>


请求处理程序可以通过 **self.request** 访问到代表当前请求的对象。该 HTTPRequest 对象包含了一些有用的属性，包括：

+ **arguments** - 所有的 GET 或 POST 的参数
+ files - 所有通过 multipart/form-data POST 请求上传的文件
+ path - 请求的路径（ ? 之前的所有内容）
+ headers - 请求的头信息
          
你可以通过查看源代码 httpserver 模组中 HTTPRequest 的定义，从而了解到它的所有属性。


## 重写RequestHandler的方法函数

除了 get()/post()等以外，RequestHandler 中的一些别的方法函数，都是一些空函数，需要在子类中实现它们。对于一个请求的处理的代码调用次序如下：

1. 程序为每一个请求创建一个 RequestHandler 对象
2. 程序调用 initialize() 函数，这个函数的参数是 Application 配置中的关键字参数定义。（initialize 方法是 Tornado 1.1 中新添加的，旧版本中你需要重写 __init__ 以达到同样的目的） initialize 方法一般只是把传入的参数存 到成员变量中，而不会产生一些输出或者调用像 send_error 之类的方法。
3. 程序调用 prepare()。无论使用了哪种 HTTP 方法，prepare 都会被调用到，因此 这个方法通常会被定义在一个基类中，然后在子类中重用。prepare可以产生输出信息。如果它调用了finish（或send_error` 等函数），那么整个处理流程就此结束。
4. 程序调用某个 HTTP 方法：例如 get()、post()、put() 等。如果 URL 的正则表达式模式中有分组匹配，那么相关匹配会作为参数传入方法。



[1]: http://www.tornadoweb.cn
[2]: http://friendfeed.com

