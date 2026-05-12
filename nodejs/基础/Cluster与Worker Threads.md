---
tags:
  - Node.js
  - Cluster
  - Worker Threads
  - 多线程
date: 2026-05-12
status: 已完成
difficulty: 进阶
---

# Cluster 与 Worker Threads

## What — 是什么

> Cluster 和 Worker Threads 是 Node.js 提供的两种并行机制：Cluster 用于多进程并行处理网络请求，Worker Threads 用于多线程并行执行 CPU 密集计算。两者互补，分别解决不同层面的性能瓶颈。

**核心概念：**

- **Cluster 模块**：基于 `child_process.fork()` 创建多个工作进程共享同一端口，主进程负责监听和分发连接
- **Worker Threads**：Node.js 10.5+ 引入的多线程 API，工作线程与主线程共享进程内存，通过 `MessagePort` 通信
- **SharedArrayBuffer**：可在主线程和工作线程间共享的内存区域，零拷贝传递大数据
- **Atomics**：对 SharedArrayBuffer 的原子操作，保证多线程并发安全（add/and/compareExchange/load/store 等）
- **Round-Robin 调度**：Cluster 主进程的默认连接分发策略（除 Windows 外），轮流分配给工作进程
- **负载均衡**：Cluster 模式下请求均匀分配，Worker Threads 需手动分配任务

**关键特性：**

- Cluster 工作进程是独立进程，崩溃不影响其他进程
- Worker Threads 共享进程内存，一个线程的未捕获异常可能导致整个进程崩溃
- Cluster 适合 HTTP 服务水平扩展，Worker Threads 适合 CPU 密集计算
- SharedArrayBuffer 不需要序列化，但只支持二进制数据
- `worker_threads` 的 `transferList` 可零拷贝转移 ArrayBuffer 所有权

## Why — 为什么

**适用场景：**

- Cluster：多核 HTTP 服务（API 服务器、Web 应用）、提高吞吐量、零停机重启
- Worker Threads：图像处理、加密计算、数据压缩、大数据分析、CPU 密集型任务

**对比并行方案：**

| 维度 | Cluster | Worker Threads | child_process.fork |
|------|---------|---------------|-------------------|
| 并行类型 | 多进程 | 多线程 | 多进程 |
| 内存 | 独立（不共享） | 可共享 SharedArrayBuffer | 独立 |
| 通信方式 | IPC（JSON 序列化） | MessagePort + SharedArrayBuffer | IPC |
| 通信开销 | 高（序列化） | 低（共享内存/转移所有权） | 高 |
| 稳定性 | 高（进程隔离） | 中（共享进程，异常影响全局） | 高 |
| 内存占用 | 高（每进程 ~30MB） | 低（共享堆） | 高 |
| 创建开销 | 大（fork 进程） | 小（创建线程） | 大 |
| 适合场景 | HTTP 服务扩容 | CPU 密集计算 | 独立任务隔离 |

**优缺点：**

- ✅ 优点：
  - Cluster 让单机吞吐量随 CPU 核数线性增长
  - Worker Threads 共享内存，通信开销极低
  - SharedArrayBuffer 零拷贝传递大数据
  - 两者可组合使用：Cluster 扩进程 + Worker Threads 扩线程
- ❌ 缺点：
  - Cluster 进程间不共享状态（需 Redis 等外部存储）
  - Worker Threads 不能使用 native addon 的同步 API
  - SharedArrayBuffer 操作需 Atomics 保证原子性，编程复杂
  - 调试多线程/多进程代码比单线程困难得多

## How — 怎么用

### 快速上手

```javascript
// Cluster 快速启动
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isPrimary) {
    console.log(`主进程 ${process.pid} 启动`);
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
    cluster.on('exit', (worker) => {
        console.log(`工作进程 ${worker.process.pid} 退出，重新 fork`);
        cluster.fork();
    });
} else {
    http.createServer((req, res) => {
        res.writeHead(200);
        res.end(`进程 ${process.pid} 处理请求\n`);
    }).listen(3000);
}

// Worker Threads 快速启动
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');

if (isMainThread) {
    const worker = new Worker(__filename, {
        workerData: { iterations: 10000000 }
    });
    worker.on('message', (result) => console.log('计算结果:', result));
} else {
    let sum = 0;
    for (let i = 0; i < workerData.iterations; i++) sum += Math.sqrt(i);
    parentPort.postMessage(sum);
}
```

