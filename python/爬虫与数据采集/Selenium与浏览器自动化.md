---
tags:
  - Python
  - 爬虫与数据采集
  - Selenium
  - 浏览器自动化
  - Playwright
date: 2026-05-14
status: 已完成
difficulty: 中等
---

# Selenium与浏览器自动化

## What — 是什么

> Selenium 是 Web 浏览器自动化工具，通过 WebDriver 协议控制真实浏览器执行操作（点击、输入、滚动、截图），用于 JS 动态渲染页面爬取、自动化测试和 RPA 流程。Playwright 是微软出品的下一代浏览器自动化工具，速度更快、API 更现代。

**核心概念：**

- **WebDriver**：浏览器驱动协议，Selenium 通过它向浏览器发送命令（导航、点击、输入等）
- **WebElement**：页面元素的 Python 对象，封装查找、交互、属性获取等操作
- **等待策略**：显式等待（WebDriverWait）和隐式等待（implicitly_wait），解决元素动态加载问题
- **无头模式（Headless）**：不显示浏览器窗口运行，服务器部署和 CI 环境必需
- **Playwright**：新一代浏览器自动化工具，原生支持 Chromium/Firefox/WebKit，自动等待、网络拦截、多标签页

**核心架构：**

- Selenium 架构：Python 代码 → WebDriver 协议 → 浏览器驱动（chromedriver/geckodriver） → 浏览器执行
- Playwright 架构：Python 代码 → Playwright Server → CDP（Chrome DevTools Protocol）/ WebSocket → 浏览器执行
- 关键区别：Selenium 通过 WebDriver 协议（HTTP），Playwright 通过 CDP/WebSocket（更底层更快）

**关键特性：**

- Selenium：多语言支持（Python/Java/JS/C#）、跨浏览器、生态最成熟
- Playwright：自动等待、网络拦截、多浏览器（Chromium/Firefox/WebKit）、代码生成、Trace Viewer

## Why — 为什么

**适用场景：**

- JS 动态渲染页面爬取——SPA、无限滚动、懒加载
- 自动化测试——Web 应用的 E2E 测试
- RPA 流程自动化——自动填表、数据录入、批量操作
- 截图与 PDF——网页快照、报告生成
- 竞品监控——价格监控、库存变化

**对比同类工具：**

| 维度 | Selenium | Playwright | Puppeteer | Cypress |
|------|----------|-----------|-----------|---------|
| 语言支持 | Python/Java/JS/C#/Ruby | Python/JS/Java/.NET | JS/TS | JS/TS |
| 浏览器 | Chrome/Firefox/Safari/Edge | Chromium/Firefox/WebKit | Chromium | Chromium |
| 协议 | WebDriver (HTTP) | CDP + WebSocket | CDP | CDP |
| 自动等待 | 需手动 WebDriverWait | 内置自动等待 | 需手动 | 内置 |
| 网络拦截 | 有限 | 原生支持 | 原生支持 | 原生支持 |
| 多标签页 | 支持 | 原生支持 | 原生支持 | 不支持 |
| 无头模式 | 支持 | 默认无头 | 默认无头 | 默认无头 |
| 速度 | 中 | 快 | 快 | 快 |
| 并行执行 | 需 Selenium Grid | 原生支持 | 需手动 | 原生支持 |
| 代码生成 | Selenium IDE | Playwright Codegen | 无 | 无 |
| 学习曲线 | 中 | 低 | 中 | 低 |
| 定位 | 爬虫 + 测试 | 爬虫 + 测试 | 爬虫为主 | 测试为主 |

**优缺点：**

- ✅ Selenium 优点：
  - 生态最成熟，社区和教程最多
  - 多语言、多浏览器覆盖最广
  - Selenium Grid 分布式执行
- ❌ Selenium 缺点：
  - 速度慢（WebDriver HTTP 协议开销）
  - 没有自动等待，需手动处理时序
  - API 偏老，不够 Pythonic
- ✅ Playwright 优点：
  - 自动等待，减少 flaky 测试
  - 速度快（CDP 协议更底层）
  - API 现代，更 Pythonic
  - 内置网络拦截、截图、Trace
