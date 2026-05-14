---
tags:
  - Python
  - 网络请求与数据采集
  - Scrapy
  - 爬虫框架
date: 2026-05-14
status: 已完成
difficulty: 中高
---

# Scrapy框架

## What — 是什么

> Scrapy 是 Python 最强大的爬虫框架，提供从请求调度、页面解析、数据存储到分布式爬取的全流程解决方案，适用于大规模、结构化的数据采集项目。

**核心概念：**

- **Spider**：爬虫核心类，定义起始 URL、解析规则、跟进策略，是整个爬虫的业务逻辑载体
- **Item**：数据容器，类似 ORM 的 Model，定义采集数据的字段和结构
- **Pipeline**：数据处理流水线，对 Item 进行清洗、验证、存储（数据库、文件、API）
- **Middleware**：中间件，拦截请求（Downloader Middleware）和响应（Spider Middleware），实现代理、User-Agent 轮换、重试、Cookie 处理等
- **Selector**：基于 lxml 的选择器，支持 XPath 和 CSS 选择器，是 Scrapy 的数据提取引擎
- **Scheduler**：请求调度器，管理待爬 URL 队列，去重、优先级排序

**核心架构：**

```
Spider → Requests → Scheduler → Downloader → Response → Spider → Items → Pipelines
   ↑                                                    |
   └──────── 新 Requests（跟进链接）←────────────────────┘
```

- 数据流：Spider 生成初始 Requests → Scheduler 排队去重 → Downloader 下载 → Response 回传 Spider → Spider 解析产生 Items + 新 Requests → Items 进入 Pipelines 处理存储
- 并发模型：基于 Twisted 异步框架，单线程事件驱动，默认并发 16 个请求
- 去重机制：默认基于 URL 的 SHA1 哈希，可自定义去重规则

**关键特性：**

- 自动限速（AutoThrottle）：根据服务器响应时间自动调整爬取速率
- 内置中间件：重试、重定向、Cookie、压缩、缓存、HTTP 认证
- 信号系统：爬虫启动/关闭、请求丢弃、响应接收等事件钩子
- 扩展机制：自定义扩展（Extensions），如统计、日志、邮件通知
- 分布式：配合 scrapy-redis 实现分布式爬虫

## Why — 为什么

**适用场景：**

- 大规模数据采集——电商全站商品、新闻全量归档、招聘数据采集
- 定期增量爬取——每天采集新数据、监控价格变化
- 深度爬取——多层级页面跟进（列表→详情→评论）
- 数据管道处理——采集→清洗→验证→存储的完整流水线
- 分布式爬虫——多机协作爬取海量数据

**对比同类方案：**

| 维度 | Scrapy | requests+BS4 | Crawlee (Python) | PySpider |
|------|--------|-------------|-------------------|----------|
| 定位 | 全功能爬虫框架 | HTTP库+解析库 | 新一代爬虫框架 | 可视化爬虫 |
| 并发模型 | Twisted 异步 | 同步（需手动异步） | asyncio | Tornado 异步 |
| 调度去重 | 内置 | 需手动实现 | 内置 | 内置 |
| 数据管道 | Pipeline 完善 | 需手动实现 | 内置 | 内置 |
| 中间件 | 丰富（代理/UA/重试） | 需手动实现 | 内置 | 基础 |
| 分布式 | scrapy-redis | 需手动实现 | 原生支持 | 需手动 |
| 可视化 | 无 | 无 | 无 | Web UI |
| 学习曲线 | 中高 | 低 | 中 | 中 |
| 生态成熟度 | 最成熟 | — | 新项目 | 维护少 |

**优缺点：**

- ✅ 优点：
  - 全流程覆盖，开箱即用
  - Twisted 异步，高并发性能好
  - 中间件和 Pipeline 机制灵活
  - 自动去重、调度、限速
  - 生态成熟，社区活跃
