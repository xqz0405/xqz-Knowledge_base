---
tags:
  - Python
  - 网络请求与数据采集
  - BeautifulSoup
  - lxml
  - HTML解析
date: 2026-05-14
status: 已完成
difficulty: 中等
---

# BeautifulSoup与HTML解析

## What — 是什么

> BeautifulSoup 是 Python 最流行的 HTML/XML 解析库，提供优雅的树形遍历和搜索 API，配合 lxml 或 html.parser 解析器，从网页中提取结构化数据。lxml 是高性能的 XML/HTML 解析库，基于 C 库 libxml2，速度是 BeautifulSoup 的 10-100 倍。

**核心概念：**

- **BeautifulSoup**：HTML 文档解析和遍历库，将 HTML 转为树形结构，提供 `find()`、`find_all()`、`select()` 等搜索方法
- **Tag 对象**：HTML 标签的 Python 表示，包含名称、属性、文本、子节点等信息
- **解析器**：底层 HTML 解析引擎——`html.parser`（标准库，容错好）、`lxml`（C 实现，速度最快）、`html5lib`（最严格，按浏览器方式解析）
- **CSS 选择器**：`select()` 方法支持 jQuery 风格的 CSS 选择器语法，精确选取元素
- **lxml**：高性能 XML/HTML 处理库，支持 XPath、XSLT、Schema 验证，是 BeautifulSoup 的最佳解析器搭档

**核心架构：**

- BeautifulSoup 流程：HTML 字符串 → 解析器 → 文档树（Tag/NavigableString/Comment） → 搜索/遍历 → 提取数据
- lxml 流程：HTML 字符串 → C 解析器 → ElementTree → XPath 查询 → 提取数据
- 性能对比：lxml XPath > lxml CSS选择器 > BeautifulSoup(lxml) > BeautifulSoup(html.parser) > html5lib

**关键特性：**

- BeautifulSoup：自动编码检测、损坏 HTML 容错、Unicode 支持、CSS 选择器
- lxml：XPath 1.0、CSS 选择器（lxml.cssselect）、流式解析、增量解析、HTML 清理

## Why — 为什么

**适用场景：**

- 网页数据采集——提取新闻标题、商品价格、评论内容
- API 数据清洗——解析返回的 HTML 片段
- 自动化测试——验证页面结构和内容
- 数据迁移——从旧系统 HTML 导出数据
- 日志解析——解析 HTML 格式的日志和报告

**对比解析方案：**

| 维度 | BeautifulSoup + lxml | 纯 lxml | pyquery | 正则表达式 |
|------|---------------------|---------|---------|-----------|
| API 风格 | Pythonic、链式调用 | XPath + ElementTree | jQuery 风格 | 字符串匹配 |
| 速度 | 中 | 极快 | 快 | 极快 |
| 容错能力 | 强 | 中 | 中 | 无 |
| XPath | 不支持 | 支持 | 不支持 | 不适用 |
| CSS 选择器 | 支持 | 支持（cssselect） | 支持 | 不适用 |
| 学习曲线 | 低 | 中 | 低（会 jQuery 即可） | 高 |
| Unicode | 自动处理 | 自动处理 | 自动处理 | 需手动 |
| 适用场景 | 通用 HTML 解析 | 高性能/复杂结构 | jQuery 用户 | 简单文本提取 |

**优缺点：**

- ✅ BeautifulSoup 优点：
  - API 极其简洁，上手快
  - 损坏 HTML 容错能力最强
  - 文档和社区最完善
  - 支持多种解析器切换
- ❌ BeautifulSoup 缺点：
  - 速度比纯 lxml 慢（封装开销）
  - 不支持 XPath
  - 内存占用较高（全量解析）
- ✅ lxml 优点：
  - 速度极快（C 实现）
  - XPath 功能强大
  - 流式解析节省内存
- ❌ lxml 缺点：
  - C 依赖，安装有时困难
  - API 偏底层
  - 容错不如 BeautifulSoup

