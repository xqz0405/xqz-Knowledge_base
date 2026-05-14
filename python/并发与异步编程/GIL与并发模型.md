---
tags:
  - Python
  - GIL
  - 并发
  - 多线程
  - 多进程
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# GIL与并发模型

## What — 是什么

> GIL（Global Interpreter Lock，全局解释器锁）是 CPython 解释器中的一把互斥锁，确保同一时刻只有一个线程执行 Python 字节码。它是 Python 并发模型的核心约束，深刻影响了多线程、多进程和协程的选择策略。

**核心概念：**

- **GIL**：CPython 进程中的全局互斥锁，保护解释器内部数据结构（引用计数等）的线程安全
- **并发模型三剑客**：多线程（threading）、多进程（multiprocessing）、协程（asyncio）
- **CPU 密集 vs IO 密集**：任务类型决定了并发模型的选择

**关键特性：**

- GIL 仅存在于 CPython 实现，Jython、IronPython 无此限制
- GIL 在 IO 操作、时间片到期时主动释放，不是永远持有
- Python 3.2+ 采用 Antoine Pitrou 的 GIL 实现，改善了 IO 密集场景的公平性
- C 扩展可以在执行计算时释放 GIL（`Py_BEGIN_ALLOW_THREADS`）

**运行机制：**

- **内存模型**：Python 对象的引用计数是线程不安全的，GIL 保护了引用计数的原子性更新
- **执行模型**：线程获取 GIL → 执行字节码 → 检查间隔（sys.getcheckinterval）→ 释放 GIL → 其他线程竞争
- **GIL 切换机制**：Python 3.2+ 使用超时机制（默认 5ms），持有 GIL 的线程在超时后设置 `gil_drop_request` 标志，主动让出
- **协程与 GIL**：协程在单线程内切换，不涉及 GIL 竞争，因此异步 IO 不受 GIL 影响

**类型系统：**

- `threading.Thread`：操作系统级线程，受 GIL 约束
- `multiprocessing.Process`：独立进程，各自持有 GIL，真正并行
- `asyncio.Task`：单线程协程，通过事件循环调度，不涉及 GIL

## Why — 为什么

**适用场景：**

- IO 密集型任务（网络请求、文件读写、数据库查询）→ 多线程或协程
- CPU 密集型任务（数值计算、图像处理、加密运算）→ 多进程
- 混合型任务 → 多进程 + 协程组合

**并发模型对比：**

| 维度 | 多线程 threading | 多进程 multiprocessing | 协程 asyncio |
|------|-----------------|----------------------|-------------|
| 并行能力 | 伪并行（受 GIL 限制） | 真并行（多核利用） | 伪并行（单线程） |
| 内存开销 | 低（共享地址空间） | 高（进程独立空间） | 极低（用户态切换） |
| 切换开销 | 中（内核态上下文切换） | 高（进程切换） | 低（用户态协程切换） |
| 数据共享 | 天然共享，需加锁 | 需 IPC，天然隔离 | 天然共享（单线程） |
| 适合任务 | IO 密集 | CPU 密集 | 高并发 IO |
| 编程复杂度 | 中（锁、竞态） | 中（IPC 序列化） | 高（回调/Promise） |
| GIL 影响 | 受限 | 不受限 | 不受限 |

**GIL 存在的原因：**

- ✅ 保护引用计数的线程安全，避免给每个对象加细粒度锁带来的性能损失
- ✅ 简化 C 扩展的开发，扩展作者无需考虑线程安全
- ✅ 单线程性能优于细粒度锁方案（早期 Greg Stein 的实验证明）
- ❌ 多核 CPU 无法用于 CPU 密集型 Python 线程
- ❌ 给 Python 初学者造成并发认知误区

**优缺点：**

- ✅ 优点：
  - 简化解释器实现，C 扩展编写容易
  - 单线程场景下无锁竞争开销
  - IO 密集型场景多线程仍有效
- ❌ 缺点：
  - CPU 密集型多线程无法利用多核
  - 多线程编程仍需处理竞态条件（GIL 不保护业务逻辑）
  - 线程切换有开销但无并行收益（CPU 密集场景）

## How — 怎么用

### 快速上手：验证 GIL 的存在