- ❌ Playwright 缺点：
  - 不支持旧版浏览器
  - 社区和生态不如 Selenium 成熟

## How — 怎么用

### 1. Selenium 安装与配置

```bash
pip install selenium
# Selenium 4.6+ 内置 Selenium Manager，自动下载浏览器驱动
# 旧版需要手动下载 chromedriver
```

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# Chrome 配置
options = Options()
options.add_argument('--headless=new')          # 无头模式
options.add_argument('--no-sandbox')             # 容器环境必需
options.add_argument('--disable-dev-shm-usage')  # 限制内存使用
options.add_argument('--disable-gpu')
options.add_argument('--window-size=1920,1080')
options.add_argument('--lang=zh-CN')
options.add_argument('--disable-blink-features=AutomationControlled')  # 反检测

# 反爬虫检测
options.add_experimental_option('excludeSwitches', ['enable-automation'])
options.add_experimental_option('useAutomationExtension', False)

# User-Agent
options.add_argument('--user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/120.0.0.0')

# 代理
# options.add_argument('--proxy-server=http://proxy:8080')

# 禁用图片加载（加速）
# prefs = {'profile.managed_default_content_settings.images': 2}
# options.add_experimental_option('prefs', prefs)

driver = webdriver.Chrome(options=options)

# 反自动化检测
driver.execute_cdp_cmd('Page.addScriptToEvaluateOnNewDocument', {
    'source': '''
        Object.defineProperty(navigator, 'webdriver', {get: () => undefined});
        window.navigator.chrome = {runtime: {}};
        Object.defineProperty(navigator, 'plugins', {get: () => [1, 2, 3]});
        Object.defineProperty(navigator, 'languages', {get: () => ['zh-CN', 'zh', 'en']});
    '''
})

driver.get('https://example.com')
```

### 2. 元素定位与交互

```python
# 8 种定位方式
# By.ID           → find_element(By.ID, 'username')
# By.NAME         → find_element(By.NAME, 'email')
# By.CLASS_NAME   → find_element(By.CLASS_NAME, 'btn-primary')
# By.TAG_NAME     → find_element(By.TAG_NAME, 'h1')
# By.LINK_TEXT    → find_element(By.LINK_TEXT, '登录')
# By.PARTIAL_LINK_TEXT → find_element(By.PARTIAL_LINK_TEXT, '登录')
# By.CSS_SELECTOR → find_element(By.CSS_SELECTOR, '#login-form input[name="user"]')
# By.XPATH        → find_element(By.XPATH, '//button[contains(text(), "提交")]')

# 推荐：CSS_SELECTOR 和 XPATH 最灵活
element = driver.find_element(By.CSS_SELECTOR, '.product-list .item:first-child')
elements = driver.find_elements(By.CSS_SELECTOR, '.product-list .item')

# 输入文本
search_box = driver.find_element(By.CSS_SELECTOR, 'input[name="q"]')
search_box.clear()
search_box.send_keys('Python 爬虫')
search_box.send_keys(Keys.RETURN)  # 回车

# 点击
button = driver.find_element(By.CSS_SELECTOR, 'button.submit')
button.click()

# 获取元素信息
element.text                    # 文本内容
element.get_attribute('href')   # 属性值
element.get_attribute('class')
element.is_displayed()          # 是否可见
element.is_enabled()            # 是否可操作
element.is_selected()           # 是否选中（复选框/单选框）
element.tag_name                # 标签名
element.size                    # 尺寸 {'width': 100, 'height': 30}
element.location                # 位置 {'x': 50, 'y': 100}

# 下拉选择
from selenium.webdriver.support.select import Select
select = Select(driver.find_element(By.CSS_SELECTOR, 'select#country'))
select.select_by_visible_text('中国')
select.select_by_value('CN')
select.select_by_index(0)

# 复选框/单选框
checkbox = driver.find_element(By.CSS_SELECTOR, 'input[type="checkbox"]')
if not checkbox.is_selected():
    checkbox.click()