## How — 怎么用

### 1. BeautifulSoup 基础

```bash
pip install beautifulsoup4 lxml
```

```python
from bs4 import BeautifulSoup

html = """
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>示例页面</title>
</head>
<body>
    <div id="main" class="container">
        <h1 class="title">文章标题</h1>
        <p class="summary">这是文章摘要</p>

        <ul class="tags">
            <li class="tag">Python</li>
            <li class="tag">爬虫</li>
            <li class="tag">数据分析</li>
        </ul>

        <div class="articles">
            <article data-id="1">
                <h2><a href="/posts/1">第一篇文章</a></h2>
                <p class="author">作者：张三</p>
                <span class="date">2026-01-15</span>
            </article>
            <article data-id="2">
                <h2><a href="/posts/2">第二篇文章</a></h2>
                <p class="author">作者：李四</p>
                <span class="date">2026-02-20</span>
            </article>
            <article data-id="3">
                <h2><a href="/posts/3">第三篇文章</a></h2>
                <p class="author">作者：王五</p>
                <span class="date">2026-03-10</span>
            </article>
        </div>

        <table id="data-table">
            <tr><th>名称</th><th>价格</th><th>库存</th></tr>
            <tr><td>商品A</td><td>99.9</td><td>50</td></tr>
            <tr><td>商品B</td><td>199.0</td><td>30</td></tr>
        </table>
    </div>
</body>
</html>
"""

# 创建 BeautifulSoup 对象
soup = BeautifulSoup(html, 'lxml')       # 推荐：lxml 解析器（最快）
# soup = BeautifulSoup(html, 'html.parser')  # 标准库，无需额外安装
# soup = BeautifulSoup(html, 'html5lib')     # 最严格，按浏览器方式解析
```

### 2. 搜索与查找

```python
# find() — 查找第一个匹配元素
title = soup.find('h1')                    # <h1 class="title">文章标题</h1>
title_text = title.text                     # '文章标题'
title_class = title['class']               # ['title']

# find_all() — 查找所有匹配元素
tags = soup.find_all('li', class_='tag')   # 所有 <li class="tag">
tag_texts = [tag.text for tag in tags]     # ['Python', '爬虫', '数据分析']

# 按属性查找
article = soup.find('article', attrs={'data-id': '2'})
articles = soup.find_all('article')

# 组合条件
author = soup.find('p', class_='author', string='作者：李四')

# 限制返回数量
first_two = soup.find_all('article', limit=2)

# 递归控制（不搜索子标签）
direct_children = soup.find_all('div', recursive=False)
```

**CSS 选择器（推荐，更灵活）：**

```python
# select() — CSS 选择器，返回列表
# select_one() — CSS 选择器，返回第一个

# 标签选择
articles = soup.select('article')

# class 选择
tags = soup.select('.tag')

# id 选择
main_div = soup.select_one('#main')

# 后代选择
article_links = soup.select('article a')

# 属性选择
article_2 = soup.select_one('article[data-id="2"]')
links_with_href = soup.select('a[href]')

# 子选择器（直接子元素）
items = soup.select('#data-table > tr')

# 伪选择器
first_article = soup.select_one('article:first-child')
last_tag = soup.select_one('.tag:last-child')
nth_article = soup.select_one('article:nth-child(2)')

# 组合选择器
author_in_article = soup.select('article .author')
```

**查找方法对比：**

| 方法 | 返回 | 适用场景 |
|------|------|----------|
| `find(name, attrs)` | 单个 Tag | 精确查一个元素 |
| `find_all(name, attrs)` | 列表 | 查多个同标签元素 |
| `select(selector)` | 列表 | 复杂 CSS 选择器 |
| `select_one(selector)` | 单个 Tag | 复杂选择取第一个 |
| `find_parent()` | 单个 Tag | 向上找父元素 |
| `find_next_sibling()` | 单个 Tag | 找下一个兄弟元素 |
| `find_previous_sibling()` | 单个 Tag | 找上一个兄弟元素 |

