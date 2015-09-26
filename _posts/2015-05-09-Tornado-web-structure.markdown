---
layout: post
title: Tornado Web 应用程序结构解析
date: 2015-05-09
meta: Tornado 
digest: "tornado web application由一个Application类和至少一个RequestHandler子类构成，然后由main()函数启动server。<br/>
Application负责全局配置，包括requests到handlers的路由表。<br/>
Tornado web application 大部分工作由该 RequestHandler 的子类完成。该子类可实现 HTTP 方法如 get() 和 post()，捕获从上述 Application 正则表达式得来的参数进行相应处理。"
---

Tornado web application 由一个 Application 类和至少一个 RequestHandler 子类构成，然后由 main() 函数启动 server。

"hello world"样例程序：

<pre><code class="python">
    from tornado.ioloop import IOLoop
    from tornado.web import RequestHandler, Application, url
    
    class HelloHandler(RequestHandler):
        def get(self):
            self.write("Hello, world")
    
    def make_app():
        return Application([
            url(r"/", HelloHandler),
            ])
    
    def main():
        app = make_app()
        app.listen(8888)
        IOLoop.current().start()    
</code></pre>

### 1. 注意线程安全
通常RequestHandler中的方法（包括tornado中的其他方法）**不是**线程安全的。特别的如write(), finish(), flush()必须且只能从主线程中调用。如果你在多线程中使用它，需要使用**IOLoop.add_callback**在request完成前将控制返回给主线程。


### 2. Application类
Application负责全局配置，包括requests到handlers的路由表。该路由表是一个 **[URLSpec]** 类或元组构成的链表，其中包含一个正则表达式和一个handler类，**出现顺序决定匹配顺序**。Tornado会自动在 pattern 的前后分别加上'^'和'&'字符，也即 url 与 pattern 需要完全匹配。如果正则表达式含有捕获组(capture groups)，这些组会作为 path arguments 传递给 handler 的 HTTP 方法中作为参数。

URLSpec 在 Tornado.web 中定义，对它介绍一下：

参数:

1. **pattern**  : Regular expression to be matched. Any groups in the regex will be passed in to the handler’s get/post/etc methods as arguments.
2. **handler**  : RequestHandler subclass to be invoked.
3. **kwargs**   : (optional): A dictionary of additional arguments to be passed to the handler’s constructor.
4. **name**     : (optional): A name for this handler. Used by Application.reverse_url.

pattern 中匹配的分组可以作为 get 或 post 等方法的变量，在 kwargs 中传入的变量将传给 RequestHandler.initialize 函数，第四个参数 name 用于 RequestHandler.reverse_url 函数中。


### 3. RequestHandler 类
tornado web application 大部分工作由该 RequestHandler 的子类完成。该子类可实现 HTTP 方法如 get() 和 post()，捕获从上述 Application 正则表达式得来的参数进行相应处理。

在 handler 中，使用 RequestHandler.render 或 RequestHandler.write 来产生响应。render() 通过 html 文件名加载 Template，将所给参数传入其中，对模板进行渲染后返回。write() 用于无模板时输出。接受字节组、字符串或者字典（将字典编码为 JSON）。

RequestHandler 类中有很多方法设计在子类中实现并用于整个 application。通常我们会定义一个 BaseHandler，复写 write_error 和 get_current_user，然后继承 BaseHandler。


#### 3.1 处理请求输入
RequestHandler 中有 self.request 变量代表当前请求类，它是 HTTPServerRequest 的实例。

可以通过 get_query_argument(url query) 和 get_body_argument(html form) 来获得传入参数
。

<pre><code class="python">
    class MyFormHandler(RequestHandler):
        def get(self):
            self.write("""&lt;html&gt;&lt;body&gt;&lt;form action="/myform" method="POST"&gt;
                       &lt;input type="text" name="message"&gt;
                       &lt;input type="submit" value="Submit"&gt;
                       &lt;/form&gt;&lt;/body&gt;&lt;/html&gt;""")
    
        def post(self):
            self.set_header("Content-Type", "text/plain")
            self.write("You wrote " + self.get_body_argument("message"))
</code></pre>

由于 html form 编码模糊（不知将参数当做单值还是单值列表），需要区分参数是单个值还是单值列表。 RequestHandler 由 get_query_arguments 和 get_body_arguments 来表示程序期望参数为列表。
由于 html form 的模糊编码问题， tornado 不会尝试统一 form 参数和其他类型输入。 特别的， 不会解析 JSON request body，应用程序可以使用 prepare 函数来实现 JSON 代替 form-encoding。

<pre><code class="python">
    def prepare(self):
        if self.request.headers["Content-Type"].startswith("application/json"):
            self.json_args = json.loads(self.request.body)
        else:
            self.json_args = None
</code></pre>

prepare函数在get/post调用前调用，常用语request方法之前的handler初始化操作。
异步支持： 用gen.coroutine或return_future装饰本方法实现异步（asynchronous修饰符不能用于prepare）。如果函数返回Future，则知道Future完成程序才会处理。

#### 3.2 重写 RequestHandler 方法
在每次请求产生时，触发如下调用顺序：

