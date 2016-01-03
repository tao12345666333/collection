译者说
-----
`Tornado 4.3`于2015年11月6日发布，该版本正式支持`Python3.5`的`async`/`await`关键字，并且用旧版本CPython编译Tornado同样可以使用这两个关键字，这无疑是一种进步。其次，这是最后一个支持`Python2.6`和`Python3.2`的版本了，在后续的版本了会移除对它们的兼容。现在网络上还没有`Tornado4.3`的中文文档，所以为了让更多的朋友能接触并学习到它，我开始了这个翻译项目，希望感兴趣的小伙伴可以一起参与翻译，项目地址是[tornado-zh on Github](https://github.com/tao12345666333/tornado-zh)，翻译好的文档在[Read the Docs上](https://tornado-zh.readthedocs.org/)直接可以看到。欢迎Issues or PR。

# 示例 - 一个并发网络爬虫

Tornado的 `tornado.queues` 模块实现了异步生产者/消费者模式的协程, 类似于通过Python 标准库的 `queue`实现线程模式.

一个yield `Queue.get` 的协程直到队列中有值的时候才会暂停. 如果队列设置了最大长度yield `Queue.put` 的协程直到队列中有空间才会暂停.

一个`Queue`从0开始对完成的任务进行计数. `Queue.put` 加计数;`Queue.task_done`减少计数.

这里的网络爬虫的例子, 队列开始的时候只包含base\_url. 当一个worker抓取到一个页面它会解析链接并把它添加到队列中, 然后调用`Queue.task_done` 减少计数一次. 最后, 当一个worker抓取到的页面URL都是之前抓取到过的并且队列中没有任务了.于是worker调用 `Queue.task_done` 把计数减到0. 等待 `Queue.join` 的主协程取消暂停并且完成.

```python

    import time
    from datetime import timedelta
    
    try:
        from HTMLParser import HTMLParser
        from urlparse import urljoin, urldefrag
    except ImportError:
        from html.parser import HTMLParser
        from urllib.parse import urljoin, urldefrag
    
    from tornado import httpclient, gen, ioloop, queues
    
    base_url = 'http://www.tornadoweb.org/en/stable/'
    concurrency = 10
    
    
    @gen.coroutine
    def get_links_from_url(url):
        """Download the page at `url` and parse it for links.
    
        Returned links have had the fragment after `#` removed, and have been made
        absolute so, e.g. the URL 'gen.html#tornado.gen.coroutine' becomes
        'http://www.tornadoweb.org/en/stable/gen.html'.
        """
        try:
            response = yield httpclient.AsyncHTTPClient().fetch(url)
            print('fetched %s' % url)
    
            html = response.body if isinstance(response.body, str) \
                else response.body.decode()
            urls = [urljoin(url, remove_fragment(new_url))
                    for new_url in get_links(html)]
        except Exception as e:
            print('Exception: %s %s' % (e, url))
            raise gen.Return([])
    
        raise gen.Return(urls)
    
    
    def remove_fragment(url):
        pure_url, frag = urldefrag(url)
        return pure_url
    
    
    def get_links(html):
        class URLSeeker(HTMLParser):
            def __init__(self):
                HTMLParser.__init__(self)
                self.urls = []
    
            def handle_starttag(self, tag, attrs):
                href = dict(attrs).get('href')
                if href and tag == 'a':
                    self.urls.append(href)
    
        url_seeker = URLSeeker()
        url_seeker.feed(html)
        return url_seeker.urls
    
    
    @gen.coroutine
    def main():
        q = queues.Queue()
        start = time.time()
        fetching, fetched = set(), set()
    
        @gen.coroutine
        def fetch_url():
            current_url = yield q.get()
            try:
                if current_url in fetching:
                    return
    
                print('fetching %s' % current_url)
                fetching.add(current_url)
                urls = yield get_links_from_url(current_url)
                fetched.add(current_url)
    
                for new_url in urls:
                    # Only follow links beneath the base URL
                    if new_url.startswith(base_url):
                        yield q.put(new_url)
    
            finally:
                q.task_done()
    
        @gen.coroutine
        def worker():
            while True:
                yield fetch_url()
    
        q.put(base_url)
    
        # Start workers, then wait for the work queue to be empty.
        for _ in range(concurrency):
            worker()
        yield q.join(timeout=timedelta(seconds=300))
        assert fetching == fetched
        print('Done in %d seconds, fetched %s URLs.' % (
            time.time() - start, len(fetched)))
    
    
    if __name__ == '__main__':
        import logging
        logging.basicConfig()
        io_loop = ioloop.IOLoop.current()
        io_loop.run_sync(main)
    
```