```python
import threading
import time

def cpu_work():
    """CPU 密集任务：计算素数"""
    count = 0
    for i in range(2, 5000000):
        for j in range(2, int(i ** 0.5) + 1):
            if i % j == 0:
                break
        else:
            count += 1
    return count

# 单线程
start = time.time()
cpu_work()
cpu_work()
print(f"单线程: {time.time() - start:.2f}s")

# 双线程（不会加速，反而可能更慢）
start = time.time()
t1 = threading.Thread(target=cpu_work)
t2 = threading.Thread(target=cpu_work)
t1.start(); t2.start()
t1.join(); t2.join()
print(f"双线程: {time.time() - start:.2f}s")

# 双进程（真正并行加速）
from multiprocessing import Process
start = time.time()
p1 = Process(target=cpu_work)
p2 = Process(target=cpu_work)
p1.start(); p2.start()
p1.join(); p2.join()
print(f"双进程: {time.time() - start:.2f}s")
```

### 代码示例1：IO 密集型 — 多线程 vs 协程

```python
import threading
import asyncio
import time
import urllib.request

URLS = [
    "https://httpbin.org/delay/1",
    "https://httpbin.org/delay/1",
    "https://httpbin.org/delay/1",
    "https://httpbin.org/delay/1",
    "https://httpbin.org/delay/1",
]

# 多线程方案
def fetch_url(url):
    with urllib.request.urlopen(url) as resp:
        return resp.read()

def io_threading():
    start = time.time()
    threads = [threading.Thread(target=fetch_url, args=(url,)) for url in URLS]
    for t in threads:
        t.start()
    for t in threads:
        t.join()
    print(f"多线程: {time.time() - start:.2f}s")

# 协程方案
async def async_fetch(session, url):
    async with session.get(url) as resp:
        return await resp.read()

async def io_asyncio():
    import aiohttp
    start = time.time()
    async with aiohttp.ClientSession() as session:
        tasks = [async_fetch(session, url) for url in URLS]
        await asyncio.gather(*tasks)
    print(f"协程: {time.time() - start:.2f}s")

# 运行
io_threading()
asyncio.run(io_asyncio())
```

### 代码示例2：多进程实战 — 进程池与共享状态

```python
from multiprocessing import Pool, Manager, cpu_count
import time

def square(x):
    return x * x

def worker_with_dict(shared_dict, key, value):
    """多进程写入共享字典"""
    shared_dict[key] = value

if __name__ == "__main__":
    # 进程池 map
    print(f"CPU 核心数: {cpu_count()}")
    with Pool(processes=4) as pool:
        results = pool.map(square, range(100))
        print(f"结果: {results[:10]}...")

    # 共享状态
    with Manager() as manager:
        shared_dict = manager.dict()
        from multiprocessing import Process
        processes = [
            Process(target=worker_with_dict, args=(shared_dict, i, i*i))
            for i in range(5)
        ]
        for p in processes:
            p.start()
        for p in processes:
            p.join()
        print(f"共享字典: {dict(shared_dict)}")
```

### 代码示例3：混合模型 — 多进程 + 协程