### 代码示例

**Cluster 主进程管理 + 优雅重启：**

```javascript
const cluster = require('cluster');
const http = require('http');
const os = require('os');

if (cluster.isPrimary) {
    const numCPUs = os.cpus().length;
    const workers = new Map();

    // 启动工作进程
    for (let i = 0; i < numCPUs; i++) {
        const worker = cluster.fork();
        workers.set(worker.process.pid, worker);
    }

    // 工作进程退出时自动重启
    cluster.on('exit', (worker, code, signal) => {
        console.log(`工作进程 ${worker.process.pid} 退出 (${signal || code})`);
        workers.delete(worker.process.pid);
        const newWorker = cluster.fork();
        workers.set(newWorker.process.pid, newWorker);
    });

    // 优雅重启：逐个替换工作进程
    function gracefulReload() {
        const pids = [...workers.keys()];
        let index = 0;

        function replaceNext() {
            if (index >= pids.length) {
                console.log('所有工作进程已替换');
                return;
            }
            const oldPid = pids[index++];
            const oldWorker = workers.get(oldPid);

            // 启动新工作进程
            const newWorker = cluster.fork();
            workers.set(newWorker.process.pid, newWorker);

            // 新工作进程就绪后关闭旧进程
            newWorker.on('listening', () => {
                console.log(`新进程 ${newWorker.process.pid} 就绪，关闭旧进程 ${oldPid}`);
                oldWorker.send('shutdown');
                setTimeout(() => oldWorker.kill(), 5000);
                replaceNext();
            });
        }
        replaceNext();
    }

    process.on('SIGUSR2', gracefulReload);
} else {
    const app = require('./app');
    const server = http.createServer(app);
    server.listen(3000);

    process.on('message', (msg) => {
        if (msg === 'shutdown') {
            server.close(() => process.exit(0));
            setTimeout(() => process.exit(1), 10000);
        }
    });
}
```

**Worker Threads + SharedArrayBuffer 共享计算：**

```javascript
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');

if (isMainThread) {
    // 使用 SharedArrayBuffer 共享大数据
    const size = 1024 * 1024 * 10; // 10MB
    const sharedBuffer = new SharedArrayBuffer(Int32Array.BYTES_PER_ELEMENT * size);
    const sharedArray = new Int32Array(sharedBuffer);

    // 初始化数据
    for (let i = 0; i < size; i++) sharedArray[i] = i;

    // 创建多个工作线程分片计算
    const numWorkers = 4;
    const chunkSize = Math.ceil(size / numWorkers);
    let completed = 0;
    let totalSum = 0;

    for (let i = 0; i < numWorkers; i++) {
        const start = i * chunkSize;
        const end = Math.min(start + chunkSize, size);

        const worker = new Worker(__filename, {
            workerData: { start, end, sharedBuffer }
        });

        worker.on('message', (partialSum) => {
            totalSum += partialSum;
            completed++;
            if (completed === numWorkers) {
                console.log(`总和: ${totalSum}`);
            }
        });
    }
} else {
    // 工作线程：计算分片求和
    const { start, end, sharedBuffer } = workerData;
    const sharedArray = new Int32Array(sharedBuffer);
    let sum = 0;
    for (let i = start; i < end; i++) {
        sum += sharedArray[i];
    }
    parentPort.postMessage(sum);
}

// 使用 transferList 零拷贝转移 ArrayBuffer
if (isMainThread) {
    const bigBuffer = new ArrayBuffer(1024 * 1024 * 100); // 100MB
    const view = new Float64Array(bigBuffer);
    for (let i = 0; i < view.length; i++) view[i] = Math.random();

    const worker = new Worker('./processor.js', {
        workerData: { buffer: bigBuffer },
        transferList: [bigBuffer] // 转移所有权，主线程不再可用
    });
}
```