### 3. 遍历文档树

```python
# 子节点遍历
div = soup.find('div', class_='articles')

# 直接子标签（不包含文本节点）
for child in div.children:
    if child.name:  # 跳过 NavigableString（空白文本）
        print(child.name, child.get('data-id'))

# 所有子孙标签
for descendant in div.descendants:
    if descendant.name:
        print(descendant.name)

# 父节点与兄弟节点
article = soup.find('article')
parent = article.parent                    # 父标签
parents = list(article.parents)            # 所有祖先

next_sibling = article.find_next_sibling()  # 下一个兄弟
prev_sibling = article.find_previous_sibling()  # 上一个兄弟
next_siblings = article.find_next_siblings()    # 后续所有兄弟

# 文本内容
article = soup.find('article')
all_text = article.get_text()              # 所有文本拼接
all_text_strip = article.get_text(strip=True)  # 去空白
all_text_sep = article.get_text(separator='|')  # 用分隔符

# 字符串搜索
author_elements = soup.find_all(string='作者：张三')       # 精确匹配
author_elements = soup.find_all(string=re.compile(r'作者'))  # 正则匹配
```

### 4. 提取属性与文本

```python
# 属性操作
link = soup.find('a')
link['href']                  # '/posts/1'
link.get('href')              # '/posts/1'（不存在返回 None）
link.get('href', '#')         # 默认值
link.attrs                    # {'href': '/posts/1'}

# 多值属性
div = soup.find('div', id='main')
div['class']                  # ['container']（class 自动拆分为列表）
div['id']                     # 'main'（单值属性是字符串）

# 修改属性
link['href'] = '/new-url'
link['target'] = '_blank'     # 添加新属性
del link['target']            # 删除属性

# 文本提取
h1 = soup.find('h1')
h1.string                     # '文章标题'（直接子文本）
h1.text                       # '文章标题'（所有子孙文本拼接）
h1.get_text()                 # 等价于 .text

# 嵌套标签中的文本
article = soup.find('article')
article.string                # None（有子标签时为 None）
article.text                  # '第一篇文章\n作者：张三\n2026-01-15'
article.get_text(strip=True)  # '第一篇文章作者：张三2026-01-15'

# 遍历所有文本节点
for string in article.strings:     # 含空白
    print(repr(string))
for string in article.stripped_strings:  # 去空白
    print(string)
```

### 5. 表格数据提取

```python
# 通用表格解析
def parse_table(table_tag):
    """将 HTML table 解析为二维列表"""
    headers = []
    rows = []

    # 表头
    header_row = table_tag.find('tr')
    if header_row:
        headers = [th.get_text(strip=True) for th in header_row.find_all('th')]

    # 数据行
    for tr in table_tag.find_all('tr')[1:]:  # 跳过表头行
        cells = tr.find_all(['td', 'th'])
        row = [cell.get_text(strip=True) for cell in cells]
        if row:
            rows.append(row)

    return headers, rows

# 使用
table = soup.find('table', id='data-table')
headers, rows = parse_table(table)
print(headers)  # ['名称', '价格', '库存']
print(rows)     # [['商品A', '99.9', '50'], ['商品B', '199.0', '30']]

# 转为 pandas DataFrame（常见操作）
import pandas as pd
df = pd.DataFrame(rows, columns=headers)
print(df)
#     名称    价格  库存
# 0  商品A  99.9  50
# 1  商品B 199.0  30

# pandas 直接读取 HTML 表格
dfs = pd.read_html(str(table))
df = dfs[0]  # 第一个表格
```

### 6. lxml 与 XPath

lxml 是高性能的替代方案，XPath 是其核心查询语言。

```bash
pip install lxml
```

