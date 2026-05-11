---
tags:
  - Web前端
  - JavaScript
  - WebWorker
  - 并行
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Web Worker与并行计算

## What — 是什么

> Web Worker 是浏览器提供的独立后台线程，让 JavaScript 可以执行 CPU 密集任务而不阻塞主线程（UI 线程）。

**核心概念：**

- **专用 Worker**：`new Worker(url)`，一对一通信
- **Shared Worker**：多标签页共享同一个 Worker 实例
- **Service Worker**：特殊 Worker，拦截网络请求（见 [[Service Worker与PWA]]）
- **通信机制**：`postMessage` 发送数据，`onmessage` 接收数据
- **Transferable**：ArrayBuffer 等可零拷贝转移所有权

**关键特性：**

- Worker 线程无法访问 DOM、window、document
- 通信数据会被结构化克隆（深拷贝），Transferable 除外
- Worker 内可使用 fetch、IndexedDB、WebSocket、setTimeout
- Worker 出错不会崩溃主线程

## Why — 为什么

**适用场景：**

- 大数据计算（排序、加密、压缩）
- 图片/音视频处理
- 大文件解析（CSV、JSON、Excel）
- 复杂算法（路径规划、数据挖掘）

**对比替代方案：**

| 维度 | Web Worker | 主线程分片 | WASM |
|------|-----------|-----------|------|
| 不阻塞 UI | ✅ 完全不阻塞 | 部分（需让出主线程） | ❌ 仍阻塞主线程 |
| 计算速度 | 原生 JS 速度 | 原生 JS 速度 | 接近 C 速度 |
| DOM 访问 | 不可 | 可以 | 不可 |
| 通信开销 | 结构化克隆 | 无 | 需共享内存 |
| 学习成本 | 低 | 低 | 高 |

**优缺点：**

- ✅ 优点：
  - 彻底解决计算阻塞 UI
  - 独立线程，错误不影响主线程
  - 可利用多核 CPU
- ❌ 缺点：
  - 通信有序列化开销
  - 不能访问 DOM
  - 调试相对不便

## How — 怎么用

### 快速上手

**主线程：**

```javascript
// 创建 Worker
const worker = new Worker(new URL('./worker.ts', import.meta.url));

// 发送数据
worker.postMessage({ type: 'sort', data: largeArray });

// 接收结果
worker.onmessage = (event) => {
    console.log('排序完成:', event.data);
};

// 错误处理
worker.onerror = (error) => {
    console.error('Worker 错误:', error.message);
};

// 终止
worker.terminate();
```

**Worker 线程：**

```javascript
// worker.ts
self.onmessage = (event) => {
    const { type, data } = event.data;

    switch (type) {
        case 'sort': {
            const result = data.sort((a, b) => a - b);
            self.postMessage({ type: 'sort:result', data: result });
            break;
        }
        case 'compute': {
            const result = heavyComputation(data);
            self.postMessage({ type: 'compute:result', data: result });
            break;
        }
    }
};
```

### 代码示例

**React Hook 封装：**

```typescript
function useWorker<T, R>(
    workerFn: (data: T) => R,
): { run: (data: T) => Promise<R>; terminate: () => void } {
    const workerRef = useRef<Worker | null>(null);

    const run = useCallback((data: T): Promise<R> => {
        return new Promise((resolve, reject) => {
            if (!workerRef.current) {
                const blob = new Blob(
                    [`self.onmessage = (e) => { self.postMessage((${workerFn.toString()})(e.data)); }`],
                    { type: 'application/javascript' },
                );
                workerRef.current = new Worker(URL.createObjectURL(blob));
            }

            workerRef.current.onmessage = (e) => resolve(e.data);
            workerRef.current.onerror = (e) => reject(new Error(e.message));
            workerRef.current.postMessage(data);
        });
    }, [workerFn]);

    const terminate = useCallback(() => {
        workerRef.current?.terminate();
        workerRef.current = null;
    }, []);

    useEffect(() => () => terminate(), [terminate]);

    return { run, terminate };
}

// 使用
function DataProcessor() {
    const { run, terminate } = useWorker((data: number[]) => {
        return data.filter(x => x > 0).sort((a, b) => a - b);
    });

    const handleClick = async () => {
        const result = await run(largeArray);
        console.log(result);
    };

    return <button onClick={handleClick}>处理数据</button>;
}
```

**Transferable 零拷贝传输：**

```javascript
// 主线程
const buffer = new ArrayBuffer(1024 * 1024 * 10); // 10MB
worker.postMessage({ buffer }, [buffer]); // 第二个参数是 Transferable 列表
// 此后主线程无法访问 buffer，所有权已转移

// Worker 线程
self.onmessage = (event) => {
    const buffer = event.data.buffer;
    const view = new Float64Array(buffer);
    // 处理数据...
    self.postMessage({ buffer }, [buffer]); // 处理完传回
};
```