- ❌ 缺点：
  - 学习曲线较陡（Twisted、信号、中间件）
  - 调试不如脚本式爬虫直观
  - 不支持 JavaScript 渲染（需搭配 Splash/Playwright）
  - Twisted 与 asyncio 不兼容

## How — 怎么用

### 1. 项目创建与结构

```bash
pip install scrapy
scrapy startproject myspider
cd myspider
```

```
myspider/
├── scrapy.cfg              # 项目配置（部署用）
└── myspider/               # Python 模块
    ├── __init__.py
    ├── items.py            # Item 定义
    ├── middlewares.py      # 中间件
    ├── pipelines.py        # 数据管道
    ├── settings.py         # 爬虫设置
    └── spiders/            # 爬虫目录
        ├── __init__.py
        └── example.py      # 爬虫文件
```

```bash
# 创建爬虫
scrapy genspider example example.com

# 运行爬虫
scrapy crawl example

# 调试（交互式 shell）
scrapy shell 'https://quotes.toscrape.com/'

# 导出数据
scrapy crawl example -o items.json
scrapy crawl example -o items.csv
scrapy crawl example -o items.jl   # JSON Lines（每行一个 JSON）
```

### 2. Spider 编写

```python
# myspider/spiders/quotes.py
import scrapy
from myspider.items import QuoteItem


class QuotesSpider(scrapy.Spider):
    """基础爬虫示例：爬取 quotes.toscrape.com"""

    name = 'quotes'              # 爬虫唯一名称
    allowed_domains = ['quotes.toscrape.com']  # 域名限制
    start_urls = [
        'https://quotes.toscrape.com/page/1/',
    ]

    # 方式一：自动请求 start_urls，用 parse 解析
    def parse(self, response):
        # 提取名言
        for quote in response.css('div.quote'):
            item = QuoteItem()
            item['text'] = quote.css('span.text::text').get()
            item['author'] = quote.css('small.author::text').get()
            item['tags'] = quote.css('div.tags a.tag::text').getall()
            yield item

        # 跟进下一页
        next_page = response.css('li.next a::attr(href)').get()
        if next_page:
            yield response.follow(next_page, callback=self.parse)

    # 方式二：自定义 start_requests（更灵活）
    # def start_requests(self):
    #     urls = [f'https://quotes.toscrape.com/page/{i}/' for i in range(1, 11)]
    #     for url in urls:
    #         yield scrapy.Request(url, callback=self.parse)
```

**Spider 类型：**

| 类型 | 类 | 说明 |
|------|-----|------|
| 基础 | `scrapy.Spider` | 最通用，手动定义请求和解析 |
| CrawlSpider | `CrawlSpider` | 自动跟进符合规则的链接 |
| XMLFeedSpider | `XMLFeedSpider` | 解析 XML/RSS |
| CSVFeedSpider | `CSVFeedSpider` | 解析 CSV |
| SitemapSpider | `SitemapSpider` | 基于 sitemap.xml 爬取 |

**CrawlSpider（自动跟进链接）：**

```python
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule
from myspider.items import ArticleItem


class NewsSpider(CrawlSpider):
    """自动跟进新闻网站的列表页和详情页"""
    name = 'news'
    allowed_domains = ['news.example.com']
    start_urls = ['https://news.example.com/']

    # 定义链接提取规则
    rules = (
        # 列表页：跟进分页链接
        Rule(
            LinkExtractor(allow=r'/page/\d+'),
            follow=True,  # 继续跟进，不解析
        ),
        # 详情页：提取并解析
        Rule(
            LinkExtractor(allow=r'/article/\d+'),
            callback='parse_article',  # 调用解析方法
            follow=False,
        ),
    )

    def parse_article(self, response):
        """解析文章详情页"""
        item = ArticleItem()
        item['title'] = response.css('h1::text').get()
        item['author'] = response.css('.author::text').get()
        item['content'] = response.css('.article-body').get()
        item['url'] = response.url
        item['published_at'] = response.css('time::attr(datetime)').get()
        yield item
```

### 3. Selector 选择器