# 文件上传
file_input = driver.find_element(By.CSS_SELECTOR, 'input[type="file"]')
file_input.send_keys('/path/to/file.pdf')  # 直接传文件路径
```

### 3. 等待策略（关键）

等待是 Selenium 最重要的知识点——页面元素动态加载，直接操作未加载的元素会报错。

```python
# ❌ 隐式等待（全局等待，不推荐单独用）
driver.implicitly_wait(10)  # 全局最多等 10 秒找元素
# 缺点：对所有查找生效，无法针对特定条件

# ✅ 显式等待（推荐）
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# 等待元素出现
element = WebDriverWait(driver, 10).until(
    EC.presence_of_element_located((By.CSS_SELECTOR, '.result-list'))
)

# 等待元素可见
element = WebDriverWait(driver, 10).until(
    EC.visibility_of_element_located((By.CSS_SELECTOR, '.result-item'))
)

# 等待元素可点击
button = WebDriverWait(driver, 10).until(
    EC.element_to_be_clickable((By.CSS_SELECTOR, 'button.submit'))
)

# 等待文本出现
WebDriverWait(driver, 10).until(
    EC.text_to_be_present_in_element(
        (By.CSS_SELECTOR, '.status'), '加载完成'
    )
)

# 等待元素消失（如 loading 遮罩）
WebDriverWait(driver, 10).until(
    EC.invisibility_of_element_located((By.CSS_SELECTOR, '.loading'))
)

# 自定义等待条件
def element_has_text(driver):
    element = driver.find_element(By.CSS_SELECTOR, '.result')
    return element.text if element.text else False

result_text = WebDriverWait(driver, 15).until(element_has_text)

# 等待 URL 变化
WebDriverWait(driver, 10).until(
    EC.url_contains('/dashboard')
)

# 等待新窗口
WebDriverWait(driver, 10).until(
    EC.new_window_is_opened(driver.window_handles)
)
```

**常用等待条件：**

| 条件 | 说明 |
|------|------|
| `presence_of_element_located` | 元素存在于 DOM |
| `visibility_of_element_located` | 元素可见 |
| `element_to_be_clickable` | 元素可点击 |
| `text_to_be_present_in_element` | 元素包含指定文本 |
| `invisibility_of_element_located` | 元素消失 |
| `url_contains` | URL 包含指定字符串 |
| `title_contains` | 标题包含指定字符串 |
| `new_window_is_opened` | 新窗口打开 |
| `frame_to_be_available_and_switch_to_it` | iframe 可用 |

### 4. 页面操作进阶

```python
# JavaScript 执行
driver.execute_script('window.scrollTo(0, document.body.scrollHeight)')  # 滚动到底部
driver.execute_script('arguments[0].scrollIntoView()', element)          # 滚动到元素
driver.execute_script('arguments[0].click()', element)                   # JS 点击（绕过遮挡）

# 获取渲染后的 HTML
page_source = driver.page_source          # 完整 HTML
element_html = element.get_attribute('outerHTML')  # 元素 HTML

# iframe 切换
driver.switch_to.frame('iframe-id')       # 切入 iframe
driver.switch_to.default_content()         # 切回主页面

# 窗口切换
driver.switch_to.window(driver.window_handles[-1])  # 最新窗口
driver.switch_to.window(driver.window_handles[0])    # 第一个窗口

# 弹窗处理
alert = driver.switch_to.alert
alert_text = alert.text
alert.accept()          # 确认
alert.dismiss()         # 取消
alert.send_keys('text') # 输入

# 鼠标操作
from selenium.webdriver.common.action_chains import ActionChains

actions = ActionChains(driver)
actions.move_to_element(element).perform()                    # 悬停
actions.drag_and_drop(source, target).perform()               # 拖拽
actions.context_click(element).perform()                       # 右键
actions.double_click(element).perform()                        # 双击
actions.click_and_hold(element).move_by_offset(50, 0).release().perform()  # 拖动

# 键盘操作
from selenium.webdriver.common.keys import Keys
element.send_keys(Keys.CONTROL + 'a')  # Ctrl+A 全选
element.send_keys(Keys.CONTROL + 'c')  # Ctrl+C 复制
element.send_keys(Keys.TAB)            # Tab

