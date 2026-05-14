---
tags:
  - Python
  - 并发与异步编程
  - asyncio
  - 协程
  - 异步
date: 2026-05-14
status: 已完成
difficulty: 中高
---

# asyncio深入与协程模式

## What — 是什么

> asyncio 是 Python 的异步 I/O 框架，基于事件循环（Event Loop）调度协程（Coroutine），在单线程内实现高并发 I/O 操作。深入理解 asyncio 需要掌握事件循环机制、Task 调度、异步上下文管理器、异步迭代器等高级特性。

**核心概念：**

- **事件循环（Event Loop）**：asyncio 的心脏，负责调度协程、处理 I/O 回调、管理定时器，是整个异步运行时的核心
- **Task**：对协程的包装，由事件循环调度执行，支持取消、回调、结果获取
- **async/await**：协程定义（`async def`）和挂起（`await`）的语法，await 将控制权交还事件循环
- **异步上下文管理器**：`async with` 支持的上下文协议（`__aenter__`/`__aexit__`），用于异步资源的获取和释放
- **异步迭代器**：`async for` 支持的迭代协议（`__aiter__`/`__anext__`），用于流式数据异步遍历

**核心架构：**

```
async def coro()          # 协程定义
    → asyncio.create_task(coro())   # 包装为 Task
    → Event Loop 调度执行            # Task 在事件循环中运行
    → await 挂起                     # 遇到 I/O 时让出控制权
    → I/O 完成回调                   # 事件循环恢复协程
    → Task 完成                      # 结果可通过 future.result() 获取
```

**关键特性：**

- `asyncio.gather()`：并发执行多个协程，等待全部完成
- `asyncio.create_task()`：立即调度协程为 Task
- `asyncio.wait()`：更灵活的等待（FIRST_COMPLETED/FIRST_EXCEPTION/ALL_COMPLETED）
- `asyncio.Queue`：协程间安全通信
- `asyncio.StreamReader/StreamWriter`：异步 TCP 通信

## Why — 为什么

**适用场景：**

- 高并发 HTTP 请求——数千个 API 调用同时等待响应
- 实时通信服务器——WebSocket、聊天服务
- 数据库连接池——异步 ORM（SQLAlchemy async、Tortoise ORM）
- 流式数据处理——大文件逐行处理、实时日志分析
- 微服务间调用——并发调用多个下游服务

**asyncio 高级特性 vs 基础用法：**

| 维度 | 基础（async/await入门） | 深入（本文覆盖） |
|------|----------------------|----------------|
| 事件循环 | `asyncio.run()` 一行搞定 | 手动管理循环、策略、调试 |
| Task | `gather()` 并发 | `create_task()`、取消、回调、超时 |
| 同步原语 | Lock/Event | Semaphore、Condition、Barrier |
| 异步协议 | async def/await | async with、async for、async generator |
| 错误处理 | try/except | TaskGroup、异常传播、取消处理 |
| 生产模式 | 简单脚本 | 连接池、限流、优雅关闭 |

## How — 怎么用

### 1. 事件循环深入

```python
import asyncio

# asyncio.run() 做了什么？
# 1. 创建新事件循环
# 2. 运行传入的协程
# 3. 关闭事件循环（取消所有未完成 Task）
# 4. 清理

# 手动管理事件循环（高级场景）
loop = asyncio.new_event_loop()
asyncio.set_event_loop(loop)
try:
    loop.run_until_complete(main())
finally:
    loop.close()

# 获取当前事件循环
loop = asyncio.get_event_loop()
loop = asyncio.get_running_loop()  # 只在协程内可用

# 事件循环策略（Windows 默认用 ProactorEventLoop，Linux 用 SelectorEventLoop）
if sys.platform == 'win32':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())

# 在事件循环中执行阻塞代码
async def main():
    loop = asyncio.get_running_loop()

    # 在线程池中执行阻塞函数（不阻塞事件循环）
    result = await loop.run_in_executor(None, blocking_function, arg1, arg2)

    # 指定线程池大小
    from concurrent.futures import ThreadPoolExecutor
    executor = ThreadPoolExecutor(max_workers=4)
    result = await loop.run_in_executor(executor, blocking_function, arg1)

    # 在进程池中执行 CPU 密集函数
    from concurrent.futures import ProcessPoolExecutor
    process_executor = ProcessPoolExecutor(max_workers=4)
    result = await loop.run_in_executor(process_executor, cpu_intensive_function, data)

# 调试模式
asyncio.run(main(), debug=True)  # 启用调试：检测阻塞调用、未 awaited 协程
```