```python
from lxml import etree

html = """
<div id="main">
    <h1 class="title">文章标题</h1>
    <ul class="tags">
        <li class="tag active">Python</li>
        <li class="tag">爬虫</li>
        <li class="tag">数据分析</li>
    </ul>
    <div class="articles">
        <article data-id="1">
            <h2><a href="/posts/1">第一篇</a></h2>
            <p class="author">张三</p>
        </article>
        <article data-id="2">
            <h2><a href="/posts/2">第二篇</a></h2>
            <p class="author">李四</p>
        </article>
    </div>
</div>
"""

# 解析 HTML
tree = etree.HTML(html)

# XPath 基础查询
# /  — 直接子节点
# // — 所有子孙节点
# @  — 属性
# [] — 谓词（条件）

# 查找所有 li
lis = tree.xpath('//li')
print([li.text for li in lis])  # ['Python', '爬虫', '数据分析']

# 按属性查找
title = tree.xpath('//h1[@class="title"]/text()')[0]  # '文章标题'
links = tree.xpath('//a/@href')  # ['/posts/1', '/posts/2']

# 按位置查找
first_article = tree.xpath('//article[1]//h2/text()')[0]  # '第一篇'
last_tag = tree.xpath('//li[@class="tag"][last()]/text()')[0]  # '数据分析'

# 多条件
active_tag = tree.xpath('//li[contains(@class, "active")]/text()')[0]  # 'Python'
articles_with_id = tree.xpath('//article[@data-id="2"]//p/text()')[0]  # '李四'

# 常用 XPath 函数
tree.xpath('count(//article)')         # 2.0（数量）
tree.xpath('//li[position()<=2]/text()')  # ['Python', '爬虫']（前两个）
tree.xpath('//h1/normalize-space()')    # '文章标题'（去空白）
```

**XPath 语法速查：**

| 表达式 | 含义 | 示例 |
|--------|------|------|
| `//div` | 所有 div | `//article` |
| `/div` | 根节点下的 div | `/html/body/div` |
| `@class` | class 属性 | `div@class` |
| `[@id='main']` | id 为 main | `div[@id='main']` |
| `[1]` | 第一个 | `//article[1]` |
| `[last()]` | 最后一个 | `//li[last()]` |
| `[position()>2]` | 第 3 个起 | `//tr[position()>1]` |
| `contains(@class,'tag')` | class 含 tag | `//div[contains(@class,'tag')]` |
| `starts-with(@href,'http')` | href 以 http 开头 | `//a[starts-with(@href,'http')]` |
| `text()` | 文本内容 | `//h1/text()` |
| `..` | 父节点 | `//h2/..` |
| `*` | 任意元素 | `//article/*` |
| `|` | 联合（或） | `//h1 \| //h2` |

**lxml CSS 选择器：**

```python
from lxml import etree
from lxml.cssselect import CSSSelector

tree = etree.HTML(html)

# 方式一：CSSSelector 编译
sel = CSSSelector('article .author')
elements = sel(tree)
print([el.text for el in elements])  # ['张三', '李四']

# 方式二：cssselect 直接转 XPath
from lxml.cssselect import cssselect
xpath_expr = cssselect('article[data-id="2"] .author')
# 生成的 XPath: "descendant-or-self::article[@data-id and @data-id='2']/descendant-or-self::*[@class and contains(concat(' ', normalize-space(@class), ' '), ' author ')]"
elements = tree.xpath(xpath_expr)
```

### 7. BeautifulSoup + lxml 混用

最佳实践：BeautifulSoup 解析 + CSS 选择器 + lxml XPath 补充。

```python
from bs4 import BeautifulSoup
from lxml import etree

# BeautifulSoup 负责 HTML 容错解析
soup = BeautifulSoup(html, 'lxml')

# 需要时转到 lxml 做 XPath 查询
doc = etree.HTML(str(soup))

# 复杂查询用 XPath
prices = doc.xpath('//table//tr/td[2]/text()')

# 简单查询用 BeautifulSoup
title = soup.select_one('h1.title').text
```