Scrapy 的 Selector 基于 lxml，支持 XPath 和 CSS 两种语法。

```python
# 在 scrapy shell 中调试
# scrapy shell 'https://quotes.toscrape.com/'

# CSS 选择器
response.css('title::text').get()                    # 'Quotes to Scrape'
response.css('div.quote').getall()                    # 所有 quote 元素
response.css('span.text::text').get()                 # 第一个文本
response.css('span.text::text').getall()              # 所有文本
response.css('a::attr(href)').get()                   # href 属性
response.css('div.quote:first-child').get()           # CSS 伪选择器

# XPath
response.xpath('//title/text()').get()
response.xpath('//div[@class="quote"]').getall()
response.xpath('//span[@class="text"]/text()').get()
response.xpath('//a/@href').get()
response.xpath('//div[contains(@class, "quote")][1]').get()

# 链式调用
quote = response.css('div.quote')[0]
quote.css('span.text::text').get()
quote.xpath('.//small[@class="author"]/text()').get()

# 正则匹配
response.css('span.text::text').re(r'“(.+)”')        # 正则提取
response.css('span.text::text').re_first(r'“(.+)”')   # 第一个匹配

# 常用选择器速查
# .css('tag::text')       标签内文本
# .css('tag::attr(name)') 标签属性
# .css('.class')          类选择器
# .css('#id')             ID 选择器
# .xpath('//tag')         全局搜索
# .xpath('./tag')         当前节点下搜索
# .xpath('.//tag')        当前节点下递归搜索
```

### 4. Item 与 Pipeline

```python
# myspider/items.py
import scrapy
from itemloaders.processors import TakeFirst, MapCompose, Join


def strip_text(value):
    """清洗文本：去除首尾空白"""
    return value.strip() if value else ''


def remove_currency(value):
    """清洗价格：移除货币符号"""
    return value.replace('¥', '').replace('$', '').strip()


class QuoteItem(scrapy.Item):
    """名言数据结构"""
    text = scrapy.Field(
        input_processor=MapCompose(strip_text),
        output_processor=TakeFirst(),
    )
    author = scrapy.Field(
        input_processor=MapCompose(strip_text),
        output_processor=TakeFirst(),
    )
    tags = scrapy.Field()  # 列表，保留全部
    url = scrapy.Field(output_processor=TakeFirst())
    crawled_at = scrapy.Field(output_processor=TakeFirst())


class ProductItem(scrapy.Item):
    """商品数据结构"""
    name = scrapy.Field(
        input_processor=MapCompose(strip_text),
        output_processor=TakeFirst(),
    )
    price = scrapy.Field(
        input_processor=MapCompose(strip_text, remove_currency, float),
        output_processor=TakeFirst(),
    )
    description = scrapy.Field(
        input_processor=MapCompose(strip_text),
        output_processor=Join(' '),
    )
    images = scrapy.Field()
    url = scrapy.Field(output_processor=TakeFirst())
```