1. 每个 request 产生一个新的 RequestHandler 类创建
2. intialize() 被调用（使用从 Application 获得的初始化参数），典型应用是函数保存参数到成员变量中。本函数不产生输出或调用方法如 send_error
3. prepare() 被调用。最有用的是在所有 handler 的基类中定义该函数，因此这个函数会在所有 HTTP 方法之前被调用。函数可产生输出，如果调用 finish 或 redirect，则处理停止，不往下走
4. HTTP 方法被调用。从 url 正则表达式捕获的组会被传入方法作为参数。
5. 当 request 结束时，on_finish 被调用。对同步来说，在 get() 等函数之后，对异步函数来说在 finish() 函数之后。

如下是经常实现的函数，要查看所有复写函数请参见RequestHandler文档。

*   write_error - 输出html用于错误页面
*   on_connection_close - 当客户端断开连接时调用；应用可以可以选择检测这种情况，挂起等待进一步处理 注意部保证关闭了的联接能被迅速检测到。
*   get_current_user - 参见*用户认证*。
*   get_user_locale - 返回Locale类用于当前用户。
*   set_default_headers - 用于设置额外响应头。

#### 3.3 错误处理
若 handlers 引发异常，Tornado 将调用 write_error 来生成错误页面。 tornado.web.HTTPError 用于产生特殊错误状态码，其他异常产生"500"状态码。

在 debug 模式下默认错误页面包括 statck trace 和一行错误描述。如需自定义错误页面，可以重写 RequestHandler.write_error 函数（通常在 BaseHandler 中定义，使得在所有 Handler 中可以共享）。函数可调用 write 和 render。若错误有异常产生，exc_info 元组作为 keyword 参数传入该函数（注意，异常不保证 sys.exc_info 保存当前异常，因此使用 traceback.format_exception 而不是 traceback.exc_info）。

也可通过调用 **set_status** 从regular handler method 生成错误页面，写响应并返回，而不是使用 write_error。

对于 404 errors， 使用 default_handler_class  Application settings。该 handler 应复写 prepare 使得在其他 HTTP 方法中可用。用上述方式产生错误页面，一种是引发 HTTPError(404)，复写 write_error 函数，另一种是调用 self.set_status(404)，在 prepare() 中直接返回响应。

#### 3.4 重定向
两种方式实现重定向请求： RequestHandler.redirect 或 RedirectHandler。

第一种：

在RequestHandler中使用self.redirect()重定向到其他地方。可选参数permanent默认为false，产生一个“ 302 Found ”HTTP响应码，这在成功的POST请求后进行重定向很有用。若permanent为true，则产生“ 301 Moved Permanently ”响应码。

第二种：

RedirectHandler方式是在Application routing表里直接配置重定向，如例：

<pre><code class="python">
    app = tornado.web.Application([
        url(r"/app", tornado.web.RedirectHandler,
            dict(url="http://itunes.apple.com/my-app-id")),
        ])
</code></pre>

它也支持政治表达式替换，如下例子将所有 /pictures/ 重定向到 /photos/ 中：

<pre><code class="python">
    app = tornado.web.Application([
        url(r"/photos/(.*)", MyPhotoHandler),
        url(r"/pictures/(.*)", tornado.web.RedirectHandler,
            dict(url=r"/photos/\1")),
        ])
</code></pre>

#### 3.5 异步handlers
默认 handler 里 get/post 等方法是同步的，当一个 handler 运行时其他请求都被阻塞，所以 long-running handler 应实现异步，以非阻塞方式调用他的缓慢操作。这个主题在 *Asynchronous and non-Blocking I/O* 中详细说明，本节主要说明RequestHandler子类中的特殊异步技术。

最简单方式是使用 **coroutine** 修饰符，以 yield 关键字实现非阻塞 I/O，直到 coroutine 返回才产生响应。

有时，coroutines 没有 **callback-oriented style** 方便，即使用 tornado.web.asynchronous 修饰符。 使用它时响应不会自动发送， request 会保持打开知道回调函数调用 RequestHandler.finish。必须使用该函数，否则浏览器会挂起。

如下是一个异步例子：

<pre><code class="python">
    class MainHandler(tornado.web.RequestHandler):
        @tornado.web.asynchronous
        def get(self):
            http = tornado.httpclient.AsyncHTTPClient()
            http.fetch("http://friendfeed-api.com/v2/feed/bret",
                       callback=self.on_response)
    
        def on_response(self, response):
            if response.error: raise tornado.web.HTTPError(500)
            json = tornado.escape.json_decode(response.body)
            self.write("Fetched " + str(len(json["entries"])) + " entries "
                       "from the FriendFeed API")
            self.finish()
</code></pre>

当 get 返回时， request 还未结束，http client 调用 on_response 函数，request 仍然打开， 调用 self.finish() 后 response 被 flush 到浏览器。

为了对比，如下是协程例子：

<pre><code class="python">
    class MainHandler(tornado.web.RequestHandler):
        @tornado.gen.coroutine
        def get(self):
            http = tornado.httpclient.AsyncHTTPClient()
            response = yield http.fetch("http://friendfeed-api.com/v2/feed/bret")
            json = tornado.escape.json_decode(response.body)
            self.write("Fetched " + str(len(json["entries"])) + " entries "
                       "from the FriendFeed API")
</code></pre>        

更好的异步例子可参考 tornado demo 中的 chat 样例程序。它使用 long polling 实现了 ajax chat room 。使用 long polling 可能需要复写 on_connection_close() 函数来在连接关闭后进行清空操作。


[URLSpec]: (http://tornado.readthedocs.org/en/latest/web.html#tornado.web.URLSpec)