### 8. 实战：完整网页爬取

```python
"""爬取新闻网站：请求 + 解析 + 存储"""
import requests
from bs4 import BeautifulSoup
import json
import csv
import time
from typing import List, Dict


class NewsScraper:
    """新闻爬虫示例"""

    def __init__(self, base_url: str):
        self.base_url = base_url
        self.session = requests.Session()
        self.session.headers.update({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
        })

    def fetch_page(self, url: str) -> BeautifulSoup:
        """请求页面并解析"""
        response = self.session.get(url, timeout=10)
        response.raise_for_status()
        response.encoding = response.apparent_encoding  # 自动检测编码
        return BeautifulSoup(response.text, 'lxml')

    def parse_list_page(self, soup: BeautifulSoup) -> List[Dict]:
        """解析列表页：提取文章标题、链接、日期"""
        articles = []

        for item in soup.select('article'):
            title_tag = item.select_one('h2 a')
            if not title_tag:
                continue

            article = {
                'title': title_tag.get_text(strip=True),
                'url': title_tag['href'],
                'date': '',
                'author': '',
                'summary': '',
            }

            # 日期
            date_tag = item.select_one('.date, time, [datetime]')
            if date_tag:
                article['date'] = date_tag.get('datetime') or date_tag.get_text(strip=True)

            # 作者
            author_tag = item.select_one('.author, .byline')
            if author_tag:
                article['author'] = author_tag.get_text(strip=True)

            # 摘要
            summary_tag = item.select_one('.summary, .excerpt, p')
            if summary_tag:
                article['summary'] = summary_tag.get_text(strip=True)[:200]

            articles.append(article)

        return articles

    def parse_detail_page(self, soup: BeautifulSoup) -> Dict:
        """解析详情页：提取正文内容"""
        content_area = soup.select_one('.article-content, .post-body, article')

        if not content_area:
            return {'content': '', 'images': []}

        # 移除不需要的元素
        for tag in content_area.select('script, style, .ad, .sidebar, nav'):
            tag.decompose()

        # 提取正文
        content = content_area.get_text(separator='\n', strip=True)

        # 提取图片
        images = []
        for img in content_area.select('img'):
            src = img.get('src') or img.get('data-src')
            if src:
                images.append({
                    'url': src,
                    'alt': img.get('alt', ''),
                })

        return {'content': content, 'images': images}

    def scrape(self, max_pages: int = 3) -> List[Dict]:
        """完整爬取流程"""
        all_articles = []

        for page in range(1, max_pages + 1):
            url = f'{self.base_url}/page/{page}'
            print(f'正在爬取第 {page} 页: {url}')

            try:
                soup = self.fetch_page(url)
                articles = self.parse_list_page(soup)

                if not articles:
                    print(f'第 {page} 页没有文章，停止爬取')
                    break

                # 爬取详情页
                for article in articles[:5]:  # 限制每页爬 5 篇
                    detail_url = article['url']
                    if not detail_url.startswith('http'):
                        detail_url = self.base_url + detail_url

                    try:
                        detail_soup = self.fetch_page(detail_url)
                        detail = self.parse_detail_page(detail_soup)
                        article.update(detail)
                    except Exception as e:
                        print(f'  详情页爬取失败: {e}')

                    time.sleep(1)  # 礼貌延迟

                all_articles.extend(articles)
                print(f'  获取 {len(articles)} 篇文章')

            except Exception as e:
                print(f'第 {page} 页爬取失败: {e}')

            time.sleep(2)  # 页间延迟

        return all_articles

    @staticmethod
    def save_json(data: List[Dict], filepath: str):
        """保存为 JSON"""
        with open(filepath, 'w', encoding='utf-8') as f:
            json.dump(data, f, ensure_ascii=False, indent=2)

    @staticmethod
    def save_csv(data: List[Dict], filepath: str):
        """保存为 CSV"""
        if not data:
            return
        with open(filepath, 'w', encoding='utf-8-sig', newline='') as f:
            writer = csv.DictWriter(f, fieldnames=data[0].keys())
            writer.writeheader()
            writer.writerows(data)


# 使用
if __name__ == '__main__':
    scraper = NewsScraper('https://news.example.com')
    articles = scraper.scrape(max_pages=2)
    scraper.save_json(articles, 'news.json')
    scraper.save_csv(articles, 'news.csv')
    print(f'共爬取 {len(articles)} 篇文章')
```

