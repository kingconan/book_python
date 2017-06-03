# Scrapy简介

---

好吧，这种简介[官方](https://doc.scrapy.org/en/latest/topics/architecture.html)最简单明了。

如果我来说，scarpy提供了一个基本的框架，让爬虫使用者只关注核心的业务逻辑，包括请求的收集，数据的解析。然后也提供了必要的middleware或者callback机制，允许我们干扰或自定义爬虫的基本处理方式。

至于请求方面的机制，我们不必太关心，当然，有心也可以学习下~反正我暂时木有这个想法，啊哈哈哈

开始之前，根据官方指导，安装scrapy，本人的开发环境是Mac+python2.7+pycharm。同时存储我这里使用的是mongodb，所以安装mogondb和pymongo。

# 

# 示例

---

通过scarpy建立zhihu爬虫项目，基本的目录结构大概如下

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
    "Mozilla/5.0 (Windows; U; MSIE 9.0; Windows NT 9.0; en-US)",
    "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 2.0.50727; Media Center PC 6.0)",
    "Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 1.0.3705; .NET CLR 1.1.4322)",
    "Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.2; .NET CLR 1.1.4322; .NET CLR 2.0.50727; InfoPath.2; .NET CLR 3.0.04506.30)",
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN) AppleWebKit/523.15 (KHTML, like Gecko, Safari/419.3) Arora/0.3 (Change: 287 c9dfb30)",
    "Mozilla/5.0 (X11; U; Linux; en-US) AppleWebKit/527+ (KHTML, like Gecko, Safari/419.3) Arora/0.6",
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.2pre) Gecko/20070215 K-Ninja/2.1.1",
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9) Gecko/20080705 Firefox/3.0 Kapiko/3.0",
    "Mozilla/5.0 (X11; Linux i686; U;) Gecko/20070322 Kazehakase/0.4.5",
    "Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.0.8) Gecko Fedora/1.9.0.8-1.fc10 Kazehakase/0.5.6",
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.56 Safari/535.11",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_3) AppleWebKit/535.20 (KHTML, like Gecko) Chrome/19.0.1036.7 Safari/535.20",
    "Opera/9.80 (Macintosh; Intel Mac OS X 10.6.8; U; fr) Presto/2.9.168 Version/11.52",
]
PROXIES = [#代理IP定义
    {'ip_port': '1.28.182.175:80', 'user_pass': ''},
    {'ip_port': '106.123.54.220:80', 'user_pass': ''},
]
```

# 

# QA

---

1. `yeild` 是什么鬼
2. `@classmethod` 是什么鬼

# 

# 扩展

---