# 截图
driver.save_screenshot('page.png')                     # 整页截图
element.screenshot('element.png')                       # 元素截图

# Cookie 操作
driver.get('https://example.com')
driver.add_cookie({'name': 'session', 'value': 'abc123'})
cookies = driver.get_cookies()
driver.delete_cookie('session')
driver.delete_all_cookies()
```

### 5. Playwright 快速上手

```bash
pip install playwright
playwright install  # 安装浏览器（Chromium + Firefox + WebKit）
```

```python
from playwright.sync_api import sync_playwright

# 同步 API
with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()

    # 导航
    page.goto('https://example.com')

    # 等待 + 点击（自动等待，无需显式 WebDriverWait）
    page.click('button.submit')

    # 输入
    page.fill('input[name="q"]', 'Python 爬虫')

    # 获取文本
    title = page.text_content('h1')

    # 获取属性
    href = page.get_attribute('a.link', 'href')

    # 截图
    page.screenshot(path='screenshot.png')

    # 全页截图
    page.screenshot(path='full.png', full_page=True)

    # PDF
    page.pdf(path='page.pdf')

    # 获取渲染后 HTML
    html = page.content()

    browser.close()
```

```python
# 异步 API（高性能）
import asyncio
from playwright.async_api import async_playwright

async def scrape(urls):
    async with async_playwright() as p:
        browser = await p.chromium.launch(headless=True)

        # 并发爬取
        async def scrape_one(url):
            page = await browser.new_page()
            await page.goto(url, wait_until='networkidle')
            title = await page.title()
            content = await page.content()
            await page.close()
            return {'url': url, 'title': title, 'content': content}

        tasks = [scrape_one(url) for url in urls]
        results = await asyncio.gather(*tasks)

        await browser.close()
        return results

results = asyncio.run(scrape([
    'https://example.com/page1',
    'https://example.com/page2',
]))
```

**Playwright 网络拦截：**

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()

    # 拦截请求：屏蔽图片和样式（加速）
    def handle_route(route):
        if route.request.resource_type in ('image', 'stylesheet', 'font', 'media'):
            route.abort()
        else:
            route.continue_()

    page.route('**/*', handle_route)

    # 拦截 API 响应
    api_data = []

    def handle_api(route):
        response = route.fetch()
        if 'api' in route.request.url:
            data = response.json()
            api_data.append(data)
        route.fulfill(response=response)

    page.route('**/api/**', handle_api)

    page.goto('https://example.com')
    print(f'拦截到 {len(api_data)} 个 API 响应')

    browser.close()
```

### 6. 实战：动态渲染页面爬取