```python
# myspider/pipelines.py
import json
import sqlite3
import pymongo
from itemadapter import ItemAdapter
from datetime import datetime


class JsonPipeline:
    """保存为 JSON Lines"""

    def open_spider(self, spider):
        self.file = open('items.jl', 'w', encoding='utf-8')

    def close_spider(self, spider):
        self.file.close()

    def process_item(self, item, spider):
        line = json.dumps(dict(item), ensure_ascii=False) + '\n'
        self.file.write(line)
        return item


class SQLitePipeline:
    """保存到 SQLite"""

    def open_spider(self, spider):
        self.conn = sqlite3.connect('items.db')
        self.cursor = self.conn.cursor()
        self.cursor.execute('''
            CREATE TABLE IF NOT EXISTS quotes (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                text TEXT,
                author TEXT,
                tags TEXT,
                url TEXT,
                crawled_at TEXT
            )
        ''')
        self.conn.commit()

    def close_spider(self, spider):
        self.conn.close()

    def process_item(self, item, spider):
        self.cursor.execute(
            'INSERT INTO quotes (text, author, tags, url, crawled_at) VALUES (?, ?, ?, ?, ?)',
            (item.get('text'), item.get('author'),
             json.dumps(item.get('tags', [])),
             item.get('url'), item.get('crawled_at'))
        )
        self.conn.commit()
        return item


class MongoDBPipeline:
    """保存到 MongoDB"""

    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db

    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI', 'mongodb://localhost:27017'),
            mongo_db=crawler.settings.get('MONGO_DATABASE', 'scrapy'),
        )

    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]

    def close_spider(self, spider):
        self.client.close()

    def process_item(self, item, spider):
        collection = self.db[item.__class__.__name__.lower()]
        collection.insert_one(dict(item))
        return item


class DuplicatesPipeline:
    """去重管道：基于 URL 去重"""

    def __init__(self):
        self.urls_seen = set()

    def process_item(self, item, spider):
        url = item.get('url')
        if url in self.urls_seen:
            spider.logger.info(f'重复项已丢弃: {url}')
            raise DropItem(f'重复项: {url}')
        self.urls_seen.add(url)
        return item


class ValidationPipeline:
    """数据验证管道"""

    def process_item(self, item, spider):
        if not item.get('text'):
            raise DropItem(f'缺少 text 字段: {item}')
        if not item.get('author'):
            raise DropItem(f'缺少 author 字段: {item}')
        return item


class AddTimestampPipeline:
    """添加时间戳管道"""

    def process_item(self, item, spider):
        item['crawled_at'] = datetime.now().isoformat()
        return item
```

```python
# myspider/settings.py — 启用 Pipeline
ITEM_PIPELINES = {
    'myspider.pipelines.ValidationPipeline': 100,    # 数字越小越先执行
    'myspider.pipelines.DuplicatesPipeline': 200,
    'myspider.pipelines.AddTimestampPipeline': 300,
    'myspider.pipelines.SQLitePipeline': 400,
    # 'myspider.pipelines.MongoDBPipeline': 500,
}
```

### 5. 中间件

```python
# myspider/middlewares.py
import random
import time
from scrapy import signals
from scrapy.downloadermiddlewares.retry import RetryMiddleware
from scrapy.utils.response import response_status_message


class RotateUserAgentMiddleware:
    """随机 User-Agent 中间件"""

    def __init__(self):
        self.user_agents = [
            'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/120.0.0.0',
            'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 Chrome/120.0.0.0',
            'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:121.0) Gecko/20100101 Firefox/121.0',
            'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 Safari/17.2',
            'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 Chrome/120.0.0.0',
        ]

    def process_request(self, request, spider):
        request.headers['User-Agent'] = random.choice(self.user_agents)


class RotateProxyMiddleware:
    """代理轮换中间件"""

    def __init__(self, proxy_list):
        self.proxy_list = proxy_list
        self.current = 0

    @classmethod
    def from_crawler(cls, crawler):
        proxy_file = crawler.settings.get('PROXY_LIST_FILE', 'proxies.txt')
        with open(proxy_file) as f:
            proxies = [line.strip() for line in f if line.strip()]
        return cls(proxies)

    def process_request(self, request, spider):
        if self.proxy_list:
            proxy = self.proxy_list[self.current % len(self.proxy_list)]
            request.meta['proxy'] = proxy
            self.current += 1

    def process_exception(self, request, exception, spider):
        """代理失败时移除"""
        proxy = request.meta.get('proxy')
        if proxy and proxy in self.proxy_list:
            self.proxy_list.remove(proxy)
            spider.logger.info(f'移除失效代理: {proxy}')


class TooManyRequestsMiddleware(RetryMiddleware):
    """处理 429 Too Many Requests"""

    def __init__(self, settings):
        super().__init__(settings)
        self.max_retry_times = settings.getint('RETRY_TIMES', 3)

    def process_response(self, request, response, spider):
        if response.status == 429:
            retry_after = response.headers.get('Retry-After')
            wait = int(retry_after) if retry_after else random.randint(5, 30)
            spider.logger.info(f'429 限流，等待 {wait}s 后重试')
            time.sleep(wait)
            return self._retry(request, response_status_message(response.status), spider) or response

        return super().process_response(request, response, spider)
```