### 9. lxml 流式解析（大文件）

```python
from lxml import etree
from io import BytesIO

# 大型 XML/HTML 文件流式解析（不一次性加载到内存）
def parse_large_html(filepath, tag='article'):
    """增量解析大型 HTML 文件"""
    context = etree.iterparse(filepath, events=('end',), tag=tag)

    for event, elem in context:
        # 处理每个 article 元素
        title = elem.xpath('.//h2/text()')
        author = elem.xpath('.//p[@class="author"]/text()')

        yield {
            'title': title[0] if title else '',
            'author': author[0] if author else '',
        }

        # 清理已处理的元素，释放内存
        elem.clear()
        # 同时清理前面的兄弟节点
        while elem.getprevious() is not None:
            del elem.getparent()[0]


# 使用
for article in parse_large_html('large_page.html', tag='article'):
    print(article)
```

### 10. 编码与清洗

```python
from bs4 import BeautifulSoup
import re

# 编码问题处理
response = requests.get('https://example.com')
# 方式一：让 requests 自动检测
response.encoding = response.apparent_encoding

# 方式二：BeautifulSoup 自动处理
soup = BeautifulSoup(response.content, 'lxml')  # 传 bytes 让 BS 自动检测

# 方式三：手动指定
soup = BeautifulSoup(response.content.decode('gbk'), 'lxml')

# HTML 清理（移除脚本、样式等无关内容）
def clean_html(soup: BeautifulSoup) -> str:
    """清洗 HTML，只保留有意义的文本"""
    # 移除不需要的标签
    for tag in soup.find_all(['script', 'style', 'nav', 'footer', 'header', 'aside']):
        tag.decompose()

    # 移除广告和无关内容
    for tag in soup.find_all(class_=re.compile(r'ad|banner|popup|cookie|social')):
        tag.decompose()

    # 移除注释
    for comment in soup.find_all(string=lambda text: isinstance(text, Comment)):
        comment.extract()

    return soup.get_text(separator='\n', strip=True)

# 提取正文（通用方案）
def extract_content(url: str) -> dict:
    """通用正文提取"""
    response = requests.get(url, timeout=10)
    response.encoding = response.apparent_encoding
    soup = BeautifulSoup(response.text, 'lxml')

    # 标题
    title = soup.find('title')
    title_text = title.text.strip() if title else ''

    # 正文区域（常见选择器优先级）
    content_selectors = [
        'article', '.article-content', '.post-body', '.entry-content',
        '.content-body', '#article-content', 'main',
    ]
    content = None
    for selector in content_selectors:
        content = soup.select_one(selector)
        if content:
            break

    if not content:
        content = soup.find('body') or soup

    # 清洗并提取
    clean_text = clean_html(content)

    return {
        'title': title_text,
        'content': clean_text,
        'url': url,
    }
```

---

## 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 乱码 | 编码不匹配 | 传 bytes 而非 str 给 BS，或手动设置 `response.encoding` |
| find_all 返回空 | 选择器不匹配 | 检查 class 名是否有动态前缀，用 `contains()` 或正则 |
| 属性不存在报错 | 直接 `tag['attr']` 不存在时抛 KeyError | 用 `tag.get('attr')` 或 `tag.get('attr', '')` |
| text 含大量空白 | HTML 缩进和换行产生空白 | `get_text(strip=True)` 或 `separator='\n'` |
| lxml 安装失败 | C 依赖编译问题 | Windows 用 wheel 包，Linux 装 `libxml2-dev` |
| 解析速度慢 | html5lib 或 html.parser 太慢 | 切换为 `BeautifulSoup(html, 'lxml')` |
| 嵌套标签文本提取不准 | `.string` 只返回直接子文本 | 用 `.get_text()` 或 `.text` 获取所有子孙文本 |
| 动态内容拿不到 | JS 动态渲染的内容不在 HTML 中 | 配合 [[Selenium与浏览器自动化]] 获取渲染后 HTML |