```python
"""Selenium 爬取无限滚动页面"""
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time
import json


class InfiniteScrollScraper:
    """无限滚动页面爬虫"""

    def __init__(self, headless=True, max_scrolls=20):
        options = Options()
        if headless:
            options.add_argument('--headless=new')
        options.add_argument('--no-sandbox')
        options.add_argument('--disable-dev-shm-usage')
        options.add_argument('--window-size=1920,1080')

        self.driver = webdriver.Chrome(options=options)
        self.wait = WebDriverWait(self.driver, 10)
        self.max_scrolls = max_scrolls

    def scrape_infinite_scroll(self, url, item_selector, scroll_pause=2):
        """爬取无限滚动页面"""
        self.driver.get(url)
        items = []
        seen_texts = set()
        last_height = 0

        for scroll_count in range(self.max_scrolls):
            # 提取当前可见项
            elements = self.driver.find_elements(By.CSS_SELECTOR, item_selector)
            for el in elements:
                text = el.text.strip()
                if text and text not in seen_texts:
                    seen_texts.add(text)
                    items.append({'text': text, 'scroll': scroll_count})

            # 滚动到底部
            self.driver.execute_script(
                'window.scrollTo(0, document.body.scrollHeight)'
            )

            # 等待新内容加载
            time.sleep(scroll_pause)

            # 检查是否到底
            new_height = self.driver.execute_script(
                'return document.body.scrollHeight'
            )
            if new_height == last_height:
                print(f'已到底部，共滚动 {scroll_count} 次')
                break
            last_height = new_height

            print(f'第 {scroll_count + 1} 次滚动，已采集 {len(items)} 条')

        return items

    def scrape_lazy_images(self, url, img_selector):
        """爬取懒加载图片"""
        self.driver.get(url)

        images = []
        img_elements = self.driver.find_elements(By.CSS_SELECTOR, img_selector)

        for img in img_elements:
            # 滚动到图片位置触发懒加载
            self.driver.execute_script('arguments[0].scrollIntoView()', img)
            time.sleep(0.5)

            src = img.get_attribute('src') or img.get_attribute('data-src')
            if src and not src.startswith('data:'):
                images.append(src)

        return images

    def login_and_scrape(self, login_url, target_url, credentials):
        """登录后爬取"""
        self.driver.get(login_url)

        # 填写登录表单
        username = self.wait.until(
            EC.presence_of_element_located((By.CSS_SELECTOR, 'input[name="username"]'))
        )
        username.send_keys(credentials['username'])

        password = self.driver.find_element(By.CSS_SELECTOR, 'input[name="password"]')
        password.send_keys(credentials['password'])

        # 点击登录
        submit = self.wait.until(
            EC.element_to_be_clickable((By.CSS_SELECTOR, 'button[type="submit"]'))
        )
        submit.click()

        # 等待登录成功
        self.wait.until(EC.url_contains('/dashboard'))

        # 访问目标页面
        self.driver.get(target_url)
        return self.driver.page_source

    def close(self):
        self.driver.quit()


# 使用
if __name__ == '__main__':
    scraper = InfiniteScrollScraper(headless=True, max_scrolls=10)

    items = scraper.scrape_infinite_scroll(
        url='https://example.com/infinite-list',
        item_selector='.list-item',
    )

    with open('items.json', 'w', encoding='utf-8') as f:
        json.dump(items, f, ensure_ascii=False, indent=2)

    print(f'共采集 {len(items)} 条数据')
    scraper.close()
```

### 7. 性能优化

```python
# Selenium 性能优化策略
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

options = Options()

# 1. 无头模式（必须）
options.add_argument('--headless=new')

# 2. 禁用不必要的功能
options.add_argument('--disable-extensions')
options.add_argument('--disable-infobars')
options.add_argument('--disable-notifications')

# 3. 禁用图片（节省带宽和时间）
prefs = {
    'profile.managed_default_content_settings.images': 2,
    'profile.default_content_setting_values.notifications': 2,
}
options.add_experimental_option('prefs', prefs)

# 4. 设置页面加载策略
# 'normal' — 等待所有资源加载（默认，最慢）
# 'eager'  — DOM ready 即可（不等图片/样式）
# 'none'   — 仅下载 HTML（最快，需手动等待）
options.page_load_strategy = 'eager'

# 5. 设置超时
driver = webdriver.Chrome(options=options)
driver.set_page_load_timeout(30)    # 页面加载超时
driver.set_script_timeout(20)       # JS 执行超时
driver.implicitly_wait(5)           # 隐式等待

# 6. 复用浏览器实例（不要每次创建新的）
# 在爬虫循环外创建 driver，循环内复用

# 7. 关闭不需要的标签页
# driver.close() 关闭当前标签页
# driver.quit() 关闭整个浏览器
```

**Playwright 性能优势：**

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)

    # 浏览器上下文隔离（比新开浏览器更快）
    context = browser.new_context(
        viewport={'width': 1920, 'height': 1080},
        ignore_https_errors=True,
    )

    # 多页面并发（共享一个浏览器实例）
    pages = [context.new_page() for _ in range(5)]

    # 每个页面独立爬取
    for page in pages:
        page.goto('https://example.com', wait_until='domcontentloaded')
        # wait_until: 'load' | 'domcontentloaded' | 'networkidle' | 'commit'

    browser.close()