### 2. Task 高级操作

```python
import asyncio

# create_task — 立即调度（不需要 await 才开始执行）
async def fetch(url):
    await asyncio.sleep(1)
    return f'data from {url}'

async def main():
    # ✅ 立即创建 Task 并调度
    task = asyncio.create_task(fetch('https://api.example.com'))
    # 此时 fetch 已经在后台运行
    # 可以做其他事情
    print('Task 已提交，做其他事情...')
    result = await task  # 需要结果时再 await
    print(result)

# Task 取消
async def cancellable_task():
    try:
        await asyncio.sleep(10)
    except asyncio.CancelledError:
        print('Task 被取消，清理资源...')
        raise  # 重新抛出，让调用者知道取消了

async def main():
    task = asyncio.create_task(cancellable_task())
    await asyncio.sleep(1)
    task.cancel()  # 请求取消
    try:
        await task
    except asyncio.CancelledError:
        print('已确认取消')

# Task 回调
async def main():
    task = asyncio.create_task(fetch('https://api.example.com'))

    def on_done(t):
        """Task 完成时回调（在事件循环线程中执行）"""
        try:
            result = t.result()
            print(f'回调收到: {result}')
        except Exception as e:
            print(f'回调异常: {e}')

    task.add_done_callback(on_done)
    await task

# 超时控制
async def main():
    # 方式一：asyncio.wait_for
    try:
        result = await asyncio.wait_for(
            fetch('https://slow.example.com'),
            timeout=3.0
        )
    except asyncio.TimeoutError:
        print('请求超时')

    # 方式二：asyncio.timeout（Python 3.11+）
    async with asyncio.timeout(3.0):
        result = await fetch('https://slow.example.com')

# asyncio.shield — 保护 Task 不被外层取消
async def main():
    task = asyncio.create_task(fetch('https://api.example.com'))
    try:
        # shield 内的 task 不会被外层 cancel 取消
        result = await asyncio.shield(task)
    except asyncio.CancelledError:
        # 即使 main 被取消，task 仍继续执行
        result = await task  # 等待实际完成
```

### 3. 并发模式

```python
import asyncio

# gather — 并发执行，等待全部完成
async def main():
    results = await asyncio.gather(
        fetch('https://api1.example.com'),
        fetch('https://api2.example.com'),
        fetch('https://api3.example.com'),
        return_exceptions=True,  # 异常作为结果返回，不抛出
    )
    for result in results:
        if isinstance(result, Exception):
            print(f'失败: {result}')
        else:
            print(f'成功: {result}')

# TaskGroup — Python 3.11+，更安全的并发（任一失败则全部取消）
async def main():
    async with asyncio.TaskGroup() as tg:
        task1 = tg.create_task(fetch('https://api1.example.com'))
        task2 = tg.create_task(fetch('https://api2.example.com'))
    # 退出 with 块时，所有 Task 都已完成
    # 如果任一 Task 抛异常，其余自动取消，异常在 with 块抛出
    print(task1.result(), task2.result())

# wait — 更灵活的等待策略
async def main():
    tasks = [asyncio.create_task(fetch(f'https://api{i}.example.com'))
             for i in range(10)]

    # 等待策略
    done, pending = await asyncio.wait(
        tasks,
        timeout=5.0,                      # 总超时
        return_when=asyncio.FIRST_COMPLETED,  # 任一完成即返回
        # return_when=asyncio.FIRST_EXCEPTION,  # 第一个异常时返回
        # return_when=asyncio.ALL_COMPLETED,    # 全部完成（默认）
    )

    print(f'完成: {len(done)}, 未完成: {len(pending)}')
    # 取消未完成的
    for task in pending:
        task.cancel()

# as_completed — 按完成顺序获取结果
async def main():
    tasks = [asyncio.create_task(fetch(f'https://api{i}.example.com'))
             for i in range(10)]

    for coro in asyncio.as_completed(tasks, timeout=8.0):
        result = await coro
        print(f'最早完成: {result}')
```