---

## 面试题

**Q1: BeautifulSoup 的三种解析器有什么区别？如何选择？**

> (1) **lxml**——C 实现，速度最快，容错能力好，是首选解析器；(2) **html.parser**——Python 标准库，无需额外安装，速度中等，容错能力还行，适合零依赖场景；(3) **html5lib**——纯 Python，按浏览器标准解析（如自动补全缺失标签），速度最慢，但解析结果最接近浏览器渲染结果。选择原则：生产环境一律用 `lxml`（`BeautifulSoup(html, 'lxml')`），只有在无法安装 C 扩展的环境才用 `html.parser`，需要精确模拟浏览器解析时用 `html5lib`。lxml 的唯一缺点是需要 C 编译环境，但预编译 wheel 包覆盖了所有主流平台。

**Q2: BeautifulSoup 的 find/find_all 和 select/select_one 有什么区别？**

> (1) **API 风格**——`find()` 用 Python 字典传属性（`find('div', class_='main')`），`select()` 用 CSS 选择器语法（`select('div.main')`）；(2) **功能范围**——`select()` 支持更复杂的组合选择器（`article[data-id="1"] .author`）、伪选择器（`:first-child`、`:nth-child(n)`），`find()` 需要多步链式调用才能实现同等效果；(3) **性能**——简单查找 `find()` 稍快，复杂查找 `select()` 更高效（底层编译为一次性匹配）；(4) **返回值**——`find()` 返回单个 Tag 或 None，`find_all()` 返回列表，`select()` 返回列表，`select_one()` 返回单个或 None。推荐：简单查找用 `find()`，复杂选择用 `select()`，日常编码 `select()` 统一即可。

**Q3: BeautifulSoup 和 lxml 各自的优劣势是什么？如何配合使用？**

> BeautifulSoup 优势：(1) API 更 Pythonic、更简洁；(2) 容错能力最强，能处理严重损坏的 HTML；(3) 文档和社区最好。劣势：(1) 速度比纯 lxml 慢（封装开销约 2-10 倍）；(2) 不支持 XPath。lxml 优势：(1) C 实现，速度极快；(2) XPath 功能强大，适合复杂查询；(3) 流式解析节省内存。劣势：(1) C 依赖安装麻烦；(2) API 偏底层。最佳实践：**BeautifulSoup + lxml 解析器**——用 BeautifulSoup 的容错解析和简洁 API，底层用 lxml 做解析引擎（`BeautifulSoup(html, 'lxml')`），需要 XPath 时再转到 lxml（`etree.HTML(str(soup))`）。这样既享受 BS 的容错和简洁，又在需要时获得 XPath 的高性能。

**Q4: XPath 和 CSS 选择器各自的适用场景是什么？**

> CSS 选择器：(1) 语法简洁，前端开发者熟悉；(2) 适合按类名、ID、属性、层级查找；(3) 不支持按文本内容查找（XPath 的杀手锏）；(4) 不支持向上查找父节点。XPath：(1) 功能更强大——按文本内容查找（`//a[contains(text(), '更多')]`）、向上查找（`//h2/..`）、数值比较（`//td[number()>100]`）、位置计算；(2) 适合复杂结构化数据提取（表格、列表）；(3) 学习曲线比 CSS 高；(4) 在 lxml 中性能最优。选择建议：简单选择用 CSS（前端背景开发者上手快），复杂查询（按文本、按位置、向上查找）用 XPath。爬虫中 XPath 用得更多，因为很多场景需要"找到包含某文本的标签"。

