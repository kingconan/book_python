Scrapy简介

---

好吧，这种简介[官方](https://doc.scrapy.org/en/latest/topics/architecture.html)最简单明了。

如果我来说，`scarpy`提供了一个基本的框架，让爬虫使用者只关注核心的业务逻辑，包括请求的收集，数据的解析。然后也提供了必要的`middleware`或者`callback`机制，允许我们干扰或自定义爬虫的基本处理方式。

至于请求方面的机制，我们不必太关心，当然，有心也可以学习下~反正我暂时木有这个想法，啊哈哈哈

开始之前，根据官方指导，安装`scrapy`，本人的开发环境是`Mac+python2.7+pycharm`。同时存储我这里使用的是`mongodb`，所以安装`mogondb`和`pymongo`。

# 

# 示例

---

通过`scarpy`建立**zhihu**爬虫项目，基本的目录结构大概如下

```
├── scrapy.cfg
└── zhihu
    ├── __init__.py
    ├── __init__.pyc
    ├── items.py
    ├── items.pyc
    ├── middlewares.py
    ├── middlewares.pyc
    ├── pipelines.py
    ├── pipelines.pyc
    ├── settings.py
    ├── settings.pyc
    └── spiders
        ├── UserSpider.py
        ├── UserSpider.pyc
        ├── __init__.py
        └── __init__.pyc
```

说明下各种文件的用途

| files | memo |
| :--- | :--- |
| settings.py | 爬虫设置，包括注册基本的中间件及配置变量定义 |
| spiders | 该目录下使我们爬虫文件夹，这里我实现了一个UserSpider用来爬知乎用户数据 |
| items.py | 定义我们自己的数据类，从网页从提取我们关心的数据，然后映射到这里 |
| pipelines.py | 管道类，用来处理item用的。当我们解析出item后，可以yeild处理，管道类里的各个管道处理类会根据配置的优先级分别解析item。在这里，我们可以做基本的数据处理，去重，验证及存储工作 |
| middlewares.py | 中间件类，通常用来自定义爬虫行为的。比如动态使用header或者代理来防止反爬虫 |

我们来看看`UserSpider.py`里是怎样的

```py
class UserSpider(scrapy.Spider):
    name = "user" #爬虫名字, 命令调用时依据此, scrap crawl user

    def start_requests(self): #爬虫启动时会调用该函数一次，必须返回可迭代的数据
        yield scrapy.Request(url=self.login_url, headers=self.headers, callback=self.login)

    def login(self, response): #request的回调函数，会将请求结果返回
        pass
```

由于爬知乎需要登录验证，所以我们的第一个请求通常是登录，并将登录信息存下来，已给后续爬虫请求使用。

知乎登录的请求主要分为三步

1. 获得xsrf
2. 获得验证码
3. 发post登录请求

获得知乎用户信息，我们这里偷懒的是，我们分析出了知乎用户信息的相关请求后获得了可返还json数据的url

```py
user_url = "https://www.zhihu.com/api/v4/members/{user}?include={include}"
follow_url = "https://www.zhihu.com/api/v4/members/{user}/followees?include=" \
                 "{include}&amp;offset={offset}&amp;limit={limit}"
```

`items.py` 是一个简单数据类，并木有什么说，记住要继承 `scrapy.item` 即可

```py
import scrapy
class UserItem(scrapy.Item):
    # define the fields for your item here like:
    name = scrapy.Field()
    url = scrapy.Field()
    json = scrapy.Field()
    url_token = scrapy.Field()
    seed_token = scrapy.Field()
    pass
```

`pipelines.py` 里我们定义解析item的管道，注意需要再setting中注册的

```py
import pymongo as mongo
from scrapy.exceptions import DropItem

#去重pipeline，如果发现该用户已经处理过了，则丢弃该item，否则记录并传递下去
class DuplicatePipeline(object):
    def __init__(self):
        self.ids_seen = set()

    def process_item(self, item, spider):#pipeline我们关注的函数，会被框架调用处理item
        if item["url_token"] in self.ids_seen:
            raise DropItem("duplicate item found")
        else:
            self.ids_seen.add(item["url_token"])
            return item

#数据存储，注意数据库初始化我们调用的是from_crawler，open_spider这些爬虫回调来进行数据库相关初始化
class MongoPipeline(object):
    def __init__(self, mongo_uri, mongo_port, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db
        self.mongo_port = mongo_port

    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI', "127.0.0.1"),
            mongo_port=crawler.settings.get('MONGO_PORT', 27017),
            mongo_db=crawler.settings.get('MONGO_DATABASE', 'test_db')
        )

    def open_spider(self,spider):
        self.client = mongo.MongoClient(self.mongo_uri, self.mongo_port)
        self.db = self.client[self.mongo_db]

    def close_spider(self,spider):
        self.client.close()

    def process_item(self, item, spider):
        collection_name = item.__class__.__name__
        self.db[collection_name].update({"url_token": item["url_token"]},
                                        item["json"], upsert=True)
        return item
```

`middlewares.py` 中间件，注意需要settiing注册的，根据功能，我们要注册的是Downloader Middleware。

```py
import random
import base64
from settings import PROXIES

#随机伪造一个User-Agent
class RandomUserAgentMiddleware(object):
    def __init__(self, agents):
        self.agents = agents

    @classmethod
    def from_crawler(cls, crawler):
        return cls(crawler.settings.getlist("USER_AGENTS"))

    def process_request(self, request, spider):
        request.headers["User-Agent"] = random.choice(self.agents)

#随机设置代理
class ProxyMiddleware(object):
    def process_request(self, request, spider):
        proxy = random.choice(PROXIES)
        if proxy["user_pass"] is not None:
            request.meta["proxy"] = "https://%s" % proxy["ip_port"]
            encoded_user_pass = base64.encodestring(proxy["user_pass"])
            request.headers["Proxy-Authorization"] = "Basic " + encoded_user_pass
        else:
            request.meta["proxy"] = "https://%s" % proxy["ip_port"]
```

`settings.py` 配置文件

```py
DOWNLOAD_DELAY = 3 #请求延时，单位是秒，默认是开启了随机延时的，范围大概 0.5 x 3 ~ 1.5 x 3吧
ITEM_PIPELINES = {#设置管道及优先级(1-1000)
   'zhihu.pipelines.DuplicatePipeline': 350,
   'zhihu.pipelines.MongoPipeline': 400,
}
USER_AGENTS = [#定义header头信息，用来伪造
    "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; AcooBrowser; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
    "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0; Acoo Browser; SLCC1; .NET CLR 2.0.50727; Media Center PC 5.0; .NET CLR 3.0.04506)",
    "Mozilla/4.0 (compatible; MSIE 7.0; AOL 9.5; AOLBuild 4337.35; Windows NT 5.1; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
]
PROXIES = [#代理IP定义
    {'ip_port': '1.28.182.175:80', 'user_pass': ''},
    {'ip_port': '106.123.54.220:80', 'user_pass': ''},
]
```

# QA

---

**reuquest 和 response 如何传递参数**

> request里支持meta字段，可以传dict过去，然后response.meta\["key"\]来访问

**登录的cookie如何存储**

> 使用CookieJar来搞，而且一旦用户主动干扰cookie，那么cookie的使用必须显式指定了。。。

```py
    def post_login(self, response):#获取cookie信息，注意，meta的cookiejar字段要每个request都传递的
        #....
        cookie_jar = response.meta['cookiejar']
        cookie_jar.extract_cookies(response, response.request)
        self.write_cookie(cookie_jar)
        #...

    def send_request(self):
        #...
        self.cookie_jar = self.read_cookie()
        yield scrapy.Request(url=self.follow_url,
                             headers=self.headers,
                             callback=self.parse_follow,
                             cookies=self.cookie_jar,
                             meta={"seed_token": self.user_start})
        #...

    def write_cookie(self, cookie_jar):
        with open("cookie_zhihu.cookie", "wb+") as f:
            for cookie in cookie_jar:
                f.write(str(cookie) + '\n')

    def read_cookie(self):
        with open("cookie_zhihu.cookie") as f:
            cookiejar = f.read()

        p = re.compile('\<Cookie (.*?) for .zhihu.com\/\>')
        cookies = re.findall(p, cookiejar)
        cookie_jar = {}
        for cookie in cookies:
            pos = cookie.find("=") #由于zhihu.com的cookie值里有=，所以只能通过查找来取key，value
            k = cookie[0: pos]
            v = cookie[pos+1:]
            cookie_jar[k] = v

        return dict(cookie_jar)
```

**数据库帮助函数如何在spider内部调用**

> 目前是在spider定义一个ref，然后在pipeline的open\_spider里 `spider.ref = self`

# 扩展

---

制作知乎feed源，

```py
header = {'authorization': 'Bearer Mi4wQUhEQTRRRHhSUW9BQUFEelpVamlDUmNBQUFCaEFsVk5ZZFJZV1'
                         'FCTzlEdjJ4amVnYjJtY3BxV1N0ZW01MTNBc1lB|1496817567|14dd15b6c0'
                         '3e3f39322f6f09b7a1a2b9876d8eed'}
feed_url = "https://www.zhihu.com/api/v4/members/{user}/activities?" \
               "limit=10"
```

feed结构整理

| verb |  |  |
| :--- | :--- | :--- |
| ANSWER\_CREATE | 回答问题 |  |
| ANSWER\_VOTE\_UP | 赞同问题 |  |
| MEMBER\_VOTEUP\_ARTICLE | 赞文章 |  |
| MEMBER\_CREATE\_ARTICLE | 发表文章 |  |