```python
import asyncio
from concurrent.futures import ProcessPoolExecutor

async def io_task(url):
    """模拟异步 IO"""
    await asyncio.sleep(0.5)
    return f"data from {url}"

def cpu_task(data):
    """模拟 CPU 计算"""
    result = sum(i * i for i in range(len(data) * 100000))
    return result

async def hybrid_model():
    """多进程 + 协程混合模型"""
    urls = [f"http://api-{i}" for i in range(8)]

    # 协程并发做 IO
    io_results = await asyncio.gather(*[io_task(url) for url in urls])
    print(f"IO 完成: {len(io_results)} 条")

    # 进程池做 CPU 计算
    loop = asyncio.get_event_loop()
    with ProcessPoolExecutor(max_workers=4) as pool:
        cpu_results = await asyncio.gather(*[
            loop.run_in_executor(pool, cpu_task, data)
            for data in io_results
        ])
    print(f"计算完成: {cpu_results[:3]}...")

asyncio.run(hybrid_model())
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 多线程 CPU 任务更慢 | GIL 导致线程切换开销 + 无法并行 | 改用 `multiprocessing` |
| 多进程启动慢 | 进程创建 + 解释器初始化开销 | 使用进程池 `Pool` 复用进程 |
| 进程间对象传递报错 | 对象必须可 pickle 序列化 | 只传基本类型，用 `Manager` 共享 |
| `daemon` 进程数据丢失 | 主进程退出时守护进程被强制终止 | 用 `join()` 等待或设 `daemon=False` |
| 协程中调用阻塞函数 | 阻塞整个事件循环 | 用 `run_in_executor` 转到线程池 |
| 多进程在 Windows 报错 | Windows 用 spawn 方式，未保护入口 | 加 `if __name__ == "__main__":` |
| 线程安全问题 | GIL 不保护业务逻辑，仅保护字节码执行 | 用 `Lock`/`RLock` 保护共享状态 |

### 最佳实践

- IO 密集用协程（首选）或多线程，CPU 密集用多进程
- 优先使用 `concurrent.futures` 的高级接口（`ThreadPoolExecutor`/`ProcessPoolExecutor`）
- 多进程在 Windows 必须保护 `__main__` 入口
- 协程中不要直接调用阻塞 IO，用 `run_in_executor` 包装
- 进程间通信优先用 `Queue`/`Pipe`，避免大量数据序列化
- 用 `cpu_count()` 动态决定进程池大小
- 监控 GIL 争用：`python -X gil=0`（Python 3.13+ 实验性 free-threaded 模式）

## 面试题

**Q1: 什么是 GIL？为什么 CPython 需要 GIL？**
> GIL 是 CPython 中的全局互斥锁，确保同一时刻只有一个线程执行 Python 字节码。存在的原因是 CPython 的内存管理（引用计数）不是线程安全的，如果移除 GIL，需要给每个对象加细粒度锁，在单线程场景下性能反而更差。早期实验（Greg Stein, 1999）证明了这一点。

**Q2: GIL 对 CPU 密集型和 IO 密集型任务的影响有何不同？**
> CPU 密集型任务持续执行字节码，线程持有 GIL 时间长，多线程无法并行，反而因切换开销更慢。IO 密集型任务在等待 IO 时主动释放 GIL，其他线程可以执行，因此多线程能有效提升吞吐。协程在单线程内通过事件循环调度，不涉及 GIL 竞争，是高并发 IO 的最佳选择。

**Q3: 多进程为什么能绕过 GIL？有什么代价？**
> 每个进程有独立的 Python 解释器实例和 GIL，所以多进程可以真正并行执行。代价是：进程创建开销大、内存占用高（各自独立地址空间）、进程间通信需要序列化（pickle）导致数据传递受限且慢、共享状态需要 `Manager` 等特殊机制。

**Q4: Python 3.2 的 GIL 改进解决了什么问题？**
> Python 3.2 之前，IO 密集线程可能被 CPU 密集线程"饿死"——CPU 线程不断 reacquire GIL。3.2 的 Antoine Pitrou 实现改为：等待 GIL 的线程设置超时（默认 5ms），持有者超时后强制让出，等待线程优先获取。这改善了 IO 线程的响应性，但 CPU 密集多线程仍无法并行。

**Q5: 在协程中如何调用阻塞的同步函数？**
> 使用 `loop.run_in_executor(None, func, *args)` 将阻塞函数提交到线程池执行，返回一个可 await 的 Future。这样阻塞函数在线程中执行（线程在 IO 等待时释放 GIL），不会阻塞事件循环。也可以指定 `ProcessPoolExecutor` 来执行 CPU 密集型阻塞函数。

**Q6: 多线程中 GIL 是否意味着不需要加锁？**
> 不是。GIL 只保证字节码执行的原子性，不保护业务逻辑。例如 `x += 1` 编译为多条字节码（LOAD → INPLACE_ADD → STORE），线程可能在两条字节码之间被切换。因此多线程修改共享数据仍需要 `threading.Lock` 保护。

**Q7: 如何在 C 扩展中释放 GIL？为什么这样做？**
> 使用 `Py_BEGIN_ALLOW_THREADS` 和 `Py_END_ALLOW_THREADS` 宏包裹不需要访问 Python 对象的计算代码。这样 C 代码执行时其他线程可以获取 GIL 并行执行，完成后重新获取 GIL。NumPy 的数值计算就是这样实现的，因此 NumPy 多线程可以并行。

**Q8: Python 3.13 的 free-threaded 模式（PEP 703）是什么？**
> PEP 703 提议在 CPython 中移除 GIL，使 Python 支持真正的多线程并行。Python 3.13 以实验性方式引入了 free-threaded 构建（`python3.13t`），通过给每个对象添加细粒度锁、改引用计数为 biased reference counting 等方式实现。目前仍有性能开销（单线程约慢 5-10%），生态兼容性尚在推进中，预计 3.14+ 逐步稳定。

---

**相关链接：**
- [[异步编程]]
- [[类型系统与类型提示]]
- [[错误处理与调试]]
- [Python GIL 官方文档](https://docs.python.org/3/glossary.html#term-global-interpreter-lock)