```python
# settings.py — 启用中间件
DOWNLOADER_MIDDLEWARES = {
    'myspider.middlewares.RotateUserAgentMiddleware': 400,
    'myspider.middlewares.RotateProxyMiddleware': 410,
    'myspider.middlewares.TooManyRequestsMiddleware': 550,
}
```

### 6. 设置与调优

```python
# settings.py — 常用配置

# 基本设置
BOT_NAME = 'myspider'
SPIDER_MODULES = ['myspider.spiders']
NEWSPIDER_MODULE = 'myspider.spiders'

# 遵守 robots.txt（生产环境建议 True）
ROBOTSTXT_OBEY = False

# 并发设置
CONCURRENT_REQUESTS = 16              # 全局最大并发请求数
CONCURRENT_REQUESTS_PER_DOMAIN = 8    # 单域名最大并发
CONCURRENT_REQUESTS_PER_IP = 0        # 单 IP 最大并发（0 表示不限）

# 下载延迟
DOWNLOAD_DELAY = 1                     # 请求间隔（秒）
RANDOMIZE_DOWNLOAD_DELAY = True        # 随机化延迟（0.5x ~ 1.5x）

# 超时
DOWNLOAD_TIMEOUT = 30                  # 下载超时（秒）

# 重试
RETRY_ENABLED = True
RETRY_TIMES = 3                        # 重试次数
RETRY_HTTP_CODES = [429, 500, 502, 503, 504]  # 触发重试的状态码

# 自动限速（推荐开启）
AUTOTHROTTLE_ENABLED = True
AUTOTHROTTLE_START_DELAY = 1           # 起始延迟
AUTOTHROTTLE_MAX_DELAY = 30            # 最大延迟
AUTOTHROTTLE_TARGET_CONCURRENCY = 2.0  # 目标并发

# 缓存（开发调试用）
HTTPCACHE_ENABLED = True
HTTPCACHE_EXPIRATION_SECS = 86400      # 缓存 24 小时
HTTPCACHE_DIR = 'httpcache'

# 日志
LOG_LEVEL = 'INFO'                     # DEBUG / INFO / WARNING / ERROR
LOG_FILE = 'scrapy.log'

# 默认请求头
DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'zh-CN,zh;q=0.9,en;q=0.8',
}

# 禁用 Cookie（某些网站通过 Cookie 追踪）
COOKIES_ENABLED = True

# 深度限制
DEPTH_LIMIT = 0                        # 0 表示无限
DEPTH_PRIORITY = 1                     # 1=BFS（广度优先），-1=DFS（深度优先）

# 数据导出编码
FEED_EXPORT_ENCODING = 'utf-8'
```

### 7. 分布式爬虫（scrapy-redis）

```bash
pip install scrapy-redis
```

```python
# settings.py — 分布式配置
SCHEDULER = 'scrapy_redis.scheduler.Scheduler'
DUPEFILTER_CLASS = 'scrapy_redis.dupefilter.RFPDupeFilter'
SCHEDULER_PERSIST = True               # 重启后保留队列

REDIS_URL = 'redis://localhost:6379/0'

# 允许暂停和恢复
SCHEDULER_QUEUE_KEY = '%(spider)s:requests'
SCHEDULER_DUPEFILTER_KEY = '%(spider)s:dupefilter'
ITEM_PIPELINES = {
    'scrapy_redis.pipelines.RedisPipeline': 300,
}
```