**Q5: 如何处理网页编码问题？**

> 编码问题根源：HTTP 响应的编码声明、HTML meta 声明、实际编码三者可能不一致。处理方法：(1) **传 bytes 给 BeautifulSoup**——`BeautifulSoup(response.content, 'lxml')`，BS 内部会自动检测编码（优先 HTTP Content-Type，再 HTML meta charset，最后 chardet/charset-normalizer）；(2) **requests 自动检测**——`response.apparent_encoding` 使用 chardet 检测，比 `response.encoding`（从 HTTP 头推断）更准确；(3) **手动指定**——中文网站常见 GBK/GB2312，设置 `response.encoding = 'gbk'`；(4) **lxml 自动处理**——etree.HTML 接受 bytes 时会自动检测编码。最佳实践：始终传 bytes（`response.content`）给解析器，让解析器自动处理编码，避免手动解码导致错误。

**Q6: 如何高效提取网页中的表格数据？**

> 三种方案：(1) **BeautifulSoup**——手动遍历 `<tr>` 和 `<td>`，灵活但代码多，适合不规则表格；(2) **pandas.read_html()**——一行代码解析表格，底层用 lxml，自动处理 colspan/rowspan，返回 DataFrame 列表，最推荐；(3) **lxml XPath**——`//table//tr/td` 批量提取，速度快。推荐流程：先试 `pd.read_html(str(table))`，如果解析不正确再手动 BeautifulSoup。注意事项：(1) `pd.read_html` 需要 lxml 或 html5lib；(2) 合并单元格（colspan/rowspan）需要特殊处理；(3) 表格嵌套时需先定位到目标表格再解析。

**Q7: BeautifulSoup 如何修改 HTML？**

> 修改操作：(1) **修改标签内容**——`tag.string = 'new text'`（替换直接子文本）；`tag.clear()` 清空所有子节点；(2) **修改属性**——`tag['class'] = 'new-class'`、`tag['href'] = '/new'`、`del tag['target']`；(3) **添加节点**——`tag.append(new_tag)`（末尾添加）、`tag.insert(0, new_tag)`（指定位置）、`tag.insert_before(new_tag)`（前面插入）、`tag.insert_after(new_tag)`（后面插入）；(4) **删除节点**——`tag.decompose()`（彻底删除，释放内存）、`tag.extract()`（移除并返回，可复用）；(5) **替换节点**——`tag.replace_with(new_tag)`；(6) **包装**——`tag.wrap(soup.new_tag('div'))`。典型应用：爬虫中用 `decompose()` 移除广告和脚本后再提取文本；用 `wrap()` 给无语义的标签添加结构。

**Q8: 如何处理动态渲染的网页（JS 生成的 HTML）？**

> BeautifulSoup 只能解析静态 HTML，JS 动态渲染的内容不在初始 HTML 中。处理方案：(1) **分析 API 请求**——用浏览器 DevTools 的 Network 面板找到 JS 实际调用的 API，直接请求 API 获取 JSON 数据，这是最高效的方式；(2) **Selenium/Playwright**——用浏览器自动化工具加载页面，等 JS 执行完毕后获取渲染后的 HTML，再用 BeautifulSoup 解析，最通用但最慢；(3) **requests-html**——requests 作者的另一个库，支持 JS 渲染（底层用 Chromium），API 类似 requests；(4) **分析页面源码**——有时 JS 渲染的数据实际上嵌在 `<script>` 标签的 JSON 中（如 `window.__INITIAL_STATE__`），可以用正则提取。推荐优先级：分析 API > 提取内嵌 JSON > Selenium/Playwright。

---

**相关链接：** [[Requests与HTTP客户端]] [[Scrapy框架]] [[Selenium与浏览器自动化]] [[反爬策略与应对]] [[Pandas基础]]