**SharedArrayBuffer 共享内存：**

```javascript
// 主线程
const shared = new SharedArrayBuffer(1024);
const view = new Int32Array(shared);
worker.postMessage({ shared });

// Worker 线程
self.onmessage = (event) => {
    const view = new Int32Array(event.data.shared);
    Atomics.add(view, 0, 1); // 原子操作
    Atomics.notify(view, 0); // 通知等待的线程
};

// 注意：SharedArrayBuffer 要求 Cross-Origin-Isolation
// 需设置响应头：
// Cross-Origin-Opener-Policy: same-origin
// Cross-Origin-Embedder-Policy: require-corp
```

**大文件分片处理：**

```javascript
// 主线程：分片发送
async function processLargeFile(file) {
    const CHUNK_SIZE = 1024 * 1024; // 1MB
    const results = [];

    const worker = new Worker(new URL('./file-worker.ts', import.meta.url));

    for (let offset = 0; offset < file.size; offset += CHUNK_SIZE) {
        const chunk = file.slice(offset, offset + CHUNK_SIZE);
        const buffer = await chunk.arrayBuffer();

        const result = await new Promise((resolve) => {
            worker.onmessage = (e) => resolve(e.data);
            worker.postMessage({ chunk: buffer, offset }, [buffer]);
        });

        results.push(result);
    }

    worker.terminate();
    return mergeResults(results);
}

// file-worker.ts
self.onmessage = (event) => {
    const { chunk, offset } = event.data;
    // 例如：计算每片的 MD5
    const hash = computeHash(chunk);
    self.postMessage({ hash, offset });
};
```

**Worker 内 import 模块：**

```javascript
// Vite 支持的 Worker 模块语法
// worker.ts
import { parseCSV } from './utils/csv'; // 正常 import

self.onmessage = (event) => {
    const result = parseCSV(event.data);
    self.postMessage(result);
};

// 主线程：Vite 语法
const worker = new Worker(new URL('./worker.ts', import.meta.url), {
    type: 'module', // 启用 ES Module
});
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 通信慢 | 大数据结构化克隆耗时 | 用 Transferable 零拷贝转移 ArrayBuffer |
| Worker 内报错静默 | 未监听 onerror | 始终监听 onerror 并上报 |
| Worker 重复创建 | 每次渲染新建 Worker | useRef 持有实例，卸载时 terminate |
| 无法访问 DOM | Worker 线程无 DOM | 在 Worker 中计算，主线程渲染 |
| SharedArrayBuffer 不可用 | 缺少 COOP/COEP 响应头 | 服务器配置安全头 |

### 最佳实践

- 超过 50ms 的计算任务考虑放入 Worker
- 大数据用 Transferable 零拷贝，避免序列化开销
- Worker 生命周期管理：创建 → 复用 → 终止
- Vite 项目用 `new URL('./worker.ts', import.meta.url)` 语法
- 简单计算不必用 Worker，分片 `requestIdleCallback` 也可

## 面试题

**Q1: Web Worker 的通信方式有哪些？**
> 主要通信方式：1) `postMessage`/`onmessage`：发送可结构化克隆的数据（深拷贝）；2) Transferable 对象：`postMessage(data, [transferList])` 零拷贝转移 ArrayBuffer 等所有权；3) SharedArrayBuffer：多线程共享同一块内存，配合 `Atomics` 做原子操作（需 COOP/COEP 安全头）。Worker 不可直接访问 DOM。

**Q2: Transferable 的原理是什么？为什么能提升性能？**
> Transferable 机制将 ArrayBuffer 等的所有权从发送方转移给接收方，而非复制。转移后发送方无法再访问该数据（buffer 变为 0 字节），但避免了结构化克隆的深拷贝开销。对于大型二进制数据（图片、音视频、大数组），从拷贝几十 MB 变为仅传递指针，性能提升显著。

**Q3: 主线程被阻塞时有什么解决方案？**
> 方案：1) 将 CPU 密集任务移入 Web Worker（完全不阻塞 UI）；2) 主线程分片执行：用 `requestIdleCallback` 或 `setTimeout(fn, 0)` 将长任务拆成小片段，每片执行后让出主线程；3) 使用 WASM 提升计算速度（仍阻塞但耗时更短）；4) `scheduler.yield()`（新 API）主动让出主线程。超过 50ms 的任务建议放入 Worker。

---

**相关链接：**
- [[Promise与异步]]
- [[前端性能优化]]
- [[Service Worker与PWA]]