### 4. 异步同步原语

```python
import asyncio

# Lock — 异步互斥锁
lock = asyncio.Lock()

async def safe_write(data):
    async with lock:
        await write_to_shared_resource(data)

# Semaphore — 限制并发数
semaphore = asyncio.Semaphore(5)  # 最多 5 个并发

async def limited_fetch(url):
    async with semaphore:
        return await fetch(url)

async def main():
    urls = [f'https://api.example.com/{i}' for i in range(50)]
    tasks = [limited_fetch(url) for url in urls]
    results = await asyncio.gather(*tasks)

# Event — 异步事件通知
event = asyncio.Event()

async def waiter():
    print('等待事件...')
    await event.wait()
    print('事件触发！')

async def notifier():
    await asyncio.sleep(2)
    event.set()

# Condition — 异步条件变量
condition = asyncio.Condition()
items = []

async def producer():
    for i in range(5):
        async with condition:
            items.append(i)
            condition.notify()

async def consumer():
    async with condition:
        while not items:
            await condition.wait()
        return items.pop(0)

# Queue — 协程间安全通信
queue = asyncio.Queue(maxsize=10)  # 有界队列

async def producer():
    for i in range(20):
        await queue.put(i)  # 队列满时等待
        print(f'生产: {i}')

async def consumer():
    while True:
        item = await queue.get()  # 队列空时等待
        try:
            print(f'消费: {item}')
        finally:
            queue.task_done()  # 标记处理完成

async def main():
    # 启动生产者和消费者
    producers = [asyncio.create_task(producer()) for _ in range(2)]
    consumers = [asyncio.create_task(consumer()) for _ in range(3)]

    # 等待生产完成
    await asyncio.gather(*producers)
    # 等待队列处理完
    await queue.join()
    # 取消消费者（它们在无限循环）
    for c in consumers:
        c.cancel()
```

### 5. 异步协议实现

```python
import asyncio

# 异步上下文管理器
class AsyncDBConnection:
    async def __aenter__(self):
        self.conn = await create_connection()
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.conn.close()
        if exc_type:
            await self.conn.rollback()
        else:
            await self.conn.commit()

# 使用
async def main():
    async with AsyncDBConnection() as db:
        await db.execute('SELECT 1')

# 异步迭代器
class AsyncLineReader:
    """异步逐行读取文件"""
    def __init__(self, filepath):
        self.filepath = filepath

    async def __aiter__(self):
        self.file = await aiofiles.open(self.filepath)
        return self

    async def __anext__(self):
        line = await self.file.readline()
        if not line:
            await self.file.close()
            raise StopAsyncIteration
        return line.strip()

# 使用
async def main():
    async for line in AsyncLineReader('large_file.txt'):
        process(line)

# 异步生成器
async def async_range(start, end, delay=0.1):
    """异步生成数字序列"""
    for i in range(start, end):
        await asyncio.sleep(delay)
        yield i

async def main():
    async for num in async_range(0, 10):
        print(num)
```

### 6. 实战：异步 HTTP 并发客户端

