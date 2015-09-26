---
layout	: post
title	: Tornado Coroutine（协程）
date	: 2015-05-15
meta 	: Tornado
tags	: [Python]
category: "Web development"
digest	: "Tornado 推荐使用 coroutine（协程） 方式实现异步。 使用 yield 关键字挂起和恢复，而不是用回调链。（gevent 等轻量级线程有时也被看做协程，但在 Tornado 中， 所有协程使用精确上下文切换，也被看做异步函数）。<br/>
协程与同步函数差不多简单，却没有线程开销。 使用协程使得并发更容易响应，因为减少了上下文切换资源开销。"

---

Tornado 推荐使用 **coroutine（协程）** 方式实现异步。 使用 **yield** 关键字挂起和恢复，而不是用回调链。（gevent 等轻量级线程有时也被看做协程，但在 Tornado 中， 所有协程使用精确上下文切换，也被看做异步函数）。

协程与同步函数差不多简单，却没有线程开销。 使用协程使得并发更容易响应，因为减少了上下文切换资源开销。

示例代码：

<pre><code class="python">
    from tornado import gen

    @gen.coroutine
    def fetch_coroutine(url):
        http_client = AsyncHTTPClient()
        response = yield http_client.fetch(url)
        # In Python versions prior to 3.3, returning a value from
        # a generator is not allowed and you must use
        #   raise gen.Return(response.body)
        # instead.
        return response.body
</code></pre>

### 1 工作机制
python 的函数中若包含 *yield* 则称之为生成器。所有生成器都是异步的，当被调用时他们返回生成器当前迭代值（注：生成器也是迭代器）而不是运行到结束。**@gen.coroutine** 装饰器与生成器通过 yield 交互，与协程调用者通过 **Future** 来交互。

这是协程装饰器内部循环简单例子：

<pre><code class="python">
    # Simplified inner loop of tornado.gen.Runner
    def run(self):
        # send(x) makes the current yield return x.
        # It returns when the next yield is reached
        future = self.gen.send(self.next)
        def callback(f):
            self.next = f.result()
            self.run()
        future.add_done_callback(callback)
</code></pre>        

装饰器从生成器接收 *Future*, 等待（非阻塞） Future 完成， 然后打开 Future， 将结果送回生成器作为 yield 表达式的结果，绝大多数异步代码不直接使用 Future 类，除了某些异步函数立即传递 Future 给 yield 表达式。


### 2 协程模式

#### 2.1 使用回调函数交互
对于使用回调函数而不是 Future 的异步代码进行交互，将函数打包入 Task。这将增加一个 callback argument 并且返回 Future 供你生成：
    
<pre><code class="python">    
    @gen.coroutine
    def call_task():
        # Note that there are no parens on some_function.
        # This will be translated by Task into
        #   some_function(other_args, callback=callback)
        yield gen.Task(some_function, other_args)
</code></pre>

#### 2.2 调用阻塞函数
协程中调用阻塞函数最简单方法是使用 ThreadPoolExecutor， 返回与协程兼容的 Future：

<pre><code class="python">
    thread_pool = ThreadPoolExecutor(4)
    
    @gen.coroutine
    def call_blocking():
        yield thread_pool.submit(blocking_func, args)
</code></pre>

#### 2.3 并行
协程装饰器识别出 Future 列表或字典，等待这些 Futures并行：

<pre><code class="python">
    @gen.coroutine
    def parallel_fetch(url1, url2):
        resp1, resp2 = yield [http_client.fetch(url1),
                              http_client.fetch(url2)]
    
    @gen.coroutine
    def parallel_fetch_many(urls):
        responses = yield [http_client.fetch(url) for url in urls]
        # responses is a list of HTTPResponses in the same order
    
    @gen.coroutine
    def parallel_fetch_dict(urls):
        responses = yield {url: http_client.fetch(url)
                            for url in urls}
        # responses is a dict {url: HTTPResponse}
</code></pre>

#### 2.4 交替
有时保存 Future 而不是立即生成它更有用，这样你能够在等待前开始其他操作：

<pre><code class="python">
    @gen.coroutine
    def get(self):
        fetch_future = self.fetch_next_chunk()
        while True:
            chunk = yield fetch_future
            if chunk is None: break
            self.write(chunk)
            fetch_future = self.fetch_next_chunk()
            yield self.flush()        
</code></pre>

#### 2.5 Looping
Looping对协程很具有欺骗性，因为在python中无法在for或while的每个迭代中进行yield并且捕获 yield 的结果。替换的方法是你你把循环条件和处理结果过程分开（yield和处理分开），如Motor例子一样：

<pre><code class="python">
    import motor
    db = motor.MotorClient().test
    
    @gen.coroutine
    def loop_example(collection):
        cursor = db.collection.find()
        while (yield cursor.fetch_next):
            doc = cursor.next_object() 
</code></pre>            
