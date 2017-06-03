# Scrapy简介

好吧，这种简介[官方](https://doc.scrapy.org/en/latest/topics/architecture.html)最简单明了。

如果我来说，scarpy提供了一个基本的框架，让爬虫使用者只关注核心的业务逻辑，包括请求的收集，数据的解析。然后也提供了必要的middleware或者callback机制，允许我们干扰或自定义爬虫的基本处理方式。

至于请求方面的机制，我们不必太关心，当然，有心也可以学习下~反正我暂时木有这个想法，啊哈哈哈

开始之前，根据官方指导，安装scrapy，本人的开发环境是Mac+python2.7+pycharm。同时存储我这里使用的是mongodb，所以安装mogondb和pymongo。

# 示例

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

```
class UserSpider(scrapy.Spider):
    name = "user"
    
    def start_requests(self):
        yield scrapy.Request(url=self.login_url, headers=self.headers, callback=self.login)
        
    def login(self, response0:
        pass
```

# QA

# 扩展