```python
"""生产级异步并发客户端，支持限流、重试、连接池"""
import asyncio
import aiohttp
import time
from typing import List, Dict, Any


class AsyncClient:
    """异步并发 HTTP 客户端"""

    def __init__(self, max_concurrency=10, rate_limit=20, timeout=30):
        self.max_concurrency = max_concurrency
        self.rate_limit = rate_limit
        self.timeout = aiohttp.ClientTimeout(total=timeout)
        self.semaphore = asyncio.Semaphore(max_concurrency)

    async def fetch_one(self, session: aiohttp.ClientSession, url: str) -> Dict:
        """单个请求，带限流和重试"""
        async with self.semaphore:
            for attempt in range(3):
                try:
                    async with session.get(url) as response:
                        response.raise_for_status()
                        return await response.json()
                except (aiohttp.ClientError, asyncio.TimeoutError) as e:
                    if attempt == 2:
                        return {'error': str(e), 'url': url}
                    await asyncio.sleep(1 * (attempt + 1))  # 退避
            return {'error': 'max retries', 'url': url}

    async def fetch_many(self, urls: List[str]) -> List[Dict]:
        """并发请求多个 URL"""
        connector = aiohttp.TCPConnector(
            limit=self.max_concurrency,  # 连接池大小
            limit_per_host=5,            # 单主机连接数
            ttl_dns_cache=300,           # DNS 缓存 5 分钟
        )

        async with aiohttp.ClientSession(
            connector=connector,
            timeout=self.timeout,
        ) as session:
            tasks = [self.fetch_one(session, url) for url in urls]
            results = await asyncio.gather(*tasks, return_exceptions=True)

            success = []
            failures = []
            for url, result in zip(urls, results):
                if isinstance(result, Exception):
                    failures.append({'url': url, 'error': str(result)})
                elif 'error' in result:
                    failures.append(result)
                else:
                    success.append(result)

            return success, failures


# 使用
async def main():
    client = AsyncClient(max_concurrency=10, rate_limit=20)
    urls = [f'https://httpbin.org/get?id={i}' for i in range(50)]

    start = time.perf_counter()
    success, failures = await client.fetch_many(urls)
    elapsed = time.perf_counter() - start

    print(f'完成: {len(success)} 成功, {len(failures)} 失败, 耗时 {elapsed:.2f}s')

asyncio.run(main())
```

### 7. 优雅关闭

```python
import asyncio
import signal

class GracefulShutdown:
    """优雅关闭：处理 SIGINT/SIGTERM，等待进行中的任务完成"""

    def __init__(self):
        self.shutdown_event = asyncio.Event()
        self.tasks: set = set()

    def _handle_signal(self):
        """信号处理：触发关闭"""
        self.shutdown_event.set()

    async def run(self, main_coro):
        loop = asyncio.get_running_loop()

        # 注册信号处理
        for sig in (signal.SIGINT, signal.SIGTERM):
            loop.add_signal_handler(sig, self._handle_signal)

        # 运行主协程
        main_task = asyncio.create_task(main_coro)
        shutdown_task = asyncio.create_task(self.shutdown_event.wait())

        # 等待主协程完成或收到关闭信号
        done, pending = await asyncio.wait(
            [main_task, shutdown_task],
            return_when=asyncio.FIRST_COMPLETED,
        )

        if main_task not in done:
            print('收到关闭信号，等待任务完成...')
            main_task.cancel()
            try:
                await main_task
            except asyncio.CancelledError:
                pass

        # 取消所有未完成 Task
        for task in asyncio.all_tasks(loop):
            if task is not asyncio.current_task():
                task.cancel()

        # 等待所有 Task 完成
        await asyncio.gather(*asyncio.all_tasks(loop), return_exceptions=True)

# 使用
async def long_running():
    while True:
        print('工作中...')
        await asyncio.sleep(1)

shutdown = GracefulShutdown()
asyncio.run(shutdown.run(long_running()))
```

---