```

---

## 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 元素找不到报 NoSuchElementException | 元素未加载或选择器错误 | 添加 WebDriverWait 显式等待 |
| ElementNotInteractableException | 元素被遮挡或不可见 | 滚动到元素位置，或用 JS 点击 |
| StaleElementReferenceException | 元素已从 DOM 中移除 | 重新查找元素，添加重试逻辑 |
| 超时 TimeoutException | 页面加载慢或条件未满足 | 增大等待时间，检查等待条件 |
| Chrome 版本不匹配 | chromedriver 版本与 Chrome 不一致 | Selenium 4.6+ 自动管理驱动 |
| 内存泄漏 | 大量页面未关闭 | 及时关闭标签页，定期重启浏览器 |
| 被网站检测为机器人 | navigator.webdriver 为 true | 执行 CDP 命令隐藏 webdriver 标记 |
| 无头模式截图不全 | 默认视口太小 | 设置 `--window-size=1920,1080` |

---

## 面试题

**Q1: Selenium 的显式等待和隐式等待有什么区别？**

> 隐式等待（`implicitly_wait`）：全局设置，对 `find_element` / `find_elements` 生效，在指定时间内轮询查找元素，找到立即返回，超时抛 NoSuchElementException。缺点：(1) 只能等元素出现在 DOM 中，不能等可见/可点击等条件；(2) 全局生效，无法针对特定元素设不同超时；(3) 和显式等待混用时行为不可预测。显式等待（`WebDriverWait` + `expected_conditions`）：针对特定元素和条件等待，可精确控制等待策略（可见、可点击、文本出现、元素消失等）。优点：(1) 条件丰富（20+ 内置条件）；(2) 可自定义等待条件；(3) 不影响其他元素查找。最佳实践：只用显式等待，不用隐式等待，两者混用会导致不可预测的等待时间。

**Q2: Selenium 和 Playwright 的核心区别是什么？如何选择？**

> (1) **协议**——Selenium 用 WebDriver HTTP 协议（每次操作是一个 HTTP 请求），Playwright 用 CDP/WebSocket（持久连接，延迟更低）；(2) **自动等待**——Playwright 内置自动等待（`page.click()` 自动等元素可点击），Selenium 需手动 WebDriverWait；(3) **多浏览器**——Selenium 支持更多浏览器（Safari/旧版），Playwright 仅 Chromium/Firefox/WebKit；(4) **网络控制**——Playwright 原生支持请求拦截、Mock 响应、修改请求头，Selenium 4+ 有限支持；(5) **并行**——Playwright 原生支持多浏览器上下文并行，Selenium 需要 Grid。选择：新项目优先 Playwright（API 更好、速度更快），已有 Selenium 测试套件继续用 Selenium，需要 Safari 支持用 Selenium。

**Q3: 如何处理无限滚动页面的爬取？**

> 核心思路：循环滚动 + 检测新内容 + 停止条件。步骤：(1) 记录当前页面高度 `scrollHeight`；(2) 执行 `window.scrollTo(0, document.body.scrollHeight)` 滚动到底部；(3) 等待新内容加载（`time.sleep` 或等待新元素出现）；(4) 提取当前可见的所有目标元素，与已采集集合去重；(5) 检查新的 `scrollHeight` 是否变化——未变化说明到底了；(6) 重复直到到底或达到最大滚动次数。注意事项：(1) 滚动间隔不要太短（触发限流）；(2) 某些页面用"加载更多"按钮而非自动滚动，需点击按钮；(3) 大量 DOM 元素会导致浏览器变慢，可定期清理已处理的元素。

**Q4: 如何绕过网站的 Selenium 检测？**

> 网站检测 Selenium 的方式：(1) `navigator.webdriver` 属性——默认为 true，Selenium 控制的浏览器会设置此属性；(2) Chrome CDP 命令检测——`window.chrome` 对象缺失或异常；(3) 浏览器指纹——plugins、languages、WebGL 等指纹与正常浏览器不同；(4) 行为分析——点击速度、鼠标轨迹异常。绕过方法：(1) 执行 CDP 命令移除 webdriver 标记：`driver.execute_cdp_cmd('Page.addScriptToEvaluateOnNewDocument', {'source': 'Object.defineProperty(navigator, "webdriver", {get: () => undefined})'})`；(2) 设置 `excludeSwitches: ['enable-automation']` 和 `useAutomationExtension: False`；(3) 使用 undetected-chromedriver 库（自动绕过检测）；(4) 使用 Playwright（默认不做 WebDriver 检测标记）。最有效：undetected-chromedriver 或 Playwright。

**Q5: Selenium 的 StaleElementReferenceException 是什么？如何解决？**

> 当代码持有对某个 WebElement 的引用，但该元素已从 DOM 中被移除或重新渲染时，对该元素的操作会抛出 StaleElementReferenceException。常见场景：(1) JS 动态更新了页面（如 AJAX 刷新列表），旧引用失效；(2) 页面导航后旧元素不存在了；(3) 元素被 React/Vue 重新渲染（同一个视觉位置的元素，DOM 已替换）。解决方案：(1) **重新查找**——捕获异常后在 except 块中重新 `find_element`；(2) **重试装饰器**——包装操作在重试循环中，最多重试 3 次；(3) **等待稳定**——在操作前等待 DOM 变化完成（如等待 loading 消失）；(4) **减少引用持有时间**——找到元素后立即提取信息，不要存着后面用。

**Q6: 如何在无界面服务器上运行 Selenium？**

> 两种方案：(1) **Headless Chrome**——`options.add_argument('--headless=new')`，无需显示器，最简单，但某些网站检测无头模式；(2) **Xvfb（虚拟帧缓冲）**——Linux 上用 Xvfb 模拟显示器，Selenium 以有头模式运行但看不到界面。Docker 部署：(1) 使用 `selenium/standalone-chrome` 官方镜像（内置 Chrome + chromedriver + Xvfb）；(2) 连接方式：`webdriver.Remote(command_executor='http://selenium:4444/wd/hub', options=options)`；(3) 或直接在应用镜像中安装 Chrome + chromedriver。推荐：简单场景用 Headless Chrome，需要完全模拟浏览器环境用 Xvfb + Docker。

**Q7: Playwright 的自动等待机制是什么？和 Selenium 的等待有什么区别？**

> Playwright 的自动等待：每个操作（`click()`、`fill()`、`hover()` 等）在执行前自动执行一系列可操作性检查——元素已附加到 DOM、可见、稳定（不在动画中）、可接收事件（不被遮挡）、启用。所有检查通过才执行操作，否则在超时时间内重试。区别：(1) **默认行为**——Selenium 的 `click()` 立即执行，元素不可操作直接报错；Playwright 的 `click()` 自动等到可操作；(2) **等待范围**——Selenium 只等元素出现/可见/可点击三个条件；Playwright 检查 6 个条件（附加、可见、稳定、可接收事件、启用、编辑性）；(3) **代码量**——Selenium 每次操作前需要写 WebDriverWait，Playwright 直接调用操作方法即可；(4) **可靠性**——Playwright 的自动等待大幅减少了 flaky 测试（时序相关的不稳定测试）。

**Q8: 如何用 Selenium/Playwright 实现登录后爬取？**

> 核心步骤：(1) **导航到登录页**——`driver.get(login_url)`；(2) **填写表单**——定位用户名/密码输入框，`send_keys()` 输入；(3) **提交登录**——点击登录按钮；(4) **验证登录成功**——等待 URL 变化或特定元素出现（如用户头像、欢迎文本）；(5) **访问目标页面**——登录态通过 Cookie 保持，后续请求自动携带。注意事项：(1) **等待策略**——登录后必须显式等待页面跳转完成，不能假设立即完成；(2) **Cookie 持久化**——保存登录 Cookie 到文件，下次运行直接加载，避免重复登录（注意 Cookie 有效期）；(3) **验证码**——登录页有验证码时最麻烦，方案：OCR 识别、打码平台 API、手动输入、Cookie 复用；(4) **双因素认证**——需要 TOTP 时，可用 `pyotp` 库生成验证码；(5) **Playwright 更简单**——`storage_state` 方法可保存和恢复整个浏览器状态（含 Cookie + localStorage）。

---

**相关链接：** [[Requests与HTTP客户端]] [[BeautifulSoup与HTML解析]] [[Scrapy框架]] [[反爬策略与应对]] [[异步编程]]