**线程池封装：**

```javascript
const { Worker } = require('worker_threads');
const path = require('path');

class ThreadPool {
    constructor(workerPath, size = 4) {
        this.workerPath = workerPath;
        this.size = size;
        this.workers = [];
        this.queue = [];
        this.taskId = 0;
        this.callbacks = new Map();

        for (let i = 0; i < size; i++) {
            this._createWorker();
        }
    }

    _createWorker() {
        const worker = new Worker(this.workerPath);
        worker.busy = false;

        worker.on('message', (msg) => {
            const cb = this.callbacks.get(msg.id);
            if (cb) {
                this.callbacks.delete(msg.id);
                if (msg.error) cb.reject(new Error(msg.error));
                else cb.resolve(msg.result);
            }
            worker.busy = false;
            this._processQueue();
        });

        worker.on('error', (err) => {
            console.error('Worker error:', err);
            const idx = this.workers.indexOf(worker);
            if (idx !== -1) this.workers.splice(idx, 1);
            this._createWorker();
        });

        this.workers.push(worker);
    }

    execute(data) {
        return new Promise((resolve, reject) => {
            const id = this.taskId++;
            const idleWorker = this.workers.find(w => !w.busy);

            if (idleWorker) {
                idleWorker.busy = true;
                this.callbacks.set(id, { resolve, reject });
                idleWorker.postMessage({ id, data });
            } else {
                this.queue.push({ id, data, resolve, reject });
            }
        });
    }

    _processQueue() {
        if (this.queue.length === 0) return;
        const idleWorker = this.workers.find(w => !w.busy);
        if (!idleWorker) return;

        const task = this.queue.shift();
        idleWorker.busy = true;
        this.callbacks.set(task.id, { resolve: task.resolve, reject: task.reject });
        idleWorker.postMessage({ id: task.id, data: task.data });
    }

    close() {
        this.workers.forEach(w => w.terminate());
    }
}

// 使用示例
const pool = new ThreadPool(path.join(__dirname, 'compute-worker.js'), 4);
const result = await pool.execute({ array: [1, 2, 3, 4, 5] });
pool.close();
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Cluster 模式 Session 丢失 | 工作进程内存独立 | Session 存 Redis/外部存储 |
| Worker 线程崩溃影响主线程 | 未捕获异常未被处理 | 监听 `worker.on('error')`，worker 内 `process.on('uncaughtException')` |
| SharedArrayBuffer 数据竞争 | 多线程同时写入 | 用 `Atomics` 操作保证原子性 |
| Worker 不能使用 fs 同步 API | 某些 native addon 不兼容 | 改用异步版本或改用 child_process |
| Cluster 端口冲突 | 多个进程同时 listen | Cluster 模块自动处理（主进程 listen，分发连接） |
| transferList 后主线程不可用 | ArrayBuffer 所有权转移 | 共享用 SharedArrayBuffer，转移后不需再用 |

### 最佳实践

- HTTP 服务扩容用 Cluster（或 PM2 集群模式），CPU 计算用 Worker Threads
- Cluster 模式配合 Redis 存储共享状态
- Worker Threads 中处理 uncaughtException 防止进程崩溃
- 共享大数据用 SharedArrayBuffer + Atomics，小数据用 postMessage
- 使用线程/进程池管理并发，避免频繁创建销毁
- Cluster 优雅重启：逐个替换工作进程

## 面试题

**Q1: Cluster 的工作原理是什么？多个进程如何共享同一端口？**
> Cluster 基于 `child_process.fork()` 创建工作进程。所有工作进程通过 `cluster.fork()` 创建后，调用 `server.listen()` 时，实际上只有主进程真正监听端口，工作进程通过 IPC 通知主进程自己要监听。主进程收到连接后通过 Round-Robin（轮询）策略将连接分发给工作进程。在 Windows 上使用共享端口（SO_REUSEPORT）。工作进程间内存独立，崩溃后被主进程自动重启。

**Q2: Worker Threads 和 child_process.fork() 如何选择？**
> 选择依据：需要进程隔离用 fork（一个崩溃不影响另一个），需要共享内存/低通信开销用 Worker Threads。fork 每个进程独立 V8 堆（~30MB），Worker Threads 共享堆（内存效率高）。fork 通过 IPC JSON 序列化通信（开销大），Worker Threads 通过 MessagePort + SharedArrayBuffer 通信（开销低）。HTTP 服务水平扩展用 Cluster（基于 fork），CPU 密集计算用 Worker Threads。

**Q3: SharedArrayBuffer 如何使用？为什么需要 Atomics？**
> SharedArrayBuffer 是可在主线程和工作线程间共享的内存区域，通过 `new SharedArrayBuffer(size)` 创建，用 TypedArray 视图（如 `Int32Array`）读写。Atomics 提供原子操作保证并发安全——`Atomics.add()`/`Atomics.load()`/`Atomics.store()`/`Atomics.compareExchange()` 等操作不可分割，不会被其他线程中断。不用 Atomics 可能导致数据竞争：两个线程同时 `array[0]++`，结果可能只加了 1 而非 2。

**Q4: Cluster 模式的负载均衡策略是什么？**
> Node.js 在非 Windows 系统上默认使用 Round-Robin 策略：主进程监听端口，收到连接后按轮询顺序依次分发给工作进程。这避免了"惊群效应"（所有工作进程争抢同一连接）。Windows 上默认使用共享监听（SO_REUSEPORT），所有工作进程同时监听，由操作系统分发，可能导致不均匀。可通过 `cluster.schedulingPolicy = cluster.SCHED_RR` 强制使用 Round-Robin。

**Q5: 如何在 Worker Threads 中传递大数据？**
> 三种方式：① `postMessage` + `transferList`——将 ArrayBuffer 的所有权零拷贝转移给工作线程，主线程不再可用；② `SharedArrayBuffer`——主线程和工作线程共享同一段内存，双方都可读写，需 Atomics 保证原子性；③ `workerData`——创建 Worker 时传递的初始数据，会复制一份。推荐：需要双向访问用 SharedArrayBuffer，单向传递大数据用 transferList，小数据直接 postMessage。

**Q6: Cluster 优雅重启如何实现？**
> 优雅重启步骤：① 创建新的工作进程，等待其 `listening` 事件（表示已准备好接收请求）；② 向旧工作进程发送 `shutdown` 消息或 `SIGTERM` 信号；③ 旧工作进程停止接受新连接，完成已有请求后退出；④ 逐个替换所有工作进程。关键：始终保证部分工作进程在线处理请求，不会出现全部断开的情况。PM2 的 `reload` 命令就是这个原理。

**Q7: Worker Threads 的限制有哪些？**
> 主要限制：① 不能在 Worker 中使用 `cluster` 模块；② 部分 native addon 的同步 API 不兼容（如 `node-gyp` 编译的模块）；③ `SharedArrayBuffer` 只支持二进制数据，不能直接共享 JS 对象；④ Worker 中没有 `process.stdin`；⑤ `uncaughtException` 可能导致整个进程（包括主线程）崩溃；⑥ Worker 的 `require` 不共享主线程的模块缓存（每个 Worker 独立加载）。

**Q8: Cluster 和 Worker Threads 可以组合使用吗？**
> 可以组合。典型架构：Cluster 创建多个工作进程（水平扩展 HTTP 服务），每个工作进程内部再创建 Worker Threads（并行处理 CPU 密集任务）。例如：4 核 CPU → Cluster 创建 4 个工作进程 → 每个进程内 2 个 Worker Thread → 总计 8 个计算单元。注意：进程数 × 线程数不应超过 CPU 核数太多，否则上下文切换开销抵消并行收益。推荐进程数 = CPU 核数，线程数根据任务粒度 1-2 个。

---

**相关链接：**
- [[事件循环]]
- [[进程与子进程]]
- [[进程管理与守护]]
- [[性能调优]]
- Node.js Cluster 文档：https://nodejs.org/api/cluster.html
- Node.js Worker Threads 文档：https://nodejs.org/api/worker_threads.html