## 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 协程没有执行 | 只写 `coro()` 不 await/create_task | `await coro()` 或 `asyncio.create_task(coro())` |
| 阻塞事件循环 | 在协程中调用同步阻塞函数（requests、time.sleep） | 用 `run_in_executor` 包装，或用异步版本 |
| Task 取消不生效 | 协程内部捕获了 CancelledError 但没重新抛出 | `except CancelledError: cleanup(); raise` |
| gather 一个失败全部失败 | gather 默认传播异常 | `return_exceptions=True` 让异常作为结果 |
| 内存泄漏 | 大量 Task 未完成未取消 | 用 `TaskGroup` 或在 finally 中取消所有 Task |
| 调试困难 | 异步调用栈不直观 | `asyncio.run(main(), debug=True)` + `logging` |
| Windows 事件循环异常 | Windows 默认 ProactorEventLoop 不支持某些操作 | `asyncio.WindowsSelectorEventLoopPolicy()` |
| 连接池耗尽 | 并发数 > 连接池大小 | 调整 `TCPConnector(limit=N)` + `Semaphore` |

---

## 面试题

**Q1: asyncio 的事件循环是什么？它是如何调度协程的？**

> 事件循环是 asyncio 的核心调度器，本质是一个单线程的无限循环：检查就绪的 I/O 事件 → 执行对应的回调/恢复协程 → 处理定时器 → 重复。调度机制：(1) `asyncio.create_task()` 将协程包装为 Task 加入就绪队列；(2) 事件循环从队列取出 Task 执行协程；(3) 协程遇到 `await` 时，挂起当前协程，注册 I/O 回调，控制权返回事件循环；(4) I/O 完成后，事件循环恢复对应协程从 await 处继续执行。关键：整个调度是协作式的——协程必须主动 await 才能切换，不会被打断（区别于线程的抢占式调度）。

**Q2: asyncio.create_task 和直接 await 有什么区别？**

> `await coro()`：立即执行协程，当前协程挂起等待结果，是串行执行；`asyncio.create_task(coro())`：将协程包装为 Task 立即提交到事件循环调度，当前协程继续执行（不等待），Task 在后台并发运行。核心区别：await 是"等我做完"，create_task 是"你先做着，我继续"。典型用法：需要并发时先 `create_task` 所有任务，再 `await` 收集结果；需要串行时直接 `await`。注意：`create_task` 后不 await 就丢弃了 Task 引用，Task 可能被垃圾回收导致警告——应始终保存引用或在 `TaskGroup` 中使用。

**Q3: asyncio.gather、TaskGroup、wait、as_completed 各自的使用场景？**

> (1) **gather**——并发执行多个协程，等待全部完成，保持结果顺序。适合"并发请求 N 个 API，都要结果"的场景。`return_exceptions=True` 可避免一个失败导致全部中断；(2) **TaskGroup**（3.11+）——类似 gather 但更安全，任一 Task 失败自动取消其余 Task，异常在 `async with` 块抛出。适合"要么全部成功，要么全部放弃"的场景；(3) **wait**——最灵活，可指定等待策略（任一完成/首个异常/全部完成），返回 done 和 pending 两个集合。适合"等最快的几个"或需要手动取消未完成任务；(4) **as_completed**——按完成顺序迭代结果，先完成先处理。适合"谁先回来谁先处理"的流式场景。

**Q4: 如何在 asyncio 中调用阻塞的同步函数？**

> 使用 `loop.run_in_executor()` 将同步函数提交到线程池或进程池执行，不阻塞事件循环。线程池（默认）：`await loop.run_in_executor(None, blocking_func, args)`，适合 I/O 阻塞函数（requests.get、subprocess 等）；进程池：`await loop.run_in_executor(ProcessPoolExecutor(), cpu_func, args)`，适合 CPU 密集函数。注意：(1) 回调函数在 executor 线程中执行，不是事件循环线程，不能直接操作 asyncio 对象；(2) 自定义 ThreadPoolExecutor 可控制线程数；(3) Python 3.9+ 的 `asyncio.to_thread()` 是 `run_in_executor` 的简化版：`await asyncio.to_thread(blocking_func, args)`。