```python
# spiders/distributed_spider.py
from scrapy_redis.spiders import RedisSpider


class DistributedSpider(RedisSpider):
    """分布式爬虫：从 Redis 队列获取起始 URL"""
    name = 'distributed'

    # Redis key，lpush 起始 URL
    redis_key = 'myspider:start_urls'

    def parse(self, response):
        # 和普通 Spider 一样
        for item in response.css('div.item'):
            yield {
                'title': item.css('h2::text').get(),
                'url': response.url,
            }

        next_page = response.css('a.next::attr(href)').get()
        if next_page:
            yield response.follow(next_page, callback=self.parse)
```

```bash
# 在多台机器上启动
scrapy crawl distributed

# 推送起始 URL 到 Redis
redis-cli lpush myspider:start_urls 'https://example.com/page/1'
```

### 8. 实战：电商商品爬虫

```python
# myspider/spiders/shop.py
import scrapy
from myspider.items import ProductItem
from itemloaders.processors import MapCompose, TakeFirst


class ShopSpider(scrapy.Spider):
    name = 'shop'
    allowed_domains = ['shop.example.com']

    custom_settings = {
        'DOWNLOAD_DELAY': 2,
        'CONCURRENT_REQUESTS_PER_DOMAIN': 4,
        'AUTOTHROTTLE_ENABLED': True,
        'FEEDS': {
            'products_%(time)s.jl': {'format': 'jsonlines'},
        },
    }

    def start_requests(self):
        categories = ['electronics', 'clothing', 'books', 'home']
        for cat in categories:
            url = f'https://shop.example.com/category/{cat}'
            yield scrapy.Request(url, callback=self.parse_category,
                                 meta={'category': cat})

    def parse_category(self, response):
        category = response.meta['category']

        # 提取商品列表
        for product in response.css('div.product-item'):
            url = product.css('a.product-link::attr(href)').get()
            if url:
                yield scrapy.Request(
                    url,
                    callback=self.parse_product,
                    meta={'category': category},
                )

        # 跟进分页
        next_page = response.css('a.next-page::attr(href)').get()
        if next_page:
            yield response.follow(
                next_page,
                callback=self.parse_category,
                meta={'category': category},
            )

    def parse_product(self, response):
        category = response.meta['category']

        item = ProductItem()
        item['name'] = response.css('h1.product-title::text').get(default='').strip()
        item['price'] = response.css('span.price::text').re_first(r'[\d.]+')
        item['original_price'] = response.css('span.original-price::text').re_first(r'[\d.]+')
        item['description'] = response.css('div.product-description').get()
        item['images'] = response.css('div.product-gallery img::attr(src)').getall()
        item['category'] = category
        item['url'] = response.url
        item['sku'] = response.css('span.sku::text').get()
        item['stock'] = response.css('span.stock::text').get(default='0').strip()
        item['rating'] = response.css('span.rating::attr(data-score)').get()
        item['review_count'] = response.css('span.review-count::text').re_first(r'\d+')

        yield item
```

---

## 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 爬虫被 ban | 请求过快或 UA 单一 | 降低并发、增加延迟、轮换 UA 和代理 |
| 数据重复 | URL 去重失效或参数变化 | 自定义 dupefilter、规范化 URL 参数 |
| 内存持续增长 | 响应体缓存未清理 | 关闭 HTTPCACHE、限制 DEPTH_LIMIT |
| 403 Forbidden | 反爬策略检测 | 添加 Referer、Cookie、轮换代理 |
| Pipeline 报错 | Item 字段缺失或类型错误 | ValidationPipeline 校验、get(default='') |
| 调试困难 | Scrapy 日志信息过多 | `scrapy shell` 交互调试，设 LOG_LEVEL=DEBUG |
| JS 渲染内容缺失 | Scrapy 不执行 JS | 集成 Splash 或 Playwright 中间件 |
| 爬虫启动后无请求 | start_urls 格式错误或域名被 allowed_domains 过滤 | 检查 URL 格式和 allowed_domains |

---

## 面试题

**Q1: Scrapy 的架构和数据流是怎样的？**