**Q5: asyncio 的取消机制是什么？如何正确处理取消？**

> 取消流程：`task.cancel()` 设置 Task 的取消标志 → Task 在下一个 await 点抛出 `CancelledError` → 协程可以捕获并清理资源 → 应重新抛出 CancelledError 让取消传播。正确处理模式：(1) 捕获 CancelledError 做清理，然后 raise；(2) 使用 `try/finally` 确保资源释放；(3) 使用 `async with` 异步上下文管理器自动清理。错误模式：(1) 吞掉 CancelledError 不 re-raise——导致 task.cancel() 后 await task 不会抛异常；(2) 在 finally 中再次 await——可能死循环。Python 3.11+ 的 TaskGroup 自动取消机制更安全。

**Q6: asyncio.Queue 和 multiprocessing.Queue 有什么区别？**

> (1) **线程模型**——asyncio.Queue 在单线程事件循环中使用，通过 await 实现协程间通信；multiprocessing.Queue 用于多进程间通信，通过操作系统管道和锁实现；(2) **性能**——asyncio.Queue 是纯内存操作，极快；multiprocessing.Queue 需要序列化/pickle 和 IPC，较慢；(3) **容量**——asyncio.Queue 的 `maxsize` 通过协程挂起实现背压（`put()` 在满时 await 挂起）；multiprocessing.Queue 的容量受 OS 管道缓冲区限制；(4) **安全性**——asyncio.Queue 在单线程中无需锁，天然安全；multiprocessing.Queue 有内部锁保护。选择：协程间用 asyncio.Queue，进程间用 multiprocessing.Queue，线程间用 queue.Queue。

**Q7: 什么是结构化并发？TaskGroup 相比 gather 有什么优势？**

> 结构化并发（Structured Concurrency）是指并发任务的生命周期被限定在一个明确的作用域内——进入作用域创建任务，退出作用域所有任务必须完成。类比结构化编程（不用 goto），结构化并发不允许"逃逸"的任务。TaskGroup 的优势：(1) **自动取消**——任一 Task 失败，其余自动取消，避免浪费资源；(2) **生命周期绑定**——退出 `async with` 块时，所有 Task 都已完成或取消，不存在"泄漏"的 Task；(3) **异常聚合**——多个 Task 失败时，异常通过 ExceptionGroup 聚合抛出，不会丢失任何错误信息；(4) **无引用泄漏**——TaskGroup 内部管理所有 Task 引用，不需要手动维护列表。gather 的问题：一个失败后其余仍在运行，需要手动 cancel；Task 引用需要自己保存。

**Q8: 如何调试 asyncio 程序？**

> (1) **debug 模式**——`asyncio.run(main(), debug=True)`，启用后会检测：超过 100ms 的阻塞调用、未 awaited 的协程、未关闭的资源。日志级别设为 `logging.DEBUG` 可看到调度详情；(2) **aiomonitor**——第三方库，提供类似 top 的实时监控界面，查看活跃 Task、事件循环状态；(3) **logging**——在每个协程入口/出口加日志，记录 Task 名称和耗时；(4) **超时检测**——`asyncio.wait_for` + `asyncio.timeout` 为每个操作设超时，超时即报警；(5) **Task 命名**——Python 3.11+ `asyncio.create_task(coro(), name='fetch-api')`，在日志和调试器中显示有意义的名称；(6) **异常追踪**——Task 异常不会自动打印，需要 `task.add_done_callback(lambda t: t.exception() if not t.cancelled() else None)` 或在 TaskGroup 中处理。

---

**相关链接：** [[GIL与并发模型]] [[异步编程]] [[多线程与多进程]] [[Requests与HTTP客户端]] [[WebSocket与实时通信]]