> Scrapy 核心组件：Engine（引擎）、Scheduler（调度器）、Downloader（下载器）、Spider（爬虫）、Item Pipeline（数据管道）。数据流：(1) Engine 从 Spider 获取初始 Requests；(2) Engine 将 Requests 发给 Scheduler 排队去重；(3) Engine 从 Scheduler 获取下一个 Request；(4) Engine 将 Request 发给 Downloader 下载；(5) Downloader 完成下载，将 Response 发回 Engine；(6) Engine 将 Response 发给 Spider 解析；(7) Spider 解析产生 Items 和新的 Requests；(8) Engine 将 Items 发给 Pipeline 处理，将新 Requests 发给 Scheduler；(9) 循环直到 Scheduler 为空。整个过程由 Engine 协调，基于 Twisted 异步事件循环，所有 I/O 操作（网络请求、数据库写入）都是非阻塞的。

**Q2: Scrapy 的去重机制是什么？如何自定义去重策略？**

> 默认去重：`RFPDupeFilter` 基于 Request 的 SHA1 哈希值（由 method + url + body 计算得出），存储在内存集合中。缺点：(1) 重启后去重集合丢失；(2) URL 参数顺序不同但语义相同时会被当作不同请求；(3) 大量 URL 时内存占用高。自定义策略：(1) **规范化 URL**——在 `process_request` 中统一参数排序、移除追踪参数（utm_* 等）；(2) **Redis 去重**——scrapy-redis 的 `RFPDupeFilter` 将指纹存 Redis，重启不丢失，分布式共享；(3) **Bloom Filter**——用布隆过滤器降低内存占用（有误判率，适合海量 URL）；(4) **自定义指纹函数**——继承 `RFPDupeFilter`，重写 `request_fingerprint()` 方法，如只按 URL 路径去重（忽略查询参数）。

**Q3: Spider 的 parse 方法中 yield 的 Request 和 Item 分别去了哪里？**

> (1) **yield Item**——Item 被 Engine 发送到 Item Pipeline，按 `ITEM_PIPELINES` 配置的优先级依次执行（数字小的先执行），每个 Pipeline 的 `process_item()` 方法处理后必须 return item（传给下一个 Pipeline）或 raise DropItem（丢弃）。典型流程：验证 → 去重 → 清洗 → 存储；(2) **yield Request**——Request 被 Engine 发送到 Scheduler，加入待爬队列（经过去重检查），等待被调度执行。Request 的 `callback` 参数指定响应回来后的处理方法，`meta` 字典传递数据到回调方法。关键：Spider 产生的 Request 和 Item 走完全不同的路径——Request 回到调度循环，Item 进入数据处理管道。

**Q4: Scrapy 如何处理 JavaScript 渲染的页面？**

> 三种方案：(1) **Splash**——Scrapy 官方推荐的 JS 渲染方案，Splash 是一个轻量级 HTTP API 的浏览器服务（基于 Qt WebKit），通过 `scrapy-splash` 中间件将请求转发给 Splash 渲染后返回 HTML。优点：与 Scrapy 集成好、支持 Lua 脚本控制渲染过程、资源占用比完整浏览器小；(2) **Playwright/Puppeteer**——通过自定义 Downloader Middleware 将请求交给 Playwright 渲染，功能最强（完整 Chromium），但资源消耗大（每个实例约 100MB 内存）。`scrapy-playwright` 库提供了现成集成；(3) **分析 API**——最高效方案，用 DevTools 分析页面实际请求的 API，直接请求 API 获取 JSON 数据，完全跳过 JS 渲染。推荐优先级：分析 API > Splash > Playwright。

**Q5: Scrapy 的中间件机制是什么？Downloader Middleware 和 Spider Middleware 有什么区别？**

> 中间件是 Scrapy 的拦截器模式，在请求/响应流转过程中插入自定义逻辑。Downloader Middleware 拦截 Engine→Downloader 和 Downloader→Engine 之间的数据，方法：(1) `process_request()`——请求发送前拦截，可修改请求、返回 Response（跳过下载）、或 raise IgnoreRequest；(2) `process_response()`——响应返回后拦截，可修改响应或返回新 Request；(3) `process_exception()`——下载异常时处理。典型用途：UA 轮换、代理、重试、Cookie 处理、JS 渲染。Spider Middleware 拦截 Engine↔Spider 之间的数据，方法：(1) `process_spider_input()`——Response 进入 Spider 前；(2) `process_spider_output()`——Spider 产出 Items/Requests 后；(3) `process_spider_exception()`——Spider 异常时。典型用途：过滤 Items、修改 Spider 输出、异常监控。区别：Downloader Middleware 在"网络层"工作（处理请求和响应），Spider Middleware 在"业务层"工作（处理解析结果）。

**Q6: 如何优化 Scrapy 的爬取速度？**

> 六个层面：(1) **并发数调优**——增大 `CONCURRENT_REQUESTS`（默认 16），但不要超过目标站点承载能力，按域名限制 `CONCURRENT_REQUESTS_PER_DOMAIN`；(2) **AutoThrottle**——开启 `AUTOTHROTTLE_ENABLED=True`，Scrapy 根据服务器响应时间自动调整并发和延迟，避免被 ban；(3) **减少下载量**——`DOWNLOAD_MAXSIZE` 限制响应体大小，禁用不需要的资源（图片、CSS），`HTTPDOWNLOAD_IGNORE_SIZES` 跳过大文件；(4) **连接复用**——Scrapy 默认通过 Twisted 的连接池复用 TCP 连接，确保 `CONCURRENT_REQUESTS` 不为 1；(5) **缓存**——开发阶段开启 `HTTPCACHE_ENABLED`，避免重复下载相同页面；(6) **分布式**——scrapy-redis 多机协作，线性提升吞吐量。瓶颈通常在"下载等待"（I/O 密集），不是 CPU，所以增大并发比优化解析更有效。

**Q7: Scrapy 的 Item Loader 是什么？和直接赋值 Item 有什么区别？**

> Item Loader 提供了一种更结构化的数据填充方式，相比直接赋值 Item 有三大优势：(1) **输入/输出处理器**——`MapCompose` 对每个输入值依次执行函数链（如 strip → 正则提取），`TakeFirst` 取第一个非空值，`Join` 拼接多值，在 Item 定义时声明，所有 Spider 共用，避免每个 parse 方法重复写清洗逻辑；(2) **收集多来源数据**——同一个字段可以从多个 CSS 选择器收集数据，Item Loader 自动合并，如 `loader.add_css('name', 'h1::text')` + `loader.add_css('name', '.title::text')`；(3) **上下文传递**——`loader = ItemLoader(item=Item(), response=response)` 自动传入 response，选择器可直接使用。直接赋值适合简单场景，Item Loader 适合字段多、清洗规则复杂的爬虫。

**Q8: Scrapy 和 requests+BeautifulSoup 方案各自适合什么场景？**

> requests+BS4 适合：(1) **小规模一次性采集**——几十个页面的数据抓取，写个脚本就够；(2) **快速原型**——不确定数据结构时先快速验证；(3) **简单场景**——无分页、无深度跟进、单页采集；(4) **灵活控制**——需要与 pandas、数据库等非爬虫工具深度集成。Scrapy 适合：(1) **大规模持续采集**——全站爬取、定期增量更新；(2) **复杂数据流**——多层级跟进、Pipeline 链式处理、多存储后端；(3) **生产级部署**——需要监控、日志、限速、代理、去重等工程化能力；(4) **分布式**——多机协作爬取。判断标准：脚本不超过 50 行就用 requests，超过 50 行或需要持续运行就用 Scrapy。Scrapy 的初始成本高（项目结构、配置），但复杂度超过阈值后边际成本反而低。

---

**相关链接：** [[Requests与HTTP客户端]] [[BeautifulSoup与HTML解析]] [[Selenium与浏览器自动化]] [[反爬策略与应对]] [[Redis与缓存策略]]
